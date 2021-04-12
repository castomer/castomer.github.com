---
categories:
  - tomcat
tags:
  - tomcat
title: tomcat连接器选择
url: /2015/06/03/tomcat-connector-select/
date: 2015-06-03
author: "Dongfang Qu"
---


## 推荐
- nio2(tomcat 7 only)
- nio

## 配置方法

1. tomcat 8: `protocol="org.apache.coyote.http11.Http11Nio2Protocol"`
1. tomcat 7: `protocol="org.apache.coyote.http11.Http11NioProtocol"`

## 参考
[参考](http://tomcat.apache.org/tomcat-8.0-doc/config/http.html)