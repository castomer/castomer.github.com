---
layout: single
title: "源码编译openjdk 8"
categories: java jdk
tags: java jdk


---

最近在看[<<深入理解Java虚拟机-JVM高级特性与最佳实践>>](http://book.douban.com/subject/24722612/)这本书，里面有讲解openjdk 7的源码编译方法，比较喜欢尝鲜的我决定编译一下openjdk 8。
阅读了openjdk 8的[Build README](http://hg.openjdk.java.net/jdk8/build/raw-file/tip/README-builds.html)后发现，编译方式相比openjdk 7来说简单了许多，与openjdk 1.7编译的区别：不再依赖ant和ATL_*环境变量；废话不多说，基本步骤如下：

1. 安装工具和依赖

```bash
    sudo apt-get install mercurial build-essential openjdk-7 #alsa freetype cups xrender 
    # 有些依赖即使没装，configure时根据提示逐个安装即可
```

2. 下载源码

```bash
     hg clone http://hg.openjdk.java.net/jdk8/jdk8
     cd jdk8 
     bash ./get_source.sh
```

3. gnu老式编译方法

```bash
    export LANG=C
    ./configure #  --help=short
    make # all / all-conf / images / install / clean / dist-clean 
    make image # 创建j2sdk和j2re
    # 编译的结果在build/linux-x86_64-normal-server-release/images/
```

4. 测试(optional), 依赖[jtreg](http://openjdk.java.net/jtreg/)

```bash
    cd test && make PRODUCT_HOME=`pwd`/../build/*/images/j2sdk-image all
```

