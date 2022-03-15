---
title: 通过字节码来看string对象个数
date: 2022-03-15 18:18:00
tags: [技术]
categories: 技术
---



通过字节码来看string对象个数

<!-- more -->



先看下面一段代码，这是经常被问题的问题。

```java
public void add() {

    String str = new String("a"); 

    String b = "a"; 

    String str2 = "a" + "b"; 

    String str3 = "ab"; 
}
```



通过字节码查看

```
 public void add();
    Code:
       0: new           #20                 // class java/lang/String
       3: dup
       4: ldc           #21                 // String a
       6: invokespecial #22                 // Method java/lang/String."<init>":(Ljava/lang/String;)V
       9: astore_1
      10: ldc           #21                 // String a
      12: astore_2
      13: ldc           #23                 // String ab
      15: astore_3
      16: ldc           #23                 // String ab
      18: astore        4
      20: return

```



第1句是 new 一个对象，字符串a是通过ldc从常量池加载。

 第2句 字符串也是ldc从常量池加载

第3句 是ldc从常量池加载。这里只有一个对象

第4句 ldc从常量池加载，和第3句一样。



