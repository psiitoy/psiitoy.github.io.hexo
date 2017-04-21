---
layout: post
title: "java锁总结"
date: 2016-07-25 21:15:06 
description: "java锁总结"
categories: 
    - java锁总结
tags:
    - lock
---

java锁总结

<!--more-->

Ps：说白了ReentrantLock就是基于Sync的，而Sync就是一种AQS，其中核心机制AQS都实现好了。

3.    Sync及AQS的核心实现（源码级别）

AQS核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源的设置为锁定状态。
如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，
即将暂时获取不到锁的线程加入到队列中。