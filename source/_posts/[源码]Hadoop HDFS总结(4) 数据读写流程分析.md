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
当通过客户端 hdfs dfs 命令进行文件 IO 操作时，会根据配置文件中 fs.defaultFS 的配置信息构造出一个 FileSystem 对象。具体的文件操作指令，通过 FileSystem 中对应的接口进行访问。

对于客户端HDFS操作，他的默认 FileSystem 实现类是 DistributedFileSystem, 在 DistribtedFileSystem 中有一个 DFSClient 对象。这个对象使用前一篇文章中介绍的内部 RPC 通信机制，构造了一个 NameNode 的代理对象，负责同 NameNode 间进行 RPC 操作。

## HDFS的文件写入流程
![图4-2](https://psiitoy.github.io/img/blog/hadoop/hadoop-4-2.png)

以 hadoop fs -put 操作为例:

* 当接收到 PUT 请求时，尝试在 NameNode 中 create 一个新的 INode 节点，这个节点是根据 create 中发送过去的 src 路径构建出的目标节点,如果发现节点已存在或是节点的 parent 存在且不为 INodeDirectory 则异常中断，否则则返回包含 INode 信息的 HdfsFileStatus 对象。

* 使用 HdfsFileStatus 构造一个实现了 OutputStream 接口的 DFSOutputStream 类，通过 nio 接口将需要传输的数据写入 DFSOutputStream。

* 在 DFSOutputStream 中写入的数据被以一定的 size（一般是 64 k）封装成一个 DFSPacket,压入 DataStreamer 的传输队列中。

* DataStreamer 是 Client 中负责数据传输的独立线程，当发现队列中有 DFSPacket 时，先通过 namenode.addBlock 从 NameNode 中获取可供传输的 DataNode 信息，然后同指定的 DataNode 进行数据传输。

* DataNode 中有一个专门的 DataXceiverServer 负责接收数据，当有数据到来时，就进行对应的 writeBlock 写入操作，同时如果发现还有下游的 DataNode 同样需要接收数据，就通过管道再次将发来的数据转发给下游 DataNode，实现数据的备份，避免通过 Client 一次进行数据发送。

整个操作步骤中的关键步骤有 NameNode::addBlock 以及 DataNode::writeBlock, 接下来会对这两步进行详细分析。

## NameNode::addBlock解析

在上面的数据模型中我们看到，对于一个 INodeFile 节点，我们可能会根据其数据大小将其拆分成多个 Block，因此当传输新文件或者文件传输尺寸已经超过 blockSize 的时候，就需要通过 addBlock 获取新的传输地址。

NameNode 中 addBlock 的实现路径在 FSNamesystem::getAdditionalBlock 中，这里先通过 FSDirWriteFileOp::validateAddBlock 判断是否是因为延迟或异常问题导致的无效请求，如果不是，则通过 FSDirWriteFileOp.chooseTargetForNewBlock 选取新 Block 的目标 DataNode，

chooseTargetForNewBlock 的具体算法由 BlockPlacementPolicy 完成，默认情况下会优先选择 client 自身所在机器作为 target，如果自身机器不是 DataNode，则会优先选择和当前机器处于同一机架( rack )中的 DataNode，以提升数据传输效率。

官方放置策略解释：
* The 1st replica is placed on the local machine, otherwise a random datanode. 
* The 2nd replica is placed on a datanode that is on a different rack.
* The 3rd replica is placed on a datanode which is on a different node of the rack as the second replica.

确定写入的 DataNode 后，通过 FSDirWriteFileOp::storeAllocatedBlock 构造 Block 对象，并放入 src 对应的 INodeFile 中。

## DataNode::writeBlock 解析

DataNode 中的 DataXceiverServer 负责接收从 Client 发送来的数据传输请求。当有新的链接接通时，会构造一个 DataXceiver 线程进行数据接收。

在 DataXceiver::writeBlock 中，如果发现 targets.length > 0，则说明还有下游的 DataNode 需要接收数据传输，这时候会和 Client 一样构造出一个链接到下游 DataNode 的 socket 链接，通过 new Sender(mirrorOut).writeBlock 将数据写入下游。


## HDFS的文件读取流程
![图4-3](https://psiitoy.github.io/img/blog/hadoop/hadoop-4-3.png)

hadoop fs -get操作：

GET 操作的流程，相对于 PUT 会比较简单，先通过参数中的来源路径从 NameNode 对应 INode 中获取对应的 Block 位置，然后基于返回的 LocatedBlocks 构造出一个 DFSInputStream 对象。在 DFSInputStream 的 read 方法中，根据 LocatedBlocks 找到拥有 Block 的 DataNode 地址，通过 readBlock 从 DataNode 获取字节流。


hadoop fs -mv 操作：

MV 操作只涉及对文件名称或路径的更改，因此他的主要步骤集中在 NameNode 端，Client 端只是通过 RPC 调用 NameNode::rename
![图4-4](https://psiitoy.github.io/img/blog/hadoop/hadoop-4-4.png)

从活动图中我们看到，整个 rename 的操作分了两步，第一步是 removeSrc4OldRename，将 src 从 FSDirectory 中移除，第二步是 addSourceToDestination ，将之前移除的 src 的 INode，重新根据 dst 的路径添加到 FSDirectory 中，完成整个重命名流程。


## 总结

HDFS 中的文件 IO 操作主要是发生在 Client 和 DataNode 中。

NameNode 作为整个文件系统的 Namesystem 负责管理整个文件系统的路径树，当需要新建文件或读取文件时，会从文件树中读取对应的路径节点的 Block 信息，发送回 Client 端。 Client 通过从返回数据中得到的 DataNode 和 Block 信息，直接从 DataNode 中进行数据读取。

整个数据 IO 流程中，NameNode 只负责管理节点和 DataNode 的对应关系，涉及到 IO 操作的行为少，从而将整个文件传输压力从 NameNode 转移到了 DataNode 中。