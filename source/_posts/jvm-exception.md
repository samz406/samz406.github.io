---

title: JVM 异常处理
date: 2022-05-14 19:27:47
tags: [技术]
categories: JAVA
---

从字节码角度看异常处理

<!-- more -->

### 异常基本概念



​	在Java中，异常祖先是Throwable类，Throwable有两个亲儿子类。一个是Error，是程序不应捕获的异常。当程序触发 Error 时，它的执行状态已经无法恢复，需要中止线程甚至是中止虚拟机。第二个儿子是 Exception，程序可能需要捕获并且处理的异常。它有个特殊子类 RuntimeException，用来表示“**程序虽然无法继续执行，但是还能抢救一下**”的情况。

RuntimeException 和 Error 属于 Java 里的非检查异常（**unchecked exception**）。其他异常则属于检查异常（**checked exception**）

> 检查异常都需要程序显式地捕获，或在方法声明中用 throws 关键字



![封面](ex1.png)



每个方法都附带一个异常表。异常表中的每一个条目代表一个异常处理器，并且由from指针、to指针、target 指针以及所捕获的异常类型构成，from 指针和 to 指针标示了该异常处理器所监控的范围。

from 指针和 to 指针标示了该异常处理器所监控的范围，程序抛异常时，JVM会从上至下遍历异常表中的所有异常条目。当触发异常的字节码的索引值在某个异常表条目范围内，拟机会判断所抛出的异常和该条目想要捕获的异常是否匹配。

如果匹配，虚拟机会将控制流转移至该条目 target 指针指向的字节码。如果遍历完所有异常表条目，JVM仍未匹配到异常处理器，那么它会弹出当前方法对应的 Java 栈帧，并且在调用者中重复上述操作。在最坏情况下，Java 虚拟机需要遍历当前线程 Java 栈上所有方法的异常表。



先看一段代码

```java
public void print() {
    try {
        int i = 2/0;

    } catch (ArrayIndexOutOfBoundsException ex) {
        ex.printStackTrace();
    } catch (RuntimeException ex) {
        ex.printStackTrace();
    } catch (Exception ex) {
        ex.printStackTrace();
    } catch (Throwable ex){
        ex.printStackTrace();
    }
}
```

代码有几个catch块组成。

```java
 public void print();
    Code:
       0: iconst_2
       1: iconst_0
       2: idiv
       3: istore_1
       4: goto          36
       7: astore_1
       8: aload_1
       9: invokevirtual #3                  // Method java/lang/ArrayIndexOutOfBoundsException.printStackTrace:()V
      12: goto          36
      15: astore_1
      16: aload_1
      17: invokevirtual #5                  // Method java/lang/RuntimeException.printStackTrace:()V
      20: goto          36
      23: astore_1
      24: aload_1
      25: invokevirtual #7                  // Method java/lang/Exception.printStackTrace:()V
      28: goto          36
      31: astore_1
      32: aload_1
      33: invokevirtual #9                  // Method java/lang/Throwable.printStackTrace:()V
      36: return
    Exception table:
       from    to  target type
           0     4     7   Class java/lang/ArrayIndexOutOfBoundsException
           0     4    15   Class java/lang/RuntimeException
           0     4    23   Class java/lang/Exception
           0     4    31   Class java/lang/Throwable


```

其中 from=0,to=4表示try代码块。这几个异常都是同时捕获了相同的代码块。target就是异常代码的字节码索引。

> try 代码块后面可以跟着多个 catch 代码块，来捕获不同类型的异常。JVM会从上至下匹配异常处理器，前面的 catch 代码块所捕获的异常类型不能覆盖后边的，否则编译器会报错。也就是前面的异常是后面的异常的子类。



#### finally 关键字

来看一段代码

```java
public void printFinal(){

    try {
        int i = 2/0;
    } catch (ArrayIndexOutOfBoundsException ex) {
        ex.printStackTrace();
    } finally {
        System.out.println("hello world!!!");
    }
}
```



字节码

```java
 public void printFinal();
    Code:
       0: iconst_2
       1: iconst_0
       2: idiv
       3: istore_1
       4: getstatic     #10                 // Field java/lang/System.out:Ljava/io/PrintStream;
       7: ldc           #11                 // String hello world!!!
       9: invokevirtual #12                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      12: goto          42
      15: astore_1
      16: aload_1
      17: invokevirtual #3                  // Method java/lang/ArrayIndexOutOfBoundsException.printStackTrace:()V
      20: getstatic     #10                 // Field java/lang/System.out:Ljava/io/PrintStream;
      23: ldc           #11                 // String hello world!!!
      25: invokevirtual #12                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      28: goto          42
      31: astore_2
      32: getstatic     #10                 // Field java/lang/System.out:Ljava/io/PrintStream;
      35: ldc           #11                 // String hello world!!!
      37: invokevirtual #12                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      40: aload_2
      41: athrow
      42: return
    Exception table:
       from    to  target type
           0     4    15   Class java/lang/ArrayIndexOutOfBoundsException
           0     4    31   any
          15    20    31   any
}

```

可以看出字节码索引7，23，35 都包含“hello world”。也就是在try块、catch块都有finally代码。Java编译器会生成一个或多个异常表条目，监控整个 try-catch 代码块，并且捕获所有种类的异常（any）。这些异常表条目的 target 指针将指向另一份复制的 finally 代码块。并且，在这个 finally 代码块的最后，Java 编译器会重新抛出所捕获的异常。



##### Suppressed 异常

先看一段代码

```java
public static void printStackTrace() {

    try {
        int t = 2 / 0;
    } catch (RuntimeException ex) {

        ex.printStackTrace();

        throw new RuntimeException("catch RuntimeException");
    } finally {

        throw new RuntimeException("finally RuntimeException");

    }
}
```

异常信息如下

```java
java.lang.ArithmeticException: / by zero
	at org.samz.study.basic.PrintException.printStackTrace(PrintException.java:113)
	at org.samz.study.basic.PrintException.main(PrintException.java:6)
Exception in thread "main" java.lang.RuntimeException: finally RuntimeException
	at org.samz.study.basic.PrintException.printStackTrace(PrintException.java:121)
	at org.samz.study.basic.PrintException.main(PrintException.java:6)
```

对应的字节码

```java
 public static void printStackTrace();
    Code:
       0: iconst_2
       1: iconst_0
       2: idiv
       3: istore_0
       4: new           #5                  // class java/lang/RuntimeException
       7: dup
       8: ldc           #14                 // String finally RuntimeException
      10: invokespecial #15                 // Method java/lang/RuntimeException."<init>":(Ljava/lang/String;)V
      13: athrow
      14: astore_0
      15: aload_0
      16: invokevirtual #6                  // Method java/lang/RuntimeException.printStackTrace:()V
      19: new           #5                  // class java/lang/RuntimeException
      22: dup
      23: ldc           #16                 // String catch RuntimeException
      25: invokespecial #15                 // Method java/lang/RuntimeException."<init>":(Ljava/lang/String;)V
      28: athrow
      29: astore_1
      30: new           #5                  // class java/lang/RuntimeException
      33: dup
      34: ldc           #14                 // String finally RuntimeException
      36: invokespecial #15                 // Method java/lang/RuntimeException."<init>":(Ljava/lang/String;)V
      39: athrow
    Exception table:
       from    to  target type
           0     4    14   Class java/lang/RuntimeException
           0     4    29   any
          14    30    29   any


```



结果是抛出了“finally RuntimeException”，也就是catch块的异常吃掉了。对此JDK7 引入 Suppressed 异常来解决这个问题。 Suppressed 可以将一个异常附于另一个异常之上。因此，抛出的异常可以附带多个异常的信息。然而，Java 层面的 finally 代码块缺少指向所捕获异常的引用。

再看下面的方法

```java
public static void printStackTrace2() {

        RuntimeException tempEx = null;

        try {
            int t = 2 / 0;
        } catch (RuntimeException ex) {

            ex.printStackTrace();
            tempEx = new RuntimeException("catch RuntimeException");
            throw tempEx;
        } finally {

            RuntimeException finallyEx = new RuntimeException("finally RuntimeException");
            finallyEx.addSuppressed(tempEx);
            throw finallyEx;

        }
    }
```

对应的异常

```java
java.lang.ArithmeticException: / by zero
	at org.samz.study.basic.PrintException.printStackTrace2(PrintException.java:130)
	at org.samz.study.basic.PrintException.main(PrintException.java:6)
Exception in thread "main" java.lang.RuntimeException: finally RuntimeException
	at org.samz.study.basic.PrintException.printStackTrace2(PrintException.java:138)
	at org.samz.study.basic.PrintException.main(PrintException.java:6)
	Suppressed: java.lang.RuntimeException: catch RuntimeException
		at org.samz.study.basic.PrintException.printStackTrace2(PrintException.java:134)
		... 1 more
```

打印出了catch模块和finally模块的异常。



对应的字节码

```java
public static void printStackTrace2();
    Code:
       0: aconst_null
       1: astore_0
       2: iconst_2
       3: iconst_0
       4: idiv
       5: istore_1
       6: new           #5                  // class java/lang/RuntimeException
       9: dup
      10: ldc           #14                 // String finally RuntimeException
      12: invokespecial #15                 // Method java/lang/RuntimeException."<init>":(Ljava/lang/String;)V
      15: astore_1
      16: aload_1
      17: aload_0
      18: invokevirtual #17                 // Method java/lang/RuntimeException.addSuppressed:(Ljava/lang/Throwable;)V
      21: aload_1
      22: athrow
      23: astore_1
      24: aload_1
      25: invokevirtual #6                  // Method java/lang/RuntimeException.printStackTrace:()V
      28: new           #5                  // class java/lang/RuntimeException
      31: dup
      32: ldc           #16                 // String catch RuntimeException
      34: invokespecial #15                 // Method java/lang/RuntimeException."<init>":(Ljava/lang/String;)V
      37: astore_0
      38: aload_0
      39: athrow
      40: astore_2
      41: new           #5                  // class java/lang/RuntimeException
      44: dup
      45: ldc           #14                 // String finally RuntimeException
      47: invokespecial #15                 // Method java/lang/RuntimeException."<init>":(Ljava/lang/String;)V
      50: astore_3
      51: aload_3
      52: aload_0
      53: invokevirtual #17                 // Method java/lang/RuntimeException.addSuppressed:(Ljava/lang/Throwable;)V
      56: aload_3
      57: athrow
    Exception table:
       from    to  target type
           2     6    23   Class java/lang/RuntimeException
           2     6    40   any
          23    41    40   any

```















