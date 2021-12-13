---
title: JAVA引用类型
date: 2021-12-07 22:00:45
tags: 技术
categories: JAVA
---



这里引用表示对象之间的关联关系。包括强引用、软引用、弱引用、虚引用4种，引用强度依次逐渐减弱

<!-- more -->

### 强引用（Strongly Re-ference）

​	强引用是程序代码之中普遍存在的引用赋值，即类似“Object obj=new Object()”这种引用关系。无论任何情况下，只要强引用关系还存在，垃圾收集器就永远不会回收掉被引用的对象。

### 软引用（Soft Reference）

​	软引用是用来描述一些还有用非必须的对象。只被软引用关联着的对象，在系统将要发生内存溢出异常时会对这些对象列进回收范围之中进行第二次回收，如果这次回收内存还不够，才会抛出内存溢出异常。

一般都用缓存对象。

### 弱引用（Weak Reference）

​	弱引用也是用来描述那些非必须对象，但它的强度比软引用更弱一些，被弱引用关联的对象只能生存到下一次垃圾收集发生为止。当垃圾收集器开始工作，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。



### 虚引用（Phantom Reference）

​	虚引用也称为“幽灵引用”或者“幻影引用”，它是最弱的一种引用关系。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的只是为了能在这个对象被收集器回收时收到一个系统通知。







### ConcurrentReferenceHashMap使用



ConcurrentReferenceHashMap是一个ConcurrentHashMap。但是它的key和value是soft或weak引用。这里面定义了两种类型，默认的都是SOFT,

```java
public enum ReferenceType {

		/** Use {@link SoftReference SoftReferences}. */
		SOFT,

		/** Use {@link WeakReference WeakReferences}. */
		WEAK
	}

```

下面是一个框架中使用的地方



- **SpringFactoriesLoader**

from spring-core

参数定义：

```java
private static final Map<ClassLoader, MultiValueMap<String, String>> cache = new ConcurrentReferenceHashMap<>();
```

使用的地方

```java
private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
   MultiValueMap<String, String> result = cache.get(classLoader);
    //先判断是否存在。
   if (result != null) {
      return result;
   }

   try {
      Enumeration<URL> urls = (classLoader != null ?
            classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
            ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
      result = new LinkedMultiValueMap<>();
      while (urls.hasMoreElements()) {
         URL url = urls.nextElement();
         UrlResource resource = new UrlResource(url);
         Properties properties = PropertiesLoaderUtils.loadProperties(resource);
         for (Map.Entry<?, ?> entry : properties.entrySet()) {
            String factoryClassName = ((String) entry.getKey()).trim();
            for (String factoryName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
               result.add(factoryClassName, factoryName.trim());
            }
         }
      }
       //设置参数
      cache.put(classLoader, result);
      return result;
   }
   catch (IOException ex) {
      throw new IllegalArgumentException("Unable to load factories from location [" +
            FACTORIES_RESOURCE_LOCATION + "]", ex);
   }
}
```





- **CachingSpringLoadBalancerFactory**

from openfeign-core

定义

```java
private volatile Map<String, FeignLoadBalancer> cache = new ConcurrentReferenceHashMap<>();
```

使用

```java
public FeignLoadBalancer create(String clientName) {
   FeignLoadBalancer client = this.cache.get(clientName);
   if (client != null) {
      return client;
   }
   IClientConfig config = this.factory.getClientConfig(clientName);
   ILoadBalancer lb = this.factory.getLoadBalancer(clientName);
   ServerIntrospector serverIntrospector = this.factory.getInstance(clientName,
         ServerIntrospector.class);
   client = this.loadBalancedRetryFactory != null
         ? new RetryableFeignLoadBalancer(lb, config, serverIntrospector,
               this.loadBalancedRetryFactory)
         : new FeignLoadBalancer(lb, config, serverIntrospector);
   this.cache.put(clientName, client);
   return client;
}
```



- **OrderUtils**

from spring-core

```java
/** Cache for @Priority value (or NOT_ANNOTATED marker) per Class. */
private static final Map<Class<?>, Object> priorityCache = new ConcurrentReferenceHashMap<>();
```



```java
public static Integer getOrder(Class<?> type) {
   Object cached = orderCache.get(type);
   if (cached != null) {
      return (cached instanceof Integer ? (Integer) cached : null);
   }
   Order order = AnnotationUtils.findAnnotation(type, Order.class);
   Integer result;
   if (order != null) {
      result = order.value();
   }
   else {
      result = getPriority(type);
   }
   orderCache.put(type, (result != null ? result : NOT_ANNOTATED));
   return result;
}
```



- **ReflectionUtils**

from spring-core

定义：

```java
/**
 * Cache for {@link Class#getDeclaredMethods()} plus equivalent default methods
 * from Java 8 based interfaces, allowing for fast iteration.
 */
private static final Map<Class<?>, Method[]> declaredMethodsCache = new ConcurrentReferenceHashMap<>(256);
```

使用：

```java
private static Method[] getDeclaredMethods(Class<?> clazz) {
   Assert.notNull(clazz, "Class must not be null");
   Method[] result = declaredMethodsCache.get(clazz);
   if (result == null) {
      try {
         Method[] declaredMethods = clazz.getDeclaredMethods();
         List<Method> defaultMethods = findConcreteMethodsOnInterfaces(clazz);
         if (defaultMethods != null) {
            result = new Method[declaredMethods.length + defaultMethods.size()];
            System.arraycopy(declaredMethods, 0, result, 0, declaredMethods.length);
            int index = declaredMethods.length;
            for (Method defaultMethod : defaultMethods) {
               result[index] = defaultMethod;
               index++;
            }
         }
         else {
            result = declaredMethods;
         }
          //保存
         declaredMethodsCache.put(clazz, (result.length == 0 ? EMPTY_METHOD_ARRAY : result));
      }
      catch (Throwable ex) {
         throw new IllegalStateException("Failed to introspect Class [" + clazz.getName() +
               "] from ClassLoader [" + clazz.getClassLoader() + "]", ex);
      }
   }
   return result;
}
```

