---
layout:     post
title:      JVM Garbage Collectors
subtitle:   Introduction to different kind of GC collectors
date:       2019-04-02
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

## 1. Overview
In this quick tutorial, we will show the basics of different JVM Garbage Collection (GC) implementations. Additionally, we’ll find out how to enable a particular type of Garbage Collection in our applications.

## 2. Brief Introduction to Garbage Collection
From the name, it looks like _Garbage Collection_ deals with finding and deleting the garbage from memory. However, in reality, _Garbage Collection_ tracks each and every object available in the JVM heap space and removes unused ones.

In simple words, _GC_ works in two simple steps known as Mark and Sweep:
- __Mark__ – it is where the garbage collector identifies which pieces of memory are in use and which are not
- __Sweep__ – this step removes objects identified during the “mark” phase

__Advantages:__
- No manual memory allocation/deallocation handling because unused memory space is automatically handled by _GC_
- No overhead of handling [_Dangling Pointer_](https://en.wikipedia.org/wiki/Dangling_pointer)
- Automatic [_Memory Leak_](https://en.wikipedia.org/wiki/Memory_leak) management (GC on its own can’t guarantee the full proof solution to memory leaking, however, it takes care of a good portion of it)

__Disadvantages:__
- Since JVM has to keep track of object reference creation/deletion, this activity requires more CPU power besides the original application. It may affect the performance of requests which required large memory
- Programmers have no control over the scheduling of CPU time dedicated to freeing objects that are no longer needed
- Using some GC implementations might result in application stopping unpredictably
- Automatized memory management will not be as efficient as the proper manual memory allocation/deallocation

## 3. GC Implementations
JVM has four types of _GC_ implementations:
- Serial Garbage Collector
- Parallel Garbage Collector
- CMS Garbage Collector
- G1 Garbage Collector

### 3.1. Serial Garbage Collector
This is the simplest GC implementation, as it basically works with a single thread. As a result, __this *GC* implementation freezes all application threads when it runs.__ Hence, it is not a good idea to use it in multi-threaded applications like server environments.

However, there was [an excellent talk](https://www.infoq.com/presentations/JVM-Performance-Tuning-twitter-QCon-London-2012) by _Twitter_ engineers at QCon 2012 on the performance of _Serial Garbage Collector_ – which is a good way to understand this collector better.

The Serial GC is the garbage collector of choice for most applications that do not have small pause time requirements and run on client-style machines. To enable _Serial Garbage Collector_, we can use the following argument:
```java
java -XX:+UseSerialGC -jar Application.java
```

### 3.2. Parallel Garbage Collector
It’s the default _GC_ of the JVM and sometimes called Throughput Collectors. Unlike _Serial Garbage Collector_, this __uses multiple threads for managing heap space__. But it also freezes other application threads while performing GC.

If we use this GC, we can specify maximum garbage collection _threads_ and _pause time_, _throughput_ and _footprint_ (heap size).

The numbers of garbage collector threads can be controlled with the command-line option _-XX:ParallelGCThreads=<N>_.

The maximum pause time goal (gap [in milliseconds] between two _GC_)is specified with the command-line option _-XX:MaxGCPauseMillis=<N>_.

The maximum throughput target (measured regarding the time spent doing garbage collection versus the time spent outside of garbage collection) is specified by the command-line option _-XX:GCTimeRatio=<N>_.

Maximum heap footprint (the amount of heap memory that a program requires while running) is specified using the option _-Xmx<N>_.

To enable _Parallel Garbage Collector_, we can use the following argument:
```java
java -XX:+UseParallelGC -jar Application.java
```

### 3.3. CMS Garbage Collector
The _Concurrent Mark Sweep (CMS)_ implementation uses multiple garbage collector threads for garbage collection. It’s designed for applications that prefer shorter garbage collection pauses, and that can afford to share processor resources with the garbage collector while the application is running.

Simply put, applications using this type of GC respond slower on average but do not stop responding to perform garbage collection.

A quick point to note here is that since this GC is concurrent, an invocation of explicit garbage collection such as using _System.gc()_ while the concurrent process is working, will result in [Concurrent Mode Failure / Interruption](https://blogs.oracle.com/jonthecollector/entry/what_the_heck_s_a).

If more than 98% of the total time is spent in _CMS_ garbage collection and less than 2% of the heap is recovered, then an _OutOfMemoryError_ is thrown by the _CMS collector_. If necessary, this feature can be disabled by adding the option _-XX:-UseGCOverheadLimit_ to the command line.

This collector also has a mode knows as an incremental mode which is being deprecated in Java SE 8 and may be removed in a future major release.

To enable the _CMS Garbage Collector_, we can use the following flag:
```java
java -XX:+UseParNewGC -jar Application.java
```

### 3.4. G1 Garbage Collector
_G1 (Garbage First) Garbage Collector_ is designed for applications running on multi-processor machines with large memory space. It’s available since _JDK7 Update 4_ and in later releases.

_G1 collector_ will replace the _CMS collector_ since it’s more performance efficient.

Unlike other collectors, _G1 collector_ partitions the heap into a set of equal-sized heap regions, each a contiguous range of virtual memory. When performing garbage collections, G1 shows a concurrent global marking phase (i.e. phase 1 known as _Marking_) to determine the liveness of objects throughout the heap.

After the mark phase is completed, _G1_ knows which regions are mostly empty. It collects in these areas first, which usually yields a significant amount of free space (i.e. phase 2 known as _Sweeping_). It is why this method of garbage collection is called Garbage-First.

To enable _G1 Garbage Collector_, we can use the following argument:
```java
java -XX:+UseG1GC -jar Application.java
```

### 3.5. Java 8 Changes
_Java 8u20_ has introduced one more _JVM_ parameter for reducing the unnecessary use of memory by creating too many instances of same _String_. This optimizes the heap memory by removing duplicate _String)_ values to a global single _char[]_ array.

This parameter can be enabled by adding __*-XX:+UseStringDeduplication*__ as _JVM_ parameter.

## 4. Conclusion
In this quick tutorial, we had a look at the different JVM Garbage Collection implementations and their use cases.

More detailed documentation can be found [here](http://www.oracle.com/technetwork/java/javase/gc-tuning-6-140523.html).
