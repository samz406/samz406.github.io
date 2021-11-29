---
title: JVM参数配置总结
date: 2021-08-25 20:33:26
tags: [技术]
categories: JAVA
---

JVM参数总结

<!-- more -->

### 启动参数

| 参数                          | 说明                                                       |
| ----------------------------- | ---------------------------------------------------------- |
| -verbose:gc                   | 打印 GC 日志                                               |
| PrintGCDetails                | 打印详细 GC 日志                                           |
| PrintGCDateStamps             | 系统时间，更加可读，PrintGCTimeStamps 是 JVM 启动时间      |
| PrintGCApplicationStoppedTime | 打印 STW 时间                                              |
| PrintTenuringDistribution     | 打印对象年龄分布，对调优 MaxTenuringThreshold 参数帮助很大 |
| loggc                         | 将以上 GC 内容输出到文件中                                 |
| HeapDumpOnOutOfMemoryError    | OutOfMemory 时Dump的日志                                   |
| HeapDumpPath                  | Dump文件保存路径                                           |
| ErrorFile                     | 错误日志路径                                               |

```
-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps 
-XX:+PrintGCApplicationStoppedTime -XX:+PrintTenuringDistribution 
-Xloggc:/usr/logs/gc.log -XX:+HeapDumpOnOutOfMemoryError 
-XX:HeapDumpPath=/tmp/logs -XX:ErrorFile=/usr/logs/hs_error.log 

```





### 命令

- jmap

 jmap 主动去获取内存的快照

```
jmap -dump:format=b,file=heap.bin 122222
```



- jstat

  查看gc分配情况

jstat -gcutil xxx

 



| 参数 | 说明                                     |      |
| ---- | ---------------------------------------- | ---- |
| S0   | S0空间利用率占当前空间容量的百分比       |      |
| S1   | S1空间利用率占当前空间容量的百分比       |      |
| E    | eden区空间利用率占当前空间容量的百分比   |      |
| O    | 老年代空间利用率占当前空间容量的百分比   |      |
| M    | 元空间空间利用率占当前空间容量的百分比。 |      |
| CSS  | 压缩类空间利用率百分比                   |      |
| YGC  | Yong gc 次数                             |      |
| YGCT | Young gc 时间                            |      |
| FGC  | full gc 次数                             |      |
| FGCT | full gc垃圾收集时间                      |      |
| GCT  | 垃圾收集总时间                           |      |











### 收集器设置



| 参数                    | 说明                     |
| ----------------------- | ------------------------ |
| -XX:+UseSerialGC        | 设置串行收集器           |
| -XX:+UseParallelGC      | 设置并行收集器           |
| -XX:+UseParallelOldGC   | 老年代使用并行回收收集器 |
| -XX:+UseParNewGC        | 新生代使用并行收集器     |
| -XX:+UseConcMarkSweepGC | CMS并发收集器            |
| -XX:+UseG1GC            | G1收集器                 |



