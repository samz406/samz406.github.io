---
title: Feign使用问题总结
date: 2021-11-15 19:55:51
tags: [技术]
categories: Spring
---

Feign使用问题总结

<!-- more -->

### Feign是个什么玩意

Feign是一个声明式的Web Service客户端，它的目的就是让Web Service调用更加简单。Feign提供了HTTP请求的模板，通过编写简单的接口并插入注解，就可以完成HTTP请求的参数、格式、地址等信息的声明。通过Feign代理HTTP请求，我们只需要像调用方法一样调用它就可以完成微服务请求及相关处理(网上找的)。Feign最大的感受就是不用写RestTemplate。RestTemplate虽然比较简化了，但是Feign使用起来更是无感是RPC。



### 问题

##### 问题1

```
***************************
APPLICATION FAILED TO START
***************************

Description:

The bean 'TestApi.FeignClientSpecification' could not be registered. A bean with that name has already been defined and overriding is disabled.

Action:

Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true

```

经过排查发现日志有A bean with that name has already been defined。意思是bean已经存在了。那么就有可能是TestApi被加载了两次。然后就看代码。修改记录发现再AppApplication类中的*@EnableFeignClients*的配置中出现了两个包路径指向同一个路径。所以就出现了上面的问题。

通过修改@EnableFeignClients({"package1","package2"})数组包名字符串。去掉其中一个就行。



##### 问题2 

错误日志

```java

Description:

The bean 'user.FeignClientSpecification' could not be registered. A bean with that name has already been defined and overriding is disabled.

Action:

Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true
```

代码如下。

```java
@RequestMapping("/test2")
@FeignClient(value = "user")
public interface TestApi2 {
}
```

```
@RequestMapping("/test")
@FeignClient(value = "user")
public interface TestApi {
}
```

这个问题跟上面的问题1本质是一样的。就是找到了相同了bean name。这里跟上面不一样的是@EnableFeignClients没有配置相同的包路径。这里问题是

value重复了。

**解决办法：**

​	@FeignClient 中增加contextId属性。主要目的是定义一个不同的bean名称。



样例如下：
```
	@RequestMapping("/test2")
	@FeignClient(value = "user",contextId="TestApi2")
	public interface TestApi2 {
	}
```


​	
```
    @RequestMapping("/test")
    @FeignClient(value = "user",contextId="TestApi")
    public interface TestApi {
    }
```




##### 问题3

使用@FeignClients调用第三方服务（不是同一个注册中心服务）。配置如下

```
@FeignClients(url = "http://www.test.com", contextId = "clientApi")
```

配置好后重启服务。报如下错误。

```
***************************
APPLICATION FAILED TO START
***************************

Description:

A component required a bean of type 'com.test.model.testClientApi' that could not be found.


Action:

Consider defining a bean of type 'com.test.model.TestClientApi' in your configuration.
```

看意思就是service类没有成功注入TestClientApi 。之前引用在注册中心内其他服务是这样配置@FeignClients(value= "user", contextId = "clientApi")。都会成功。



折腾了一番。翻了下FeignClients 注解才理解了下。FeignClients 有如下配置。去掉不常用的。

```java
/**
	 * The name of the service with optional protocol prefix. Synonym for {@link #name()
	 * name}. A name must be specified for all clients, whether or not a url is provided.
	 * Can be specified as property key, eg: ${propertyKey}.
	 * @return the name of the service with optional protocol prefix
	 */
	@AliasFor("name")
	String value() default "";

	/**
	 * The service id with optional protocol prefix. Synonym for {@link #value() value}.
	 * @deprecated use {@link #name() name} instead
	 * @return the service id with optional protocol prefix
	 */
	@Deprecated
	String serviceId() default "";

	/**
	 * This will be used as the bean name instead of name if present, but will not be used
	 * as a service id.
	 * @return bean name instead of name if present
	 */
	String contextId() default "";

	/**
	 * @return The service id with optional protocol prefix. Synonym for {@link #value()
	 * value}.
	 */
	@AliasFor("value")
	String name() default "";

	/**
	 * @return the <code>@Qualifier</code> value for the feign client.
	 */
	String qualifier() default "";

	/**
	 * @return an absolute URL or resolvable hostname (the protocol is optional).
	 */
	String url() default "";

	/**
	 * @return whether to mark the feign proxy as a primary bean. Defaults to true.
	 */
	boolean primary() default true;

```



其中有句话A name must be specified for all clients, whether or not a url is provided。意思就是url配置需指定一个name或value属性值。上面配置的确是没有name或value。

​	解决办法：

​	@FeignClients(value= "user", contextId = "clientApi",name="clientApi"）。在	@FeignClients增加属性name就能搞定。



### 源码分析

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(FeignClientsRegistrar.class)
public @interface EnableFeignClients {
    
}
```



@EnableFeignClients 注解中，@Import中FeignClientsRegistrar是入口。它主要功能是解析包下的@FeignClient接口。转换成bean。其中有设置bean名称。

其中设置bean名称如下

```java
private void registerClientConfiguration(BeanDefinitionRegistry registry, Object name,
      Object configuration) {
   BeanDefinitionBuilder builder = BeanDefinitionBuilder
         .genericBeanDefinition(FeignClientSpecification.class);
   builder.addConstructorArgValue(name);
   builder.addConstructorArgValue(configuration);
   registry.registerBeanDefinition(
         name + "." + FeignClientSpecification.class.getSimpleName(),
         builder.getBeanDefinition());
}
```

registry.registerBeanDefinition(name + "." + FeignClientSpecification.class.getSimpleName(),  builder.getBeanDefinition()); 这句话就是创建BeanDefinition。

name 来之如下代码。

```
private String getClientName(Map<String, Object> client) {
   if (client == null) {
      return null;
   }
   String value = (String) client.get("contextId");
   if (!StringUtils.hasText(value)) {
      value = (String) client.get("value");
   }
   if (!StringUtils.hasText(value)) {
      value = (String) client.get("name");
   }
   if (!StringUtils.hasText(value)) {
      value = (String) client.get("serviceId");
   }
   if (StringUtils.hasText(value)) {
      return value;
   }

   throw new IllegalStateException("Either 'name' or 'value' must be provided in @"
         + FeignClient.class.getSimpleName());
}
```

拿名称的顺序是contextId->value->name->serviceId。





