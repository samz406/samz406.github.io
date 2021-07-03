---
title: 函数式编程浅析
date: 2021-07-03 19:41:55
tags: [技术]
categories: JAVA
---

JDK8函数式编程学习笔记

<!-- more -->



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



### Lambda表达式

Lambda表达式是一个匿名方法，将行为像数据一样进行传递



