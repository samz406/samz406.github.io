---
title: 函数式编程浅析
date: 2021-07-03 19:41:55
tags: [技术]
categories: JAVA
---

JDK8函数式编程学习笔记

<!-- more -->



### Lambda表达式

Lambda表达式是一个匿名方法，将行为像数据一样进行传递。所有Lambda表达式中的参数类型都是由编译器推断得出。



![lambda](lambda.png)



- 没有返回值

  

```
 ()-> {}
```

该Lambda表达式不包含参数，使用空括号()表示没有参数，返回没有参数，且返回类型为void

- 有返回值

```
 ()->"hello"
```

- 有返回值

```
()->{return "hello";}
```

Lambda表达式的主体不仅可以是一个表达式，而且也可以是一段代码块，使用大括号（{}）将代码块括起来



Lambda表达式引用的是值，而不是变量




#### 函数接口

函数接口是只有一个抽象方法的接口，用作Lambda表达式的类型，函数接口中单一方法的命名并不重要，只要方法签名和Lambda表达式的类型匹配即可。但是建议还是在函数接口中为参数起一个有意义的名字，增加代码易读性。Lambda表达式中的类型推断，是在Java 7引入目标类型推断的扩展。javac根据Lambda表达式上下文信息推到出参数的正确类型。

以下三个表达式是同一个意思：

```
Predicate<Integer> prd = (Integer x) -> x > 45;
```

```
Predicate<Integer> prd = (x) -> x > 45;
```

```
Predicate<Integer> prd = x -> x > 45;
```

这里跟Map<Integer,Integer> map = new HashMap<>;一样的哦，构造函数省略了类型。



**JDK内置函数接口**

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



#### 流Stream









