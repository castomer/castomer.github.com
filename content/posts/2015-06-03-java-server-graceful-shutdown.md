---
categories:
  - java
tags:
  - java
title: java server graceful shutdown
url: /2015/06/03/java-server-graceful-shutdown/
date: 2015-06-03
author: "Dongfang Qu"
---


### 目的
避免kill -9导致内存中数据丢失，导致业务数据不一致的情况

### 实践
   
    Runtime.getRuntime().addShutdownHook(new Thread() {
        public void run() {
                // do sth
        }
    });

### 参考
- [1](http://blog.csdn.net/wgw335363240/article/details/5854402)

