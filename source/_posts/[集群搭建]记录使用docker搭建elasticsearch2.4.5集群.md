---
layout: post
title: "[集群搭建]记录使用docker搭建elasticsearch2.4.5集群"
date: 2017-06-06 20:15:06 
description: "记录使用docker搭建elasticsearch2.4.5集群"
categories: 
    - 集群搭建
tags:
    - docker
    - elasticsearch
    - 运维
---

记录使用docker搭建elasticsearch2.4.5集群

<!--more-->

## 1、环境介绍

  本文运行环境 `ubuntu16.04` + `docker17.05` + 官网上下载的`elasticsearch2.4.5`，另外docker环境为`openjdk:8-jre-alpine`。（使用alpine的原因就是没有太多不必要的组件和命令，docker内部也不需要太多组件）

> ubuntu

```bash
$ cat /proc/version
Linux version 4.4.0-46-generic (buildd@lcy01-10) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.2) ) #67-Ubuntu SMP Thu Oct 20 15:05:12 UTC 2016

```

> docker

```bash
$ docker -v
Docker version 17.05.0-ce, build 89658be

```

> elasticsearch
```bash
$ ./elasticsearch -version
Version: 2.4.5, Build: c849dd1/2017-04-24T16:18:17Z, JVM: 1.8.0_101

```

## 2、部署过程中遇到的问题
 
1) 主要就是docker的网络配置，默认是桥接网络，不知为何多个node启动来之后无法互相发现。
> 解决方案：修改`elasticsearch.yml`配置。

2) 每次重启docker实例docker自动会重新分配ip。
> 未解决。

## 3、部署过程

1) 制作生成elasticsearch(以下简称es)的`dockerfile`.
> [dockerfile的git地址](https://github.com/psiitoy/psiitoy.dockerfile.git)，如下图。

![图 es-dockerfile](/img/blog/esdocker/es-dockerfile.png)

2) 在es源文件中为es安装`head`插件。

```bash
$ plugin install mobz/elasticsearch-head

```

3) 生成es的docker镜像
> 镜像名称为`psiitoy/elasticsearch:2.4.5`。

```bash
$ docker build -t psiitoy/elasticsearch:2.4.5 .

```

4) 查看已有网桥信息。
> docker 默认使用`docker0`作为网桥

```bash
$ brctl show
bridge name	bridge id		STP enabled	interfaces
docker0		8000.0242552a7fb0	no		vetha061837

```

5) 查看网桥对应网段地址。
> 发现网段为172.17.0.x

```bash
$ ip addr show docker0
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:55:2a:7f:b0 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:55ff:fe2a:7fb0/64 scope link 
       valid_lft forever preferred_lft forever

```

6) 修改`elasticsearch.yml`配置。
> 避免docker容器es服务之间无法互相发现导致的脑裂。
> 参照[Elasticsearch部分节点不能发现集群(脑裂)问题处理](http://blog.csdn.net/huwei2003/article/details/47004745)

```bash
network.host: 0.0.0.0
discovery.zen.ping.multicast.enabled: false
discovery.zen.ping_timeout: 120s
discovery.zen.minimum_master_nodes: 2 #至少要发现集群可做master的节点数，
client.transport.ping_timeout: 60s
discovery.zen.ping.unicast.hosts: ["172.17.0.2", "172.17.0.3"]

```

6) 创建docker容器x5。
> docker run 参数介绍：
> 在使用 `-v` 参数时，代表把本地目录挂载到镜像里，我们可以在本机查看容器中的/usr/share/elasticsearch/logs目录。
> 在使用 `-d` 参数时，容器启动后会进入后台。

```bash
$ docker run -d -v ~/IdeaWorkspace/docker/psiitoy.dockerfile/dockerfile-alpine-es/elasticsearch-2.4.5/logs:/usr/share/elasticsearch/logs psiitoy/elasticsearch:2.4.5
66ca0270ef44
$ docker run -d -v ~/IdeaWorkspace/docker/psiitoy.dockerfile/dockerfile-alpine-es/elasticsearch-2.4.5/logs:/usr/share/elasticsearch/logs psiitoy/elasticsearch:2.4.5
ddfbb7ef65bc

```

7) 若想进入容器可以执行 `docker exec -it $CONTAINER_ID`
> 在使用 `-i` 参数时，打开STDIN，用于控制台交互
> 在使用 `-t` 参数时，分配tty设备，该可以支持终端登录，默认为false

```bash
$ docker exec -it 66ca0270ef44 /bin/bash
bash-4.3#

```

8) 查看集群状态
> `curl http://ip:port/_cat/health?v`，也可以用`health?pretty`
> 我们看到集群有5个节点加入了，实验成功！

```bash
$ curl http://172.17.0.2:9200/_cat/health?v
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent 
1497352537 11:15:37  elasticsearch green           5         5      0   0    0    0        0             0                  -                100.0% 

```

9) 测试一下用客户端创建索引
> 创建一个 `number_of_shards`为8,`number_of_replicas`为1 的名叫twitter的index。
> [testClient的git地址](https://github.com/sprintDragon/experiment/tree/master/experiment-elasticsearch)

![图 es-dockerfile](/img/blog/esdocker/es-test-client.png)

> 查看head集群信息
![图 es-dockerfile](/img/blog/esdocker/es-head-cluster.png)


10) 停止并删除所有docker容器
> 加`-f`参数强制停止删除

```bash
$ docker rm -f $(docker ps -aq)
b1b17d0f41eb
55d9259ff921
ff4d42575213
66ca0270ef44
ddfbb7ef65bc

```










