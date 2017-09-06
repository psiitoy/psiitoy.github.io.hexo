---
layout: post
title: "[理论]Java相关名词"
date: 2016-07-25 21:15:06 
categories: 
    - 理论
tags:
    - JAVA
---

Java相关名词

<!--more-->


 JVM：Java Virtual Machine
 负责执行符合规范的Class文件
 JRE：Java Runtime Environment
 包含JVM与类库
 JDK：Java Development Kit
 包含JRE和一些开发工具，如javac


JIT即时编译
Java 刚诞生的时候是一个解释性语言，即使编译成了字节码（byte code）也是针对JVM 的，它需要再次翻译成原生代码(native code)才能被机器执行，于是效率的担忧就提出来了。Sun 为了解决该问题在JVM 上提供一个工具，把字节码编译成原生码，等方法再次被访问的时候直接访问原生码就成了，于是JIT 就诞生了。
Java方法每次被调用时，有一个方法调用计数器记录它被调用的次数，当计数器达到阈值（-XX:CompileThreshold，client: 1500、server: 10000），会异步提交JIT编译器线程执行编译任务，当然你也可以禁用后台编译（-XX:-BackgroundCompilation戒-Xbatch），同步提交编译任务。
方法调用计数器会随着时间推移做指数衰减，这样，方法调用计数器实际记录的不是准确的累计调用次数，而是方法的“热度”。
