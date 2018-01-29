前言
说到分布式软件，就一定绕不过Hadoop。

Hadoop 是 Google 著名的 MapReduce 和 GFS 论文的开源实现，它为我们提供了一个分布式的数据存储和计算框架，能够让我们在低成本的PC设备上搭建一个大规模的分布式数据存储系统。

由于Hadoop的出现直接降低了大数据的存储和计算成本，可以说Hadoop以及他的整个生态环境拉开了大数据时代的大幕。

Hadoop主要由 Hdfs, MapReduce 和 Yarn 三个大模块组成，我会基于 Hadoop 3.0.0 alpha2 的源码，分别解析一下这三个模块的代码逻辑。

