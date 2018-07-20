---
layout: single
title: "深入jvm-1"
categories: jvm java bytecode
tags: jvm java bytecode

---

初探字节码与jvm
--------------

```java
public class Foo {
    public static void main(String[] args) {
        boolean flag = true;

        if (flag) {
            System.out.println("Hello, Java!");
        }

        if (true == flag) {
            System.out.println("Hello, JVM!");
        }
    }
}
```

上面这段java代码，执行结果会是什么样的呢？问题会不会太弱鸡了？

编译
```bash
javac Foo.java # 先编译下它
```

执行
```
java -cp . Foo # 执行下试试咯
Hello, Java!
Hello, JVM!
```

它的字节码是什么样子的呢？

先来准备个小工具[asmtools](https://wiki.openjdk.java.net/display/CodeTools/asmtools)分析下

```bash
# cd ~/workspace/jvm/code-tools/
hg clone http://hg.openjdk.java.net/code-tools/asmtools && cd asmtools/build && ant
# 编译结果 ~/workspace/jvm/code-tools/asmtools-7.0-build/release/
```

字节码反编译为java汇编码(jasm码)
```bash
java -cp ~/workspace/jvm/code-tools/asmtools-7.0-build/release/lib/asmtools.jar org.openjdk.asmtools.jdis.Main Foo.class > Foo.jasm.orig
rm Foo.class # 不再需要了，删掉它
```

```java
# cat Foo.jasm.orig
super public class Foo
        version 52:0
{


public Method "<init>":"()V"
        stack 1 locals 1
{
                aload_0;
                invokespecial   Method java/lang/Object."<init>":"()V";
                return;
}

public static Method main:"([Ljava/lang/String;)V"
        stack 2 locals 2
{
                iconst_1;
                istore_1;
                iload_1;
                ifeq    L14;
                getstatic       Field java/lang/System.out:"Ljava/io/PrintStream;";
                ldc     String "Hello, Java!";
                invokevirtual   Method java/io/PrintStream.println:"(Ljava/lang/String;)V";
        L14:    stack_frame_type append; 
                locals_map int;
                iconst_1;
                iload_1;
                if_icmpne       L27;
                getstatic       Field java/lang/System.out:"Ljava/io/PrintStream;";
                ldc     String "Hello, JVM!";
                invokevirtual   Method java/io/PrintStream.println:"(Ljava/lang/String;)V";
        L27:    stack_frame_type same;
                return;
}

} // end Class Foo
```

修改下字节码反编译的结果java汇编码(jasm码)
```bash
awk 'NR==1, /iconst_1/{sub(/iconst_1/, "iconst_2")} 1' Foo.jasm.orig > Foo.jasm # 把第一个iconst_1改为iconst_2

# 文件编辑器也可以修改哈
```

把修改后的java汇编码(jasm码)编译成字节码
```
java -cp ~/workspace/jvm/code-tools/asmtools-7.0-build/release/lib/asmtools.jar org.openjdk.asmtools.jasm.Main Foo.jasm
```

执行从jasm编译出来的Foo.class试试看哦

此处暂停一下，想想看Foo.class能执行么？执行会输出什么呢？为什么呢？
```
java -cp . Foo
# Hello, Java!
```

为什么没输出"Hello, JVM!"呢？

1. java/jvm把Boolean当作int来处理，flag变量被初始化为true，也就是1
1. awk命令把flag初始值`iconst_1`改成了`iconst_2`
1. `if(flag) `判断使用0值判断指令`ifeq`，flag等于常数2为非0，所以输出了"Hello, Java!"
1. `if(true == flag)`判断使用整数相等判断指令`if_icmpne`，flag = 2 != `iconst_1`(true)，所以没有输出"Hello, JVM!"

修改后的jasm码
```java

super public class Foo
	version 52:0
{


public Method "<init>":"()V"
	stack 1 locals 1
{
		aload_0;
		invokespecial	Method java/lang/Object."<init>":"()V";
		return;
}

public static Method main:"([Ljava/lang/String;)V"
	stack 2 locals 2
{
		iconst_2;  # 把常数2推至栈顶
		istore_1; # 将栈顶int型整数存入第二个本地变量； 也就是2
		iload_1; # 将第2个int型本地变量推送至栈顶；也就是2
		ifeq	L14; # 当栈顶int型数值等于0时跳转至L14；2 当然不会等于0了，所以不跳转
		getstatic	Field java/lang/System.out:"Ljava/io/PrintStream;";
		ldc	String "Hello, Java!";
		invokevirtual	Method java/io/PrintStream.println:"(Ljava/lang/String;)V";
	L14:	stack_frame_type append;
		locals_map int;
		iconst_1;  # 把常数1推至栈顶
		iload_1; # 将第2个int型本地变量推送至栈顶；也就是把2推送至栈顶
		if_icmpne	L27;  比较栈顶两个int型数值的大小，当结果不等时跳转至L27； 2 != 1 所以跳到了L27
		getstatic	Field java/lang/System.out:"Ljava/io/PrintStream;";
		ldc	String "Hello, JVM!";
		invokevirtual	Method java/io/PrintStream.println:"(Ljava/lang/String;)V";
	L27:	stack_frame_type same;
		return;
}

} // end Class Foo

```

# Ref
1. https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.3.4
1. https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11
1. https://wiki.openjdk.java.net/display/CodeTools/asmtools
1. https://wiki.openjdk.java.net/display/CodeTools/Appendix+A
