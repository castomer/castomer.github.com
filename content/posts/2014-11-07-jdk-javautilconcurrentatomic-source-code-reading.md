---
categories:
  - jdk
  - concurrent
  - atomic
tags:
  - jdk
  - atomic
  - concurrent
title: jdk 1.8 java.util.concurrent.atomic.* 源码阅读
url: /2014/11/07/jdk-javautilconcurrentatomic-source-code-reading/
date: 2014-11-07
author: "Dongfang Qu"
---


## 代码

1. version: 1.8
1. code path: jdk8u/jdk/src/share/classes/java/util/concurrent/atomic

## 收获

1. Atomic* 由 Unsafe实现
1. Unsafe: 声明一些native方法，jdk8u/jdk/src/share/classes/sun/misc/Unsafe.java
1. AtomicLong等比较大的数据类型，在系统层面不支持的时候采用java lock实现 // VMSupportsCS8()
1. weakCompareAndSet和CompareAndSet实现一致，不懂为什么注视说二者行为会不同
1. AtomicIntegerArray && AtomicLongArray, 位运算+Atomic*
1. Atomic*FieldUpdater 原子更新其他类的field, 反射 + Unsafe
1. LongAccumulator && DoubleAccumulator @since 1.8 高性能累加器，核心代码Striped64.java，将写压力分摊到多个cell，适用于与ConcurrentHashMap结合来统计状态，不能做key
1. LongAdder && DoubleAdder @since 1.8 只能适用于做sum && count, *Accumulator的简化版；LongAdder -> LongAccumulator((x, y) -> x + y, 0L)

