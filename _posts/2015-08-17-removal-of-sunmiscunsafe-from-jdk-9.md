---
layout: single
title: "Java 9中将移除 Sun.misc.Unsafe(译)"
categories: java 9, Unsafe
tags: java9, Unsafe，翻译


---

灾难将至，Java 9中将移除 ```Sun.misc.Unsafe```


Oracle 正在[计划在Java 9中去掉 ```sun.misc.Unsafe```](https://docs.google.com/document/d/1GDm_cAxYInmoHMor-AkStzWvwE9pw6tnz_CebJQxuUE/edit?pli=1) API。
这绝对将是一场灾难，有可能会彻底破坏整个 java 生态圈。
几乎每个使用 java开发的工具、软件基础设施、高性能开发库都在底层使用了 ```sun.misc.Unsafe```。
如下是上面链接中文档提到部分使用```Unsafe```的软件或 lib：

> - Netty
> - Hazelcast
> - Cassandra
> - Mockito / EasyMock / JMock / PowerMock
> - Scala Specs
> - Spock
> - Robolectric
> - Grails
> - Neo4j
> - Spring Framework
> - Akka
> - Apache Kafka
> - Apache Wink
> - Apache Storm
> - Apache Hadoop
> - Apache Continuum

... 这个列表很长。。。


然而， Oracle 看起来是铁了心毫无理由的去掉它。下面是一个来自他们[邮件列表](http://mail.openjdk.java.net/pipermail/openjfx-dev/2015-April/017028.html)的评论：
n
> 恕我直言 — ```sun.misc.Unsafe``` 必须死掉。 它是“不安全”的。它必须被废弃。请忽略一切理论上(想象中的)羁绊，从此走上正确的道路吧。


这个工程师似乎是毫无根据的憎恨 Unsafe。。。

## Oracle应该怎么做？
当前Unsafe 类是一个强有力的工具。
没有必要去掉它。对这个类的特性有些明确的需求，这就是为什么事实上几乎每个 Java 程序都在使用它，不知不觉中许多流行的 Java库也在使用它。

### 提供完整的文档、发布 Unsafe 类
Oracle 应该接受现实，并将Unsafe转为公开 API，提供完善的文档和开发示例。
当前，没有准确的文档，开发中需要通过 [stackoverflow 帖子](http://stackoverflow.com/questions/5574241/are-there-real-world-uses-of-sun-misc-unsafe)或者其他一些随机的博客学习怎么使用 ```Unsafe```。
移除 ```Unsafe``` 的一个主要论据是：使用它太容易让开发中犯错了。如果有完善的官方文档或许可以改善这一现状。

### 随 Unsafe一起发布新的替代 API
除了 ```Unsafe``` 文档外，Oracle 应该发布一个更易用的 API，提供 ```Unsafe``` 相同的功能。
这是上面文档中的提议的一部分。然而这不太应该以移除 ```Unsafe``` 为代价。
人们在开发新软件的时候就会逐步过渡到新的 API，```Unsafe``` 就自动被废弃了。

这类似于向 Java 8引入 ```java.time``` 包中的新的 ```DateTime``` API。
新的日期 API 的引入并不表示之前的```DateTime``` API 被彻底移除或者隐藏到某个特殊 JVM flag 里。那样也肯定会引发一些事故。

## 实际上最可能会变成什么样子？

根据事情的发展趋势，Oracle 看起来会：

> 1. 在 Java 9正常模式下移除 ```Unsafe``` 类。
> 1. 仅在必须的情况下通过向 JVM 传递一个特殊的 flag 启动 ```Unsafe```

## 这将导致绝对的灾难！

* 不仅类似 Cassandra 或Zookeeper 等基础软件，几乎所有的 Java 程序，包括 web 应用也会挂掉，因为他们使用的基础库可能在底层使用了 ```Unsafe```。
* 从此打开 Unsafe  flag 将会成为启动 JVM 的默认 flag 之一，因为如果不打开它的话 Java 应用会在毫无提示的情况下崩溃。
* 因为大多数环境不会默认把这个JVM flag 打开，当他们的系统升级 Java时软件系统会挂掉。
Java 打破了向后兼容的承诺。所有的基础库、软件基础设施从此变为两个版本：
    - Java 9之前的版本 - 使用 ```Unsafe```
    - Java 9兼容 - 不使用 ```Unsafe```。
* 迁移至 Java 9的进程会因此而变缓慢，这将影响整个 Java 生态系统。这将会类似于 Python 2升级到 Python 3的过程。

## 这种错误 JVM 社区之前曾经犯过

你是不是任务这太荒唐了，Oracle 绝不可能犯这样的错误？事实上它曾做过类似的事情了，
例如[Java 7中的字节码校验器](http://chrononsystems.com/blog/java-7-design-flaw-leads-to-huge-backward-step-for-the-jvm)。

## 结论

现在是该让大家开始意识到这个问题的时候了。从 JVM中去掉```Unsafe```或者把它隐藏在某个特殊的 flag 里面势必导致一场灾难。


## 参考链接
- [原文](http://blog.dripstat.com/removal-of-sun-misc-unsafe-a-disaster-in-the-making/)
- [infoq 报道](http://www.infoq.com/cn/news/2015/08/oracle-plan-remove-unsafe)
- [itupup 报道](http://www.itupup.com/?it08/523957.htm)
