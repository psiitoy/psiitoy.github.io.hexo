---
layout: post
title: "[源码]Hadoop HDFS 数据读写流程分析"
date: 2018-02-01 11:00:00 
categories: 
    - 源码
tags:
    - hadoop
---

Hadoop HDFS 数据读写流程分析

<!--more-->

---------------

## HDFS的数据模型
在对读写流程进行分析之前，我们需要先对 HDFS 的数据模型有一个简单的了解。
![图4-1](https://psiitoy.github.io/img/blog/hadoop/hadoop-4-1.png)
如上图所示，在 NameNode 中有一个唯一的 FSDirectory 类负责维护文件系统的节点关系。文件系统中的每个路径会被抽象为一个 INode 对象。在 FSDirectory 中有一个叫做 rootDir 的 INodeDirectory 类，继承自 INode 类，它代表着整个文件系统的根节点。

常用的 INode 节点有 INodeDirectory, INodeFile, INodeReference 三种。

* INodeDirectory 类代表着对目录对象的抽象，在类中有一个 List<INode> 对象 children 负责保存当前节点的子节点信息。

* INodeFile 类代表着对文件对象的抽象，对于一个大文件， HDFS 可能将其拆分为多个小文件进行存储，在这里的 blocks 对象是一个数据对象，代表着小文件的具体存放位置信息。

* INodeReference 类可以理解成 Unix 系统中的硬链接。当文件系统中可能出现多个 path 地址对应同一个 INode 节点时，会构造出 INodeReference 对象。
例如我们对 /abc/foo 构造一个快照 s0, 则 然后将 /abc/foo mv 到另一个路径 /xyz/bar，此时 /xyz/bar 和 /abc/.snapshot/s0/foo 虽然是不同的路径，但是对应着同一个 block 地址。

## HDFS的IO操作

未完待续...












