---
title: SpringCloud Gateway 超时机制实现
date: 2021-12-17 18:03:24
tags: [技术]
categories: SpringCloud
---

底层用的是ScheduledThreadPoolExecutor实现

<!-- more -->



最近项目一个请求发现报 504 GATEWAY_TIMEOUT。然后就想这玩意怎么实现的。一看日志大概晓得是怎么回事了。上日志



```java
org.springframework.web.server.ResponseStatusException: 504 GATEWAY_TIMEOUT "Response took longer than timeout: PT20S"; nested exception is org.springframework.cloud.gateway.support.TimeoutException: Response took longer than timeout: PT20S

 	at org.springframework.cloud.gateway.filter.NettyRoutingFilter.lambda$filter$6(NettyRoutingFilter.java:201)

 	at reactor.core.publisher.Flux.lambda$onErrorMap$25(Flux.java:6316)

 	at reactor.core.publisher.Flux.lambda$onErrorResume$26(Flux.java:6369)

 	at reactor.core.publisher.FluxOnErrorResume$ResumeSubscriber.onError(FluxOnErrorResume.java:88)

 	at reactor.core.publisher.SerializedSubscriber.onError(SerializedSubscriber.java:124)

 	at reactor.core.publisher.FluxTimeout$TimeoutOtherSubscriber.onError(FluxTimeout.java:323)

 	at reactor.core.publisher.Operators.error(Operators.java:184)

 	at reactor.core.publisher.MonoError.subscribe(MonoError.java:52)

 	at reactor.core.publisher.Mono.subscribe(Mono.java:3873)

 	at reactor.core.publisher.FluxTimeout$TimeoutMainSubscriber.handleTimeout(FluxTimeout.java:294)

 	at reactor.core.publisher.FluxTimeout$TimeoutMainSubscriber.doTimeout(FluxTimeout.java:273)

 	at reactor.core.publisher.FluxTimeout$TimeoutTimeoutSubscriber.onNext(FluxTimeout.java:390)

 	at reactor.core.publisher.StrictSubscriber.onNext(StrictSubscriber.java:89)

 	at reactor.core.publisher.FluxOnErrorResume$ResumeSubscriber.onNext(FluxOnErrorResume.java:73)

 	at reactor.core.publisher.MonoDelay$MonoDelayRunnable.run(MonoDelay.java:117)

 	at reactor.core.scheduler.SchedulerTask.call(SchedulerTask.java:50)

 	at reactor.core.scheduler.SchedulerTask.call(SchedulerTask.java:27)

 	at java.util.concurrent.FutureTask.run(FutureTask.java:266)

 	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:180)

 	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:293)

 	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)

 	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)

 	at java.lang.Thread.run(Thread.java:748)

 Caused by: org.springframework.cloud.gateway.support.TimeoutException: Response took longer than timeout: PT20S


```



从堆栈中看到是从NettyRoutingFilter抛出的。它是一个GlobalFilter。在最后执行，等待请求的响应。源码如下

```java
public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
   URI requestUrl = exchange.getRequiredAttribute(GATEWAY_REQUEST_URL_ATTR);

   String scheme = requestUrl.getScheme();
   if (isAlreadyRouted(exchange)
         || (!"http".equals(scheme) && !"https".equals(scheme))) {
      return chain.filter(exchange);
   }
   setAlreadyRouted(exchange);

   ServerHttpRequest request = exchange.getRequest();

   final HttpMethod method = HttpMethod.valueOf(request.getMethodValue());
   final String url = requestUrl.toASCIIString();

   HttpHeaders filtered = filterRequest(getHeadersFilters(), exchange);

   final DefaultHttpHeaders httpHeaders = new DefaultHttpHeaders();
   filtered.forEach(httpHeaders::set);

   boolean preserveHost = exchange
         .getAttributeOrDefault(PRESERVE_HOST_HEADER_ATTRIBUTE, false);

   Flux<HttpClientResponse> responseFlux = this.httpClient.headers(headers -> {
      headers.add(httpHeaders);
      // Will either be set below, or later by Netty
      headers.remove(HttpHeaders.HOST);
      if (preserveHost) {
         String host = request.getHeaders().getFirst(HttpHeaders.HOST);
         headers.add(HttpHeaders.HOST, host);
      }
   }).request(method).uri(url).send((req, nettyOutbound) -> {
      if (log.isTraceEnabled()) {
         nettyOutbound.withConnection(connection -> log.trace(
               "outbound route: " + connection.channel().id().asShortText()
                     + ", inbound: " + exchange.getLogPrefix()));
      }
      return nettyOutbound.options(NettyPipeline.SendOptions::flushOnEach).send(
            request.getBody().map(dataBuffer -> ((NettyDataBuffer) dataBuffer)
                  .getNativeBuffer()));
   }).responseConnection((res, connection) -> {

      // Defer committing the response until all route filters have run
      // Put client response as ServerWebExchange attribute and write
      // response later NettyWriteResponseFilter
      exchange.getAttributes().put(CLIENT_RESPONSE_ATTR, res);
      exchange.getAttributes().put(CLIENT_RESPONSE_CONN_ATTR, connection);

      ServerHttpResponse response = exchange.getResponse();
      // put headers and status so filters can modify the response
      HttpHeaders headers = new HttpHeaders();

      res.responseHeaders()
            .forEach(entry -> headers.add(entry.getKey(), entry.getValue()));

      String contentTypeValue = headers.getFirst(HttpHeaders.CONTENT_TYPE);
      if (StringUtils.hasLength(contentTypeValue)) {
         exchange.getAttributes().put(ORIGINAL_RESPONSE_CONTENT_TYPE_ATTR,
               contentTypeValue);
      }

      HttpStatus status = HttpStatus.resolve(res.status().code());
      if (status != null) {
         response.setStatusCode(status);
      }
      else if (response instanceof AbstractServerHttpResponse) {
         // https://jira.spring.io/browse/SPR-16748
         ((AbstractServerHttpResponse) response)
               .setStatusCodeValue(res.status().code());
      }
      else {
         // TODO: log warning here, not throw error?
         throw new IllegalStateException("Unable to set status code on response: "
               + res.status().code() + ", " + response.getClass());
      }

      // make sure headers filters run after setting status so it is
      // available in response
      HttpHeaders filteredResponseHeaders = HttpHeadersFilter
            .filter(getHeadersFilters(), headers, exchange, Type.RESPONSE);

      if (!filteredResponseHeaders.containsKey(HttpHeaders.TRANSFER_ENCODING)
            && filteredResponseHeaders.containsKey(HttpHeaders.CONTENT_LENGTH)) {
         // It is not valid to have both the transfer-encoding header and
         // the content-length header.
         // Remove the transfer-encoding header in the response if the
         // content-length header is present.
         response.getHeaders().remove(HttpHeaders.TRANSFER_ENCODING);
      }

      exchange.getAttributes().put(CLIENT_RESPONSE_HEADER_NAMES,
            filteredResponseHeaders.keySet());

      response.getHeaders().putAll(filteredResponseHeaders);

      return Mono.just(res);
   });

   if (properties.getResponseTimeout() != null) {
      responseFlux = responseFlux.timeout(properties.getResponseTimeout(),
            Mono.error(new TimeoutException("Response took longer than timeout: "
                  + properties.getResponseTimeout())))
            .onErrorMap(TimeoutException.class,
                  th -> new ResponseStatusException(HttpStatus.GATEWAY_TIMEOUT,
                        th.getMessage(), th));
   }

   return responseFlux.then(chain.filter(exchange));
}
```



核心代码是，就是调用的 Flux的方法 timeout(Duration timeout, @Nullable Publisher<? extends T> fallback)。

```java
if (properties.getResponseTimeout() != null) {
      responseFlux = responseFlux.timeout(properties.getResponseTimeout(),
            Mono.error(new TimeoutException("Response took longer than timeout: "
                  + properties.getResponseTimeout())))
            .onErrorMap(TimeoutException.class,
                  th -> new ResponseStatusException(HttpStatus.GATEWAY_TIMEOUT,
                        th.getMessage(), th));
   }
```

这里的fallback抛出一个TimeoutException异常。

这里说清楚。看下 timeout(Duration timeout, @Nullable Publisher<? extends T> fallback)。的实现

timeout方法调用的是

```java
public final Flux<T> timeout(Duration timeout) {
		return timeout(timeout, null, Schedulers.parallel());
	}
```



再进入方法

```java
public final Flux<T> timeout(Duration timeout,
			@Nullable Publisher<? extends T> fallback,
			Scheduler timer) {
		final Mono<Long> _timer = Mono.delay(timeout, timer).onErrorReturn(0L);
		final Function<T, Publisher<Long>> rest = o -> _timer;

		if(fallback == null) {
			return timeout(_timer, rest, timeout.toMillis() + "ms");
		}
		return timeout(_timer, rest, fallback);
	}
```

发现调用的是 Mono.delay(timeout, timer).onErrorReturn(0L);

这里的timer就是Schedulers.parallel()实例。这里面用到了ScheduledExecutorService。核心就是这个来实现延时任务。ScheduledExecutorService具体使用可以看下JDK的文档。这里就不再阐述。

















