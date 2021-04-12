---
categories:
  - docker
  - java
  - jvm
  - gc
tags:
  - docker
  - java
  - jvm
  - gc
title: 容器(docker)中运行java需关注的几个小问题
url: /2018/07/19/pitfalls-about-running-java-inside-container/
date: 2018-07-19
author: "Dongfang Qu"
---



# 简介

- container： 资源隔离、平台无关， 限制cpu、mem等资源
- Java： 平台无关、[Write once, run anywhere、WORA](https://zh.wikipedia.org/wiki/%E4%B8%80%E6%AC%A1%E7%BC%96%E5%86%99%EF%BC%8C%E5%88%B0%E5%A4%84%E8%BF%90%E8%A1%8C)


        java不知道自己运行在container里，以为它看到的资源都能用。结果：java工作在资源充足的
        
# 详述

    程序运行的两个核心资源：cpu和mem，其他资源或许也有限制，暂不涉及。

## cpu
   jvm检测可用cpu个数来优化运行时，影响jvm后台做的一些决策。

### 影响

- [java.lang.Runtime](https://docs.oracle.com/javase/8/docs/api/java/lang/Runtime.html) 所以ump的jvm监控数据一直以来都是不准确的
- [GC](https://blogs.oracle.com/java-platform-group/java-se-support-for-docker-cpu-and-memory-limits) 主要是线程数
- [JIT](https://en.wikipedia.org/wiki/Just-in-time_compilation) [代码编译/执行子系统优化](https://blogs.oracle.com/java-platform-group/java-se-support-for-docker-cpu-and-memory-limits)

## 实验

    实验所用容器宿主机器是4核CPU16G内存


- java 6/7/8/9

        docker run --cpus 1 -m 1G -it adoptopenjdk/openjdk9:latest # 给1核

        jshell -J-Xmx512M -v # 启动jshell

        Runtime.getRuntime().availableProcessors() # 结果是不是1！！！

- java 10

        docker run --cpus 1 -m 1G -it adoptopenjdk/openjdk10:latest # 给1核

        jshell -J-Xmx512M -v # 启动jshell

        Runtime.getRuntime().availableProcessors() # 结果是1


### 对策

    java 10才解决这个问题

- java 10之前：手动设置jvm相关的选项，如：
    - ParallelGCThreads
    - ConcGCThreads
    - G1ConcRefinementThreads
    - CICompilerCount / CICompilerCountPerCPU
    
- java 8u191+ UseContainerSupport， 默认开启，backported java10的feature；java 9暂未backport这个feature，估计也过不了多久

- java 10+:
    - UseContainerSupport， 默认开启

## mem

    jvm自动检测拿到的是宿主机的内存信息，它无法感知容器的资源上限
    主要需要关注：动态内存管理(上下限，默认值)

    我们先了解下java进程内存消耗在哪里


### 内存结构

 JFTR: 分代:垃圾收集的一大策略，并不是所有GC算法都分代哦

内存总量 = 广义堆内存 + 广义堆外内存

	广义堆内内存 = 狭义堆内内存 + 永久代(Perm)
		狭义对内内存 = 新生代(New) + 老年代(Old) # Xmx Xms
		新生代(New) = S0 + S1 + Eden # NewSize NewRatio SurvivorRatio

        广义堆外内存 = 狭义堆外内存(directbytebuffer)  # MaxDirectMemorySize，netty/mina等高性能网络通信常用，具体不是很了解
                        + java栈 # 需关注，线程数 * ThreadStackSize(Xss)
                        + native栈 # 不大，线程数 * VMThreadStackSize + CICompilerCount * CompilerThreadStackSize
                        + pc寄存器  # 可忽略
                        + jni（如带用c/c++ malloc）# 这个不可控，一般忽略就好


        java 8之后Metaspace替代了Perm，从广义堆内转移到广义对外，why?
            - Perm连续、固定、jvm启动即claim到max，浪费，不好控制
            - 代码实验发现Perm不受Xmx限制`java -Xmx10M -Xmx10M -XX:PermSize=100M -XX:MaxPermSize=100M -version`
            - Metaspace的实现类似链式结构，默认值-1(无限大，取决于kernel让用多少)，在fullgc时gc

        上面关于广义/狭义内存的定义是参考别人的资料后，自己定义的。。。不喜勿喷

            - 官方对于Perm的定义摇摆不定，前后矛盾，让我自己很困惑

                正方：在
                    http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html
                    http://www.oracle.com/technetwork/java/javase/memorymanagement-whitepaper-150215.pdf
                    http://karunsubramanian.com/websphere/one-important-change-in-memory-management-in-java-8/

                反方：不在
                    https://docs.oracle.com/javase/7/docs/technotes/guides/management/jconsole.html
                    https://stackoverflow.com/questions/1262328/how-is-the-java-memory-pool-divided
                    https://blogs.oracle.com/jonthecollector/presenting-the-permanent-generation
                    https://www.journaldev.com/2856/java-jvm-memory-model-memory-management-in-java
                    https://www.yourkit.com/docs/kb/sizes.jsp
                    https://blog.codecentric.de/en/2010/01/the-java-memory-architecture-1-act/

            - 与Metaspace替换Perm有点儿关系吧


综上，我们需要关注下面几类参数是否合理：
        - 狭义堆内 Xmx
        - 狭对堆外 MaxDirectMemorySize
        - Perm/Metaspace MaxPermSize/MaxMetaspaceSize


### 需关注的选项默认值

    - Xmx: 1/4 * 物理内存 # 此处的物理内存为Runtime看到的内存(大多时候是宿主机的内存)
    - MaxDirectMemorySize
        - Xmx 未设置，物理内存
        - Xmx 设置了， Xmx - S0(1/10 * Xmx) = 0.9 * Xmx # why? SurvivorRatio默认值8
    - MaxPermSize: 默认64M
            [5.0+ 64 bit: 64M * 1.3 = 85M](http://www.oracle.com/technetwork/java/javase/tech/vmoptions-jsp-140102.html)
    - MaxMetaspaceSize: -1，无限制

### 实验

实验所用容器宿主机器是4核CPU16G内存

- java 7

        docker run -m 1G -it openjdk:7u181
        java -XX:+PrintFlagsFinal -version | grep MaxHeapSize # 结果是 16G / 4 = 4G

- java 8

        docker run -m 1G -it adoptopenjdk/openjdk8:latest
        java -XX:+PrintFlagsFinal -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -version | grep MaxHeapSize # 结果是 1G / 4 = 256M

- java 9

        docker run -m 1G -it adoptopenjdk/openjdk9:latest
        java -XX:+PrintFlagsFinal -version | grep MaxHeapSize # 结果是 16G / 4 = 4G
        java -XX:+PrintFlagsFinal -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -version | grep MaxHeapSize # 结果是 1G / 4 = 256M

- java 10

        docker run -m 1G -it adoptopenjdk/openjdk10:latest # 给1G
        jshell -v # 启动jshell
        java -XX:+PrintFlagsFinal -version | grep MaxHeapSize  # 结果是 1G / 4 = 256


### 对策

- java5/6/7/8u131-：务必设置内存选项

        懒人可考虑，虽然也不准确， 参考前面对jvm内存结构的分析
        java -Xmx`cat /sys/fs/cgroup/memory/memory.limit_in_bytes`

- java8u131+和java9+

    - java 8u131+和java 9+`-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap`
    - java 8u191+ UseContainerSupport默认开启，backported；java 9暂未backport这个feature

- java10+
    - 使用最新版就好了，UseContainerSupport默认开启


# 扩展

### 排查工具

- jvm支持的选项

    - 生产

            java -XX:+PrintFlagsFinal 2>/dev/null

    - 试验

            java -XX:+PrintFlagsFinal -XX:+UnlockExperimentalVMOptions 2>/dev/null | grep experimental

    - 可热更新

            java -XX:+PrintFlagsFinal -XX:+UnlockExperimentalVMOptions 2>/dev/null | grep manageable

            如：热开启gc日志

                    jinfo -flag +PrintGC ${pid} # 官方文档说jinfo是实验工具，截至java 10，它都还在，[不过jhat在java9被去掉了](http://openjdk.java.net/jeps/241)
                    jinfo -flag +PrintGCDetails ${pid}

- 容器内执行jstat/jps/jmapOOM问题

        工具类是用C++包装的java代码，它们不识别常规的传给jvm的参数，如最大内存。
        在强悍的(超级大内存)宿主机器下，容器内经常因为OOM问题启动不了这些工具。

    - java 6及之前: 通过java调用

            java -cp ${JAVA_HOME}/lib/tools.jar -Xmx100M sun.tools.jstack.JStack ${pid}

    - java 7+

            jstack -J-Xmx100M -v # -J选项给jvm传参数

# Ref
- https://bugs.java.com/view_bug.do?bug_id=JDK-8146115
- http://royvanrijn.com/blog/2018/05/java-and-docker-memory-limits/
- https://blog.docker.com/2018/04/improved-docker-container-integration-with-java-10/
- https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html
- https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5
- http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html
- https://blogs.oracle.com/java-platform-group/java-se-support-for-docker-cpu-and-memory-limits
- http://hg.openjdk.java.net/jdk/jdk/file/03f2bfdcb636/src/hotspot/os/linux/osContainer_linux.cpp
- http://hg.openjdk.java.net/jdk/jdk/file/03f2bfdcb636/src/hotspot/os/linux/globals_linux.hpp#l62