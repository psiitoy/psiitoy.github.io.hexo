---
layout: post
title: "[源码]Hadoop HDFS 启动流程分析"
date: 2018-01-30 16:30:00 
categories: 
    - 源码
tags:
    - hadoop
---

Hadoop HDFS 启动流程分析

<!--more-->

---------------

## HDFS的基础架构
![图2-1](https://psiitoy.github.io/img/blog/hadoop/hadoop-2-1.png)

- 如上图所示。 默认情况下，HDFS 由一个 Namenode 和多个 DataNode 组成。
- HDFS作为一个分布式文件存储系统，他的文件路径和文件内容是相互隔离的。 文件路径信息保存在 NameNode 中，文件内容则分布式的保存在 DataNode中。
- 也就是说对于一个大文件，它可能被根据其文件大小切割成多个小文件进行存储，同时这些小文件可能被分布式的存储在不同的DataNode中。
- 当Client希望获取这个文件时，则需要先根据文件路径从 NameNode 中获取他对应的区块在不同的 DataNode 中的信息，然后才能够从 DataNode 中取出数据，还原成一个大文件。
- 具体的代码逻辑会在之后的读写操作中介绍，这里我们先看看 HDFS 集群的启动流程。

## HDFS集群启动流程
>默认情况下，我们通过 ${HADOOP_HOME}/sbin/start-dfs.sh 启动整个 HDFS 集群。
```
hdfs namenode "${HADOOP_HDFS_HOME}/bin/hdfs" --workers --config "${HADOOP_CONF_DIR}" --hostnames "${NAMENODES}" --daemon start namenode ${nameStartOpt}

hdfs datanode "${HADOOP_HDFS_HOME}/bin/hdfs" --workers --config "${HADOOP_CONF_DIR}" --daemon start datanode ${dataStartOpt}

hdfs secondarynamenode "${HADOOP_HDFS_HOME}/bin/hdfs" --workers --config "${HADOOP_CONF_DIR}" --hostnames "${SECONDARY_NAMENODES}" --daemon start secondarynamenode

hdfs journalnode "${HADOOP_HDFS_HOME}/bin/hdfs" --workers --config "${HADOOP_CONF_DIR}" --hostnames "${JOURNAL_NODES}" --daemon start journalnode

hdfs zkfc "${HADOOP_HDFS_HOME}/bin/hdfs" --workers --config "${HADOOP_CONF_DIR}" --hostnames "${NAMENODES}" --daemon start zkfc
```
>在start-dfs.sh文件中，我们看到shell文件通过执行hdfs命令先后启动 NameNode 、 DataNode 、 SecondaryNameNode 、 JournalNode。
- hdfs也是一个shell文件，通过文本应用打开后，我们看到在该文件的执行逻辑如下:
![图2-2](https://psiitoy.github.io/img/blog/hadoop/hadoop-2-2.png)

## HDFS节点类型和Java类的对应
>在hdfs文件中可以看到，传入的第一个参数对应着需要启动节点的类型，使用 hdfs cmd_case 根据节点类型可以找到对应需要执行的Java类，节点对应表如下。

namenode - org.apache.hadoop.hdfs.server.namenode.NameNode
datanode - org.apache.hadoop.hdfs.server.datanode.DataNode
secondarynamenode - org.apache.hadoop.hdfs.server.namenode.SecondaryNameNode
journalnode - org.apache.hadoop.hdfs.qjournal.server.JournalNode

## NameNode启动逻辑
NameNode在整个HDFS中扮演着一个很重要的角色,他负责整个文件系统的路径和数据的管理工作，比较类似 Unix 系统中的 inode table。
Client 通过 NameNode 获取到指定路径的分布式文件的 metadata,找到存放在 DataNode 的数据位置，就像从 inode talbe中获取 inode 编号，然后才能去访问具体的分布式文件。
从上一小节的表格中，我们看到 NameNode 对应的启动类是 org.apache.hadoop.hdfs.server.namenode.NameNode，接下来我们通过走读NameNode::main 具体查看节点的启动流程。
由于 NameNode 的启动参数很多，类似 FORMAT,CLUSTERID,IMPORT等等启动参数并不会被用来启动 NameNode。为了简单起见，这里只针对默认启动逻辑 REGULAR进行分析，启动流程如下:
![图2-3](https://psiitoy.github.io/img/blog/hadoop/hadoop-2-3.png)

>NameNode结构如图所示，对于REGULAR模式而言，NameNode节点主要启动了下面三个组件:
![图2-4](https://psiitoy.github.io/img/blog/hadoop/hadoop-2-4.png)

### NameNodeHttpServer
startHttpServer() 会启动一个 NameNodeHttpServer。
NameNodeHttpServer 是一个基于 jetty 服务器的简单封装，提供给使用人员一个简单的节点状态查询页面，默认的绑定端口是 :9870，静态页面代码路径是 webapps/namenode，这一部分由于不涉及到核心逻辑，不做介绍。

### FSNamesystem
loadNamesystem() 会构建一个 FSNamesystem 对象，FSNamesystem 是 NameNode 中最核心的一个模块，负责处理维护整个分布式文件系统。详细的处理逻辑会在后续的章节做仔细介绍。

### RPC.Server
createRPCServer() 会启动一个 RPC.Server 线程。
RPC.Server 负责处理 HDFS 集群中的内部通信，在 RPC.Server 中绑定了一个 socket 端口，利用protobuf的序列化框架进行数据传输，关于 RPC.Server 的交互逻辑会在下一章中做详细介绍，这里可以简单理解为一个使用socket通信的rpc访问框架
```
if (serviceRpcAddr != null) {
    serviceRpcServer = new RPC.Builder(conf).build();
}
if (lifelineRpcAddr != null) {
    lifelineRpcServer = new RPC.Builder(conf).build();
}
clientRpcServer = new RPC.Builder(conf).build()
```
在NameNodeRpcServer的构造方法中可能会构造 serviceRpcServer , lifelineRpcServer 和 clientRpcServer 三种 RPC.Server。

## DataNode启动逻辑
DataNode 在整个HDFS框架中负责具体的文件存储。
我们知道DataNode的启动类是org.apache.hadoop.hdfs.server.datanode.DataNode
![图2-5](https://psiitoy.github.io/img/blog/hadoop/hadoop-2-5.png)
他的启动逻辑如上图所示。和NameNode类似，在DataNode中同样启动了一个基于 jetty Server的 HTTP 服务器，负责向使用人员展示节点状态；同样还有一个基于 RPC.Server 的 socket 链接负责 hdfs集群 的内部通信。
![图2-6](https://psiitoy.github.io/img/blog/hadoop/hadoop-2-6.png)

### DataXceiverServer
initDataXceiver() 会启动一个 DataXceiverServer 类。
DataXceiverServer 负责接收通过TCP协议传输过来的文件数据，会在下一章介绍 hdfs 的文件传输的时候做介绍，这里先略过

### DatanodeHttpServer
DatanodeHttpServer 和 NameNodeHttpServer 一样，也是一个基于 jetty 服务器的封装，负责向使用人员提供当前Datanode的节点状态信息，默认的启动端口是 :9864,静态页面代码路径 webapps/datanode,同样这一部分不设计核心逻辑，不做介绍

### RPC.Server
DataNode中也存在一个RPC.Server对象，负责维持节点间通信，在下一篇文章中会做详细介绍。

### BlockPoolManager
由于在HDFS中文件路径和文件内容是相互隔离的，在DataNode负责存放分布式的文件内容，但是对于DataNode自身并不知道自己的文件名，只有一个唯一的 blockId 用以定位文件信息。
在 BlockPoolManager 中，DataNode会定期上报当前节点的 blockId 列表，以便告知NameNode节点中拥有的文件内容。具体的逻辑会在 hdfs的文件传输的时候做介绍。

## 总结
介绍了start-dfs.sh 的启动逻辑和 NameNode 与 DataNode 的关键组件构成。

