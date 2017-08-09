---
layout: post
title: "[源码]Elasticsearch源码(1通信机制之RPC)"
date: 2017-08-09 20:15:06 
categories: 
    - 源码
tags:
    - es
---

[源码]Elasticsearch源码(1通信机制之RPC)


<!--more-->

既然是分布式系统当然离不开通信层，首先es通信部分使用的Netty，Netty又支持了Tcp和Http两种方式做通信。
集群Node之间的通信，数据的传输，java客户端的请求调用(transport client)使用的均是Tcp，此外同样支持
Rest等Http方式的请求。

首先上图，举最简单的例子引入话题，一次Get请求调用。

场景1：Client 连接节点A并发送请求，数据在节点B。

    
场景2：简化场景1,Client 连接节点A并发送请求，数据就在A直接返回。

然后我们详细分析场景2,发现其主要做的工作就是实现了异步传输数据的方式。


如果说AQS是编程式的解决了get和异步set then notify get的问题，那么requestID解决的就是异步通信的的问题。
比如A,B,C请求对应A,B,C的响应，响应
http://www.cnblogs.com/birkhoff/p/5752464.html
