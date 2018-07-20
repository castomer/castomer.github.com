---
layout: single
title: "日志数据采集客户端logstash和fluentd的比较"
categories: logstash fluentd log
tags: logstash fluentd log


---

[logstash](http://logstash.net/)和[fluentd](http://fluentd.org/)是ruby圈的log数据采集工具，功能类似于scribed、flume，相比较而言logstash和fluentd架构设计更漂亮，生态圈更丰富一些。
近期先后阅读了fluentd和logstash的源码，大概的一些信息总结如下：

## 特点：
1. 开发语言ruby / jruby
2. 构清晰，扩展方便
3. 功能多，生态圈丰富
4. 性能一般
5. 依赖ruby/java

## fluentd
1. 基本概念：input buffer output
1. format:json
1. 数据流：input →  (buffer) → output
1. 架构：input →  output，output分buffered output和non buffered ouput

## logstash
1. 基本概念：codec input filter output
1. format:由codec负责
1. 数据流：input →  codec.decode →  filter →  codec.encode →  output

## 区别
1. fluentd无codec概念，有比较完善的重试和缓存机制，数据的处理交给了input来完成，input层功能不单一，input主动或者被动都行
1. logstash架构上更清晰，input filter output各自完成的工作划分比较清楚，重试和缓存机制由output负责，input被动的由logstash调度
1. logstash性能优一些
