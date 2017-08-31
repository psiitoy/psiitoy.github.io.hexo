---
layout: post
title: "[理论]BIO-NIO-同步IO-异步IO"
date: 2016-07-25 21:15:06 
categories: 
    - 理论
tags:
    - NIO
---

BIO-NIO-同步IO-异步IO

<!--more-->

BIO 阻塞
> 一直block住对应的进程直到操作完成

NIO 轮询
> 在kernel还准备数据的情况下会立刻返回

AIO(NIO2) 拷贝到用户内存中之后主动通知
> 异步IO

同步IO
> 做”IO operation”的时候会将process阻塞，定义中所指的”IO operation”是指真实的IO操作

异步IO
> 它就像是用户进程将整个IO操作交给了他人（kernel）完成，然后他人做完后发信号通知。在此期间，用户进程不需要去检查IO操作的状态，也不需要主动的去拷贝数据。

