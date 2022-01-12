---
title: MonoJust与MonoDefer区别
date: 2022-01-12 14:25:31
tags: [技术]
categories: 响应式编程
---

MonoJust与MonoDefer区别

<!-- more -->

先看个例子

```java
@Test
public void monoJust() {

    Mono<Integer> justMono = Mono.just((new Random()).nextInt(100));
    for (int i = 0; i < 3; i++) {
        justMono.subscribe(x -> {
            System.out.println("Mono just print " + x);
        });
    }

    final Mono<Integer> deferMono = Mono.defer(() -> Mono.just((new Random()).nextInt(100)));
    for (int i = 0; i < 3; i++) {
        deferMono.subscribe(x -> {
            System.out.println("Mono Defer print:  " + x);
        });
    }
    
    
   输出结果：
       
    Mono just print 61
    Mono just print 61
    Mono just print 61
    Mono Defer print:  23
    Mono Defer print:  80
    Mono Defer print:  77
```



例子都是实例化一个Mono，三次订阅消费打印值。可以看出Mono.just值没有变。Mono.defer每次订阅都会变成新值。

总结区别如下：

- Mono.just创建一个Observable观察者，每次订阅都会重用
- Mono.defer会在每次订阅时会重新创建一个Observable观察者。



**个人结论：**

​	如果是作为数据查询，最好选择Mono.just。

