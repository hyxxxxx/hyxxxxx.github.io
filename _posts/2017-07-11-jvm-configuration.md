---
layout: post
title: "JVM参数配置"
date: 2017-07-11
excerpt: "A ton of text to test readability."
tags: 
- JVM
- Java
comments: true
---


### 堆参数配置
- -Xms：设置java程序启动时初始堆大小
- -Xmx：设置java程序能获得的最大堆大小
- -Xmn：可以设置新生代大小，一般会设置整个堆空间的1/3到1/4左右
- -XX:SurvivorRatio：用来设置新生代中eden空间和from/to空间的比例，值就是eden/from或者eden/to的比值
- -XX:NewRatio：设置老年代和新生代的比例，值就是老年代/新生代
- -XX:+HeapDumpOnOutOfMemoryError：使用该参数可以在内存溢出时导出整个堆信息
- -XX:HeapDumpPath：设置导出堆的存放路径
- -XX:+PrintCommandLineFlags：可以将隐式或者显示传给虚拟机的参数输出
- -XX:+UseAdaptiveSizePolicy：打开**自适应模式**，这种模式下，新生代大小、eden、from/to比例以及晋升老年代的年龄参数会被自动调整，以达到各个参数之间的平衡点（一般只用这一个参数就可以了）

> **可以直接将初始的堆大小与最大堆大小设置相等，好处在于可以减少程序运行时的垃圾回收次数，从而提高性能。**

> 例如：Xmx20m -Xms20m  -XX:+UseAdaptiveSizePolicy（自适应模式）

### 栈参数配置
- -Xss：指定线程的最大栈空间，整个参数也直接决定了函数可调用的最大深度

### 方法区参数配置
- -XX:MaxPermSize：配置最大方法区大小（默认是64m），这是值根据系统运行时有多少类产生来预估

### GC参数配置
- -XX:+PrintGC：虚拟器启动后，只要遇到GC就会打印日志
- -Xloggc:PATH：指定GC打印日志保存的路径（PATH:绝对路径）
- -XX:+PrintGCDetails：可以查看详细信息，包括各个区的情况
- -XX:MaxTenuringThreshold：配置经历多少次gc后进入老年代（默认是15次）
- -XX:PretenureSizeThreshold：当对象超过设置的大小时，直接进入老年代
- -XX:ParallelGCThreads：指定ParNew回收器工作时的线程数量，最好和计算机的CPU相当
- -XX:MaxGCPauseMillis：设置最大垃圾收集停顿时间
- -XX:GCTimeRatio：设置吞吐量大小，是0到100之间的整数（默认是99）
- -XX:+ParallelGCThreads：设置垃圾收集时的线程数量
#### 垃圾回收器种类
		-XX:+UseSerialGC：使用串行回收器
		-XX:+UseParNewGC：新生代使用ParNew并行回收器，老年代默认用串行回收器
		-XX:UseParallelGC：新生代使用复制算法的GC，它非常关注系统的吞吐量（高并发业务使用）
		-XX:UseParallelOldGC：老年代使用标记压缩法的回收器，也关注吞吐量
		-XX:+UseConcMarkSweepGC：CMS回收器（目前主流），并发标记清除，主要关注系统停顿时间，因为这不是独占回收器，所以要保证内存够用。它不会等到应用程序饱和才去回收，而是在指定阈值时回收
		-XX:+UseG1GC：G1回收器（jdk1.7后提出的），属于分代垃圾回收器，区分新生代和老年代，使用了分区算法

------------


> CMS回收器（目前主流）：并发标记清除，主要关注系统停顿时间，因为这不是独占回收器，所以要保证内存够用。它不会等到应用程序饱和才去回收，而是在指定阈值时回收
- -XX:+UseConcMarkSweepGC：开启
- -XX:ConcGCThreads：设置并发线程数量
- -XX:CMSInitiatingOccupancyFraction：指定垃圾收集工作的阈值（默认是68）
- -XX:+UseCMSCompactAtFullCollection：可以设置CMS回收完成后进行一次碎片整理
- -XX:CMSFullGCsBeforeCompaction：设置进行多少次CMS回收后，对内存进行一次压缩

> G1回收器（jdk1.7后提出的）：属于分代垃圾回收器，区分新生代和老年代，使用了分区算法
- -XX:+UseG1GC
- -XX:MaxGCPauseMillis
- -XX:ParallelGCThreads
