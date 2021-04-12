---
categories:
  - jvm
  - graal
tags:
  - jvm
  - grall
title: ' 来自oracle labs 的虚拟机探索工具graal'
url: /2015/07/18/graal-compiler-from-oracle-labs/
date: 2015-07-18
author: "Dongfang Qu"
---


工作中 java 用的多一些， 难免想深入了解jvmn；搞下来 openjdk 8/9 的代码自己编译下，随便翻一翻，终究没啥大的收获。
前几天随意翻openjdk 文档，看到了个叫 graal的东西。就尝试搞一下看，能不能学点儿东西。

###  graal 是什么

一个使用 java 写的 just in time 编译器。一个探索 jvm 内部机制的工具。可以配合 HotSpot、Maxine、J9 等其他虚拟机共同使用。
 它利用一种程序内部的表现方式允许一些更高效的优化。近期的目标是通过向量（intel 最新的向量指令集）和一些激进的内存优化来提供编译器的性能峰值。

### 设计目标

该项目的目标是使用 java api 探索 虚拟机的功能。我们想让使用 java 来编写其他语言的动态编译器和解释器简单易用。
该组件将充分利用现存 jvm 其他基础设施（如 hotspot)，并且可以与其无缝结合。
使用 java特性使动态编译器的设计非常具有扩展性，增加其他的执行节点或者指令转换都是非正常简单直接。

与此同时，在虚拟机不消耗过多编译时间和内存的情况下编译出高质量的字节码。
在这个编译器的基础上，我们目标是开发出一个多语言执行框架，Java 将只是这个框架所支持的众多编程语言中的一种。
部分求值的特性将允许该框架达到更高的性能。

### 参考

- [graal 主页](http://openjdk.java.net/projects/graal/)
- [graal wiki](https://wiki.openjdk.java.net/display/Graal/Main)
- [graal compiler](http://ssw.jku.at/Research/Projects/JVM/Graal.html)