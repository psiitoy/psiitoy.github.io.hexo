---
layout: post
title: "jvm总结"
date: 2016-07-25 21:15:06 
description: "jvm总结"
categories: 
    - 虚拟机
tags:
    - jvm
---

jvm总结

<!--more-->

![图 jvmjiegou](/img/blog/jvm/jvmjiegou.png)
JVM的逻辑内存模型

1）程序计数器

几乎不占内存，用于取下一条指令

2）堆，所有通过new创建的对象的内存都在堆中分配，堆被划分为新生代和老年代。新生代有进一步划分为

Eden和Survivor区，最后Survivor由FromSpace和ToSpace组成。新建的对象都使用新生代分配内存，

Eden空间不足，会把存活对象移植到Survivor中。

3）栈，每个线程执行每个方法的时候都会在栈中申请一个栈帧，每个栈帧包括局部变量区和操作数栈，用于存放此次方法调用

过程中的临时变量、参数和中间结果

4）本地方法栈

用于支持native方法的执行，存储每个native方法调用的状态

5）方法区

存放要加载的类信息、静态变量、final类型常量、属性和方法信息，jvm用永久代来存放方法区
