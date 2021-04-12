---
categories:
  - openjdk
  - jvm
tags:
  - openjdk
  - jvm
title: adoptopenjdk官方文档-进阶
url: /2015/09/01/adoptopenjdk-getting-started-advanced-steps/
date: 2015-09-01
author: "Dongfang Qu"
---


## 编译过程性能优化的命令行参数

> AdoptOpenJDK wiki的一个[链接](https://java.net/projects/adoptopenjdk/pages/BuildPerformanceOptimisation#Command-line_arguments_for_build_performance_optimisation)，提供了几个怎么提高编译过程性能的例子。

## 编译 jcov

### 项目首页(项目信息, 编译指南, 其他…) <br/>
> https://wiki.openjdk.java.net/display/CodeTools/jcov

### 源代码: <br/>
> http://hg.openjdk.java.net/code-tools/jcov

> 从[Adopt OpenJDK持续集成网站](https://adopt-openjdk.ci.cloudbees.com/job/jcov/lastSuccessfulBuild/artifact/)下载。

### 快速编译指南

> ```
> 
> $ hg clone http://hg.openjdk.java.net/code-tools/jcov
> $ cd jcov/build
> $ ant clean
> $ ant -v -f build.xml -Dasmjar5=/path/to/asm-all-5.0.1.jar -Djavatestjar=/path/to/javatest.jar
> 
> ```

- ```asm``` 可以从http://download.forge.ow2.org/asm/asm-5.0.1-bin.zip下载

- ```jtharness``` 可以从https://adopt-openjdk.ci.cloudbees.com/job/jtharness/lastSuccessfulBuild/artifact/下载

> 当项目主页有编译指南时，请参考项目主页，同时上面的编译编译指导会被删掉，避免重复。

> 请参考[OpenJDK代码覆盖率](openjdk_code_coverage.md).

## 编译 sigtest

### 项目主页(项目信息, 编译指南, 其他…)

> https://wiki.openjdk.java.net/display/CodeTools/sigtest

### 源代码: <br/>

> http://hg.openjdk.java.net/code-tools/sigtest

### 快速编译指南 <br/>

> ```
> 
> $ svn checkout https://svn.java.net/svn/sigtest~svn/trunk
> $ cd code/build
> $ ant build -Djdk5.home=/path/to/jdk1.5.latest \
> -Djdk8.home=/path/to/jdk8.latest \
> -Dmvn2.exe=/path/to/latest/bin/mvn
> 
> ```

> 当项目主页有编译指南时，请参考项目主页，同时上面的编译编译指导会被删掉，避免重复。

## OpenJDK代码覆盖率

> Adopt OpenJDK的代码覆盖率指南[code-coverage](https://java.net/projects/adoptopenjdk/pages/Codecoverage).

### 现存代码测试覆盖率报告 (OpenJDK8 和 OpenJDK9)

> 我们最近为运行在Adopt OpenJDK 编译集群上的 OpenJDK8和 OpenJDK9持续集成增加了代码测试覆盖率报告。自动为[OpenJDK8]
(https://adopt-openjdk.ci.cloudbees.com/view/OpenJDK/job/openjdk-1.8-linux-x86_64/ws/testoutput/jdk_core/JTreport/jcov/index.html)
和[OpenJDK9](https://adopt-openjdk.ci.cloudbees.com/view/OpenJDK/job/openjdk-1.9-linux-x86_64/ws/testoutput/jdk_core/JTreport/jcov/index.html)发布[jcov 报告].

### 运行OpenJDK9的代码测试覆盖率测试

#### 提示:
> * 同样的步骤也适用于'OpenJDK8'
> * 这些步骤仅用于生成'jdk'的测试覆盖率报告
> * 我们没有能够成功的生成'langtools'的测试覆盖率报告

> * 保证最新的```jdk```映像在```OpenJDK9```编译生成目录 (参见 [编译你自己的OpenJDK](../binaries/build_your_own_openjdk.md)).
> * 安装 ```jtreg with the jcov```, 参见[JTReg的使用](../intermediate-steps/preparations.md).
> * 把这些exports添加到你的```.bash_xxx```配置文件中:
>
> ```
> 
> export SOURCE_CODE=/home/<username>/workspace/jdk9/
> export JTREG_INSTALL=/home/<username>/workspace/jtreg
> export JT_HOME=$JTREG_INSTALL
> export JTREG_HOME=$JTREG_INSTALL
> export PRODUCT_HOME=$SOURCE_CODE/build/linux-x86_64-normal-server-release/images/jdk
> export JPRT_JTREG_HOME=${JT_HOME}
> export JPRT_JAVA_HOME=${PRODUCT_HOME}
> export JTREG_TIMEOUT_FACTOR=5
> export CONCURRENCY=8
> 
> ```
>
> ```
> $ cd $SOURCES/jdk9/jdk/test
> ```
>
> * 编辑 ```Makefile``` 文件，在  ```# Make sure jtreg exists```行之前增加如下行:

> ```
>
> jdkroot=<你的jdk9代码路径, 参见上面>
> 
> JTREG_TEST_OPTIONS += -jcov/classes:$(jdkroot)/build/linux-x86_64-normal-server-release/jdk/modules/java.base
> JTREG_TEST_OPTIONS += -jcov/source:$(jdkroot)/jdk/src/java.base/share/classes
> JTREG_TEST_OPTIONS += -jcov/include:*
> 
> ```

### debug 模式执行测试

> ```
> $ cd ..
> $ make test LOG=debug
> 
> ```

### 打开生成的测试覆盖率报告

> 一旦结束了, 进入如下路径查看报告:
> 
> ```
> $ cd $SOURCES/jdk9/build/linux-x86_64-normal-server-release/testoutput/jdk_core/JTreport/jcov/
> $ open index.html
> ```
> 
> 这会花费几个小时，具体时间取决于你的系统性能和可用资源。
> 
> 请参考jcov的编译。
> 
> 

## 深入 hotspot 的东西

> hotspot 源码目录中的GC选项

> ../hotspot/src/share/vm/gc_implementation/g1/g1_globals.hpp

> ../hotspot/src/share/vm/runtime/globals.hpp

> HotSpot 命令行选项 - PrintAssembly

> HotSpot 代码片段 - 由于不同GC选项引起的各种分支选择

| GC类型 | 老年代 | 老年代 |
|------------|----------------|--------------|
| SerialGC  (-XX:+UseSerialGC)|串行|串行 |
| ParallGC  (-XX:+UseParallelGC)|并行|串行|
| Parallel Compacting(-XX:+UseParallelOldGC)|并行|并行  |
| Concurrent Mark Sweep GC (-XX:+UseConcMarkSweepGC)|并行|并发-标记-清除 |

> [参考](http://www.weblogic-training.com/performance-tuning/difference-between-serial-gc-parallgc-cms-(-concurrent-mark-sweep)-gc/)
>
> 来自[./hotspot/src/share/vm/memory/universe.cpp](http://hg.openjdk.java.net/jdk6/jdk6/hotspot/raw-file/a541ca8fa0e3/src/share/vm/memory/universe.cpp)的代码片段

> ```java
>
> Universe::initialize_heap()
> if (UseParallelGC) {
>     #ifndef SERIALGC
>     Universe::_collectedHeap = new ParallelScavengeHeap();
>     #else // SERIALGC
>         fatal("UseParallelGC not supported in this VM.");
>     #endif // SERIALGC
> } else if (UseG1GC) {
>     #ifndef SERIALGC
>     G1CollectorPolicy* g1p = new G1CollectorPolicy();
>     G1CollectedHeap* g1h = new G1CollectedHeap(g1p);
>     Universe::_collectedHeap = g1h;
>     #else // SERIALGC
>         fatal("UseG1GC not supported in java kernel vm.");
>     #endif // SERIALGC
> } else {
>     GenCollectorPolicy* gc_policy;
>     if (UseSerialGC) {
>         gc_policy = new MarkSweepPolicy();
>     } else if (UseConcMarkSweepGC) {
>         #ifndef SERIALGC
>         if (UseAdaptiveSizePolicy) {
>             gc_policy = new ASConcurrentMarkSweepPolicy();
>         } else {
>             gc_policy = new ConcurrentMarkSweepPolicy();
>         }
>         #else // SERIALGC
>             fatal("UseConcMarkSweepGC not supported in this VM.");
>         #endif // SERIALGC
>     } else { // default old generation
>         gc_policy = new MarkSweepPolicy();
>     }
>     Universe::_collectedHeap = new GenCollectedHeap(gc_policy);
> }
> ```
> 
> <br/>

### 打开串行GC -  只支持串行GC的平台?

> ```java
> .
> .
> .
> Universe::initialize_heap()
> if (UseParallelGC) {
>         fatal("UseParallelGC not supported in this VM.");
> } else if (UseG1GC) {
>         fatal("UseG1GC not supported in java kernel vm.");
> } else {
>     GenCollectorPolicy* gc_policy;
>     if (UseSerialGC) {
>         gc_policy = new MarkSweepPolicy();
>     } else if (UseConcMarkSweepGC) {
>             fatal("UseConcMarkSweepGC not supported in this VM.");
>     } else { // default old generation
>         gc_policy = new MarkSweepPolicy();
>     }
>     Universe::_collectedHeap = new GenCollectedHeap(gc_policy);
> }
> .
> .
> .
> ```
> <br/>

### 关闭串行GC - 支持并行和串行两种GC方式的平台?

> ```java
> .
> .
> .
> Universe::initialize_heap()
> 
> if (UseParallelGC) {
>     Universe::_collectedHeap = new ParallelScavengeHeap();
> } else if (UseG1GC) {
>     G1CollectorPolicy* g1p = new G1CollectorPolicy();
>     G1CollectedHeap* g1h = new G1CollectedHeap(g1p);
>     Universe::_collectedHeap = g1h;
> } else {
>     GenCollectorPolicy* gc_policy;
> 
>     if (UseSerialGC) {
>         gc_policy = new MarkSweepPolicy();
>     } else if (UseConcMarkSweepGC) {
>         if (UseAdaptiveSizePolicy) {
>             gc_policy = new ASConcurrentMarkSweepPolicy();
>         } else {
>             gc_policy = new ConcurrentMarkSweepPolicy();
>         }
>     } else { // default old generation
>         gc_policy = new MarkSweepPolicy();
>     }
> 
>     Universe::_collectedHeap = new GenCollectedHeap(gc_policy);
> }
> .
> ```

## 编译器相关

> OptViewer工具，参考这个[邮件](http://mail.openjdk.java.net/pipermail/hotspot-compiler-dev/2013-February/009778.html)。

## 修改 java.c，使用 Eclipse运行 hotspot

> [可选，但很有意思的一个挑战]
> 请参考[这些](http://bit.ly/12LxuQy)指南。
> 
> ## 修改 java.c，使用命令行运行 hotspot
> 
> [可选，但很有意思的一个挑战]
> 
> 和修改java.c并在 Eclipse运行的挑战类似，但是使用需要使用命令行和一个简单的编辑器来完成这一挑战。


## 敬请期待

- Nashorn的一些好东西
- Lambda进阶
- OpenJDK(JDK)代码测试覆盖率工具（两个）
- OpenJDK build warnings tool (which are currently suppressed in the build process)...
- OpenJDK 编译警告工具（当前在编译过程中被禁用）
- 使用 jitWatch观察 HotSpot JVM  JIT编译过程
- 运行在 JVM 上的Smalltalk
- 运行在 JVM 上的Lisp

## Hotspot JVM 任务: 附加任务 (中级和高级)

- 向 java.c 中增加 debug 级别的 log信息，重新编译 gamma，运行示例程序或者任何其他基于 java 的程序。
- 重构 java.c，插入 debug 级别的 Log细腻，重新编译 gamma，运行示例程序或者任何其他基于 java 的程序。
- 完成以上两步后，加载一个低延迟，GC 调优过的 java 程序，开启 gc log，检查 gc log，看性能是否有变化（性能调优相关的东西）。
- 在 javac 中实现 ?:运算符(学习怎么修改 javac 的好方法)，编译一个java 程序。
- 体味GC 的乐趣：使用自定义的垃圾收集器替换已有的。重新激活代码库中的 PermGen 或者 iCMS 。在现在 HotSpot 版本中增加任意你想做的修改。
- 修改 javac 使他能够解析和编译新的语言特性，或者让它能够兼容其他基于 JVM 的方言、甚至于更老的程序语言，例如 C、汇编、Scheme 或Smalltalk
- 使用自定义类加载器替换内置内加载器。

