---
title: 响应式reactor编程浅析
date: 2021-07-02 18:35:31
tags: [技术]
---

**待写**



发布类

​	

```java
public interface Publisher<T> {

    /**
     * Request {@link Publisher} to start streaming data.
     * <p>
     * This is a "factory method" and can be called multiple times, each time starting a new {@link Subscription}.
     * <p>
     * Each {@link Subscription} will work for only a single {@link Subscriber}.
     * <p>
     * A {@link Subscriber} should only subscribe once to a single {@link Publisher}.
     * <p>
     * If the {@link Publisher} rejects the subscription attempt or otherwise fails it will
     * signal the error via {@link Subscriber#onError}.
     *
     * @param s the {@link Subscriber} that will consume signals from this {@link Publisher}
     */
    public void subscribe(Subscriber<? super T> s);
}
```



订阅类

​	发布者调用subscribe方法，触发订阅者消费

```java
public interface Subscriber<T> {
    /**
     * Invoked after calling {@link Publisher#subscribe(Subscriber)}.
     * <p>
     * No data will start flowing until {@link Subscription#request(long)} is invoked.
     * <p>
     * It is the responsibility of this {@link Subscriber} instance to call {@link Subscription#request(long)} whenever more data is wanted.
     * <p>
     * The {@link Publisher} will send notifications only in response to {@link Subscription#request(long)}.
     * 
     * @param s
     *            {@link Subscription} that allows requesting data via {@link Subscription#request(long)}
     */
    public void onSubscribe(Subscription s);

    /**
     * Data notification sent by the {@link Publisher} in response to requests to {@link Subscription#request(long)}.
     * 
     * @param t the element signaled
     */
    public void onNext(T t);

    /**
     * Failed terminal state.
     * <p>
     * No further events will be sent even if {@link Subscription#request(long)} is invoked again.
     *
     * @param t the throwable signaled
     */
    public void onError(Throwable t);

    /**
     * Successful terminal state.
     * <p>
     * No further events will be sent even if {@link Subscription#request(long)} is invoked again.
     */
    public void onComplete();
}
```





Mono.defer()

参数：Supplier<? extends Mono<? extends T>> supplier，

```java
public Mono<Void> filter(ServerWebExchange exchange) {
   return Mono.defer(() -> {
      if (this.index < filters.size()) {
         GatewayFilter filter = filters.get(this.index);
         DefaultGatewayFilterChain chain = new DefaultGatewayFilterChain(this,
               this.index + 1);
         return filter.filter(exchange, chain);
      }
      else {
         return Mono.empty(); // complete
      }
   });
}
```



Mono.just()



Mono.flatMap(Function<? super T, ? extends Mono<? extends R>)

参数:T是入参， Mono<? extends R>是返回参数





