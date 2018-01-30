---
layout: post
title: "[源码]Hadoop HDFS 启动流程分析"
date: 2018-01-30 16:30:00 
categories: 
    - 源码
tags:
    - hadoop
---

<!--more-->
## HDFS的基础架构
![图2-1](https://psiitoy.github.io/img/blog/hadoop/hadoop-2-1.png)

- 如上图所示。 默认情况下，HDFS 由一个 Namenode 和多个 DataNode 组成。
- HDFS作为一个分布式文件存储系统，他的文件路径和文件内容是相互隔离的。 文件路径信息保存在 NameNode 中，文件内容则分布式的保存在 DataNode中。
- 也就是说对于一个大文件，它可能被根据其文件大小切割成多个小文件进行存储，同时这些小文件可能被分布式的存储在不同的DataNode中。
- 当Client希望获取这个文件时，则需要先根据文件路径从 NameNode 中获取他对应的区块在不同的 DataNode 中的信息，然后才能够从 DataNode 中取出数据，还原成一个大文件。
- 具体的代码逻辑会在之后的读写操作中介绍，这里我们先看看 HDFS 集群的启动流程。

## HDFS集群启动流程
>默认情况下，我们通过 ${HADOOP_HOME}/sbin/start-dfs.sh 启动整个 HDFS 集群。

- hdfs namenode "${HADOOP_HDFS_HOME}/bin/hdfs" --workers --config "${HADOOP_CONF_DIR}" --hostnames "${NAMENODES}" --daemon start namenode ${nameStartOpt}

- hdfs datanode "${HADOOP_HDFS_HOME}/bin/hdfs" --workers --config "${HADOOP_CONF_DIR}" --daemon start datanode ${dataStartOpt}

- hdfs secondarynamenode "${HADOOP_HDFS_HOME}/bin/hdfs" --workers --config "${HADOOP_CONF_DIR}" --hostnames "${SECONDARY_NAMENODES}" --daemon start secondarynamenode

- hdfs journalnode "${HADOOP_HDFS_HOME}/bin/hdfs" --workers --config "${HADOOP_CONF_DIR}" --hostnames "${JOURNAL_NODES}" --daemon start journalnode

- hdfs zkfc "${HADOOP_HDFS_HOME}/bin/hdfs" --workers --config "${HADOOP_CONF_DIR}" --hostnames "${NAMENODES}" --daemon start zkfc

>在start-dfs.sh文件中，我们看到shell文件通过执行hdfs命令先后启动 NameNode 、 DataNode 、 SecondaryNameNode 、 JournalNode。
- hdfs也是一个shell文件，通过文本应用打开后，我们看到在该文件的执行逻辑如下:
![图2-2](https://psiitoy.github.io/img/blog/hadoop/hadoop-2-2.png)

## HDFS节点类型和Java类的对应
>在hdfs文件中可以看到，传入的第一个参数对应着需要启动节点的类型，使用 hdfs cmd_case 根据节点类型可以找到对应需要执行的Java类，节点对应表如下。

- namenode org.apache.hadoop.hdfs.server.namenode.NameNode
- datanode org.apache.hadoop.hdfs.server.datanode.DataNode
- secondarynamenode org.apache.hadoop.hdfs.server.namenode.SecondaryNameNode
- journalnode org.apache.hadoop.hdfs.qjournal.server.JournalNode