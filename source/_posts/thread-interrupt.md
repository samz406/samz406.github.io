---
title: 浅析java thread interrupt
date: 2021-08-04 10:24:31
tags: [技术]
categories: JAVA
---



thread.interrupt() 调用正确处理姿势

<!-- more -->



```java
 public static void main(String[] args) throws InterruptedException {

        Thread thread = new Thread(() -> {
            while (true) {

            }
        });
        thread.start();
     	//中断前查看线程状态
        System.out.println(thread.getState().name());
     	//中断
        thread.interrupt();
        TimeUnit.SECONDS.sleep(3);
     	//中断后查看线程状态
        System.out.println(thread.getState().name());

    }
```



输出结果如下，同时main线程挂起，程序没有退出。

```
RUNNABLE
RUNNABLE
```

这里说明thread没有正确处理interrupt信号，手动调用意思是想thread线程终止，那怎么处理呢？看下面



```java
public static void main(String[] args) throws InterruptedException {

        Thread thread = new Thread(() -> {
            while (true) {
                if (Thread.currentThread().isInterrupted()) {
                    System.out.println("线程被中断，game over");
                    break;
                }
            }
        });
        thread.start();
     	//中断前查看线程状态
        System.out.println(thread.getState().name());
     	//中断
        thread.interrupt();
        TimeUnit.SECONDS.sleep(3);
     	//中断后查看线程状态
        System.out.println(thread.getState().name());

    }
```

输出结果如下，程序退出。

```
RUNNABLE
线程被中断，game over
TERMINATED
```

这才是想要达到的效果，这里差别就是在被Interrupted线程里要做一个是否中断判断Thread.currentThread().isInterrupted()。如果是就让线程退出。



