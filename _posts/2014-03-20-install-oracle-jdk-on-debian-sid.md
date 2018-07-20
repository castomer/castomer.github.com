---
layout: single
title: "debian安装Oracle(Sun) jdk 8"
categories: debian java
tags: java jdk


---

在debian linux上玩java的话，常见的jdk有Open-JDK（默认)和Oracle的官方JDK两种选择，debian的官方源中已经没有Oracle(Sun)的官方deb包，所以不能简单的使用apt-get安装了。
现在如果还想使用Oracle官方JDK的话，可以按照以下几个简单的步骤安装：


# 安装依赖
    sudo apt-get install java-package

# 下载Oracle jdk压缩包
    wget -c http://www.reucon.com/cdn/java/jdk-8u5-linux-x64.tar.gz

# 制作deb包
    make-jpkg jdk-8u5-linux-x64.tar.gz

# 安装
    sudo dpkg -i oracle-java8-jdk_8u5_amd64.deb

# 配置
    sudo update-alternatives --display java # 查看系统中有哪些java版本
    sudo update-alternatives --config java # 根据提示选择就ok了

# 测试当前jdk 版本
    java -version

# 卸载
    sudo dpkg --purge oracle-java8-jdk
