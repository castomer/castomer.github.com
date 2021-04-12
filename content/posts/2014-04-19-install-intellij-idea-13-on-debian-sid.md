---
categories:
  - java
  - tools
tags:
  - idea
  - editor
title: debian linux安装Intellij IDEA 13
url: /2014/04/19/install-intellij-idea-13-on-debian-sid/
date: 2014-04-19
author: "Dongfang Qu"
---


没有用IDE写过代码，一直都是一个铁杆的vim粉丝，可能做的工程还没不够大吧。最近开始搞java，据说最好的java集成开发环境就是这个IDEA了，于是想装来尝试一下集成开发是个什么样的感觉。

Intellij IDEA官方没有提供debian的二进制安装包，于是就问了一下谷哥。

# 安装IDEA

    git clone https://github.com/trygvis/intellij-idea-dpkg.git
    cd intellij-idea-dpkg
    ./build-package -f IC -p debian -u
    sudo dpkg -i repository/debian/intellij-idea-iu-13.1.deb

# 安装插件

1. File->Setting->Plugin->Install JetBrains Plugin... 
1. vim
1. clojure

# 安装字体
据说最好的代码字体[SourceCodePro](https://github.com/adobe/source-code-pro)，如果你不喜欢它，可去翻翻[what are the best programming fonts](http://www.slant.co/topics/67/~what-are-the-best-programming-fonts)。


初步感觉：功能繁多，眼花缭乱，慢慢学习ing~
