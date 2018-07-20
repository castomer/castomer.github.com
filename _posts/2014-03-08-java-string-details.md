---
layout: single
title: "java String 详解"
categories: java
tags: String


---

Java可表达字符串的类主要有java.lang.String, java.lang.StringBuffer和java.lang.StringBuilder三种,下面介绍一下它们三者的异同；

## [java.lang.String](http://docs.oracle.com/javase/7/docs/api/java/lang/String.html)

字符串常量，只读，每次赋值操作会destroy掉原有对象重新创建，修改代价大，适用于表达只读字符串变量，线程安全。

## [java.lang.StringBuilder](http://docs.oracle.com/javase/7/docs/api/java/lang/StringBuilder.html)

字符串变量，可修改，类似于C/C++的字符串，支持字符串编辑，delete/insert等方法，适用于字符串编辑操作，非线程安全。

## [java.lang.StringBuffer](http://docs.oracle.com/javase/7/docs/api/java/lang/StringBuffer.html)

java.lang.StringBuilder的线程安全版本，适用于多线程环境下对字符串的编辑修改。

## String 传参
String是引用类型，在函数调用中是按引用传递参数，但是String是final类型的，定长，不允许对其进行改变，函数调用里面对String类型变量的赋值操作事实上是创建了一个新的String变量；所以函数调用中对String类型变量的修改不会影响函数体外变量的值。

```java
// StringUsage.java

package String;

import java.lang.String;
import java.lang.StringBuffer;
import java.lang.StringBuilder;

public class StringUsage {
    public static void main(String[] args) {

        String str1 = "string"; // 在栈上分配，静态变量区,同一个字符串只有一个副本
        String str2 = "string";
        String str3 = new String("string"); // 在堆上分配，单独创建一个对象
        String str4 = new String("string");

        System.out.println(str1 == str2); // true, str1和str2指向相同的内存地址
        System.out.println(str2 == str3); // false, str2和str3指向的内存地址不同
        System.out.println(str3 == str4); // false, str3和str4指向不同的内存对象
        str3 = str3.intern(); // String.intern() 在常量池创一个值和str3相同的字符串返回，如果常量池已存则直接返回存在的地址
        System.out.println(str3 == str1); // true

        // 理论上修改操作的性能应该是:String << StringBuffer < StringBuilder
        long start = 0, end = 0;
        start = System.currentTimeMillis();
        for(int i = 0; i < 100000; i++)
        {
            str1+= "test";
        }
        end = System.currentTimeMillis();
        System.out.println("consumed: " + (end -start) + " ms");
        // consumed: 8925 ms

        StringBuffer stringBuffer = new StringBuffer("string buffer");
        start = System.currentTimeMillis();
        for (int i = 0; i < 100000; i++) {
            stringBuffer.append("test");
        }
        end = System.currentTimeMillis();
        System.out.println("consumed: " + (end - start) + " ms");
        // consumed: 10 ms

        StringBuilder stringBuilder = new StringBuilder("string builder");
        start = System.currentTimeMillis();
        for (int i = 0; i < 100000; i++) {
            stringBuilder.append("test");
        }
        end = System.currentTimeMillis();
        System.out.println("consumed: " + (end - start) + " ms");
        // consumed: 5 ms

    }
}
```
