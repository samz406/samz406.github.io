---
title: 函数式编程浅析
date: 2021-07-03 19:41:55
tags: [技术]
categories: JAVA
---

JDK8函数式编程学习笔记

<!-- more -->



#### 常用表达式



![lambda](lambda.png)



- 没有返回值

```
 ()-> {}
```

- 有返回值

```
 ()->"hello"
```

- 有返回值

```
()->{return "hello";}
```





#### 函数接口

函数接口是只有一个抽象方法的接口，用作Lambda表达式的类型

使用Java编程，总会遇到很多函数接口，但Java开发工具包（JDK）提供的一组核心函数接口会频繁出现

| 接口              | 参数  | 返回类型 |
| ----------------- | ----- | -------- |
| Predicate<T>      | T     | Boolean  |
| Consumer<T>       | T     |          |
| Function<T,R>     | T     | R        |
| Supplier<T>       | None  | T        |
| UnaryOperator<T>  | T     | T        |
| BinaryOperator<T> | (T,T) | T        |



函数描述符

函数式接口的抽象方法的签名基本上就是Lambda表达式的签名。这种抽象方法叫作函数描述符，例如，Runnable接口可以看作一个什么也不接受什么也不返回（void）的函数的签名，Runnable只有一个run抽象方法，这个方法什么也不接受，什么也不返回（void）。





### Lambda表达式

Lambda表达式是一个匿名方法，将行为像数据一样进行传递



