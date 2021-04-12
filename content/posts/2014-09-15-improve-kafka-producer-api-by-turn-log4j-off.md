---
categories:
  - kafka
tags:
  - kafka
  - log4j
  - performance
title: 关闭kafka java api的log4j配置提升kafka producer性能
url: /2014/09/15/improve-kafka-producer-api-by-turn-log4j-off/
date: 2014-09-15
author: "Dongfang Qu"
---


## 背景

前段时间由于项目需要，基于官方提供的kafka 0.8.1.1版java api开发一个thrift接口的kafka代理层模块，我们暂且就称它为kafka-proxy吧，
主要功能是通过thrift接口接收外部系统的数据，根据规则转发给后端不同的kafka topic。

## 问题

做为kafka的代理层，QPS非常关键。thrift的server模型选择的TThreadedSelectorServer，在代码完成初期，压测结果很不理想，具体数据如下：

### 测试条件

1. cpu: 16 core
1. mem: 128G
1. network: 2 * 1G
1. message_size: 4KB

### 测试结果

1. cpu: 194% mem: 4G QPS: 1906
1. cpu: 230% mem: 4G QPS: 2213

不管怎么压，cpu、内存都占用不高，根据QPS * message_size也可知网卡更不是瓶颈。怎么回事儿呢？

## 定位

1. `top`
1. `free`
1. `jstack`

通过jstack dump出jvm进程，发现jvm多个kafka producer线程函数栈waiting for condition在log4j上，基本定位，而且从函数调用栈看起来好像还是默认使用的debug级别，我就擦了～

## 解决方法

在log4j.properties文件中关闭kafka日志。

    log4j.logger.kafka = OFF

此时的QPS可达到2W以上，基本是原来10倍，CPU和网卡成为模块瓶颈。洗洗睡觉～


