---
layout: post
title: "[集群搭建]记录使用docker搭建elasticsearch集群"
date: 2017-06-06 20:15:06 
categories: 
    - 集群搭建
tags:
    - docker
    - elasticsearch
    - 集群
---

本文记录使用docker搭建elasticsearch集群的整个过程(文中使用的2.1.2举例)，过程亲测同样适用于elasticsearch2.x,5.x，后续作者将继续深入研究es，下一步准备基于此集群对源生elasticsearch(以下简称es)做改造测试。

<!--more-->

---------------

## 1、 环境介绍

> 本文运行环境 `ubuntu16.04` + `docker17.05` + 官网上下载的`elasticsearch2.1.2`，另外docker环境为`openjdk:8-jre-alpine`。（使用alpine的原因就是没有太多不必要的组件和命令，docker内部也不需要太多组件）
> ***[es源码下载地址](https://www.elastic.co/downloads/past-releases)***

### 1.1 ubuntu

```bash
$ cat /proc/version
Linux version 4.4.0-46-generic (buildd@lcy01-10) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.2) ) #67-Ubuntu SMP Thu Oct 20 15:05:12 UTC 2016

```

### 1.2 docker

```bash
$ docker -v
Docker version 17.05.0-ce, build 89658be

```

### 1.3 elasticsearch(版本忽略)

```bash
$ ./elasticsearch -version
Version: 2.1.2, Build: c849dd1/2017-04-24T16:18:17Z, JVM: 1.8.0_101

```

---------------

## 2、 部署过程中遇到的问题
 
### 2.1 主要就是docker的网络配置，默认是桥接网络，不知为何多个node启动来之后无法互相发现。
> 解决方案：修改`elasticsearch.yml`配置。

### 2.2 每次重启docker实例docker自动会重新分配ip。
> 未解决。

---------------

## 3、 部署过程

### 3.1 制作生成es的`dockerfile`.
> 参照***[dockerfile的git地址](https://github.com/psiitoy/psiitoy.dockerfile.git)***，如下图。

![图 es-dockerfile](https://psiitoy.github.io/img/blog/esdocker/es-dockerfile.png)

### 3.2 在es源文件中为es安装`head`插件。

```bash
$ plugin install mobz/elasticsearch-head

```

### 3.3 生成es的docker镜像
> 镜像名称为`psiitoy/elasticsearch:2.1.2`。

```bash
$ docker build -t psiitoy/elasticsearch:2.1.2 .

```

### 3.4 查看已有网桥信息。
> docker 默认使用`docker0`作为网桥

```bash
$ brctl show
bridge name	bridge id		STP enabled	interfaces
docker0		8000.0242552a7fb0	no		vetha061837

```

### 3.5 查看网桥对应网段地址。
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

### 3.6 修改`elasticsearch.yml`配置。
> 避免docker容器es服务之间无法互相发现导致的脑裂。
> 参照***[Elasticsearch部分节点不能发现集群(脑裂)问题处理](http://blog.csdn.net/huwei2003/article/details/47004745)***

```bash
network.host: 0.0.0.0
discovery.zen.ping.multicast.enabled: false
discovery.zen.ping_timeout: 120s
discovery.zen.minimum_master_nodes: 2 #至少要发现集群可做master的节点数，
client.transport.ping_timeout: 60s
discovery.zen.ping.unicast.hosts: ["172.17.0.2", "172.17.0.3"]

```

### 3.7 创建docker容器x5。
> docker run 参数介绍：
> 在使用 `-v` 参数时，代表把本地目录挂载到镜像里，我们可以在本机查看容器中的/usr/share/elasticsearch/logs目录。
> 在使用 `-d` 参数时，容器启动后会进入后台。

```bash
$ docker run -d -v ~/usr/share/elasticsearch/logs:/usr/share/elasticsearch/logs psiitoy/elasticsearch:2.1.2
66ca0270ef44
$ docker run -d -v ~/usr/share/elasticsearch/logs:/usr/share/elasticsearch/logs psiitoy/elasticsearch:2.1.2
ddfbb7ef65bc

```

### 3.8 若想进入容器可以执行 `docker exec -it $CONTAINER_ID`
> 在使用 `-i` 参数时，打开STDIN，用于控制台交互
> 在使用 `-t` 参数时，分配tty设备，该可以支持终端登录，默认为false

```bash
$ docker exec -it 66ca0270ef44 /bin/bash
bash-4.3#

```

### 3.9 查看集群状态
> `curl http://ip:port/_cat/health?v`，也可以用`health?pretty`
> 我们看到集群有5个节点加入了，实验成功！

```bash
$ curl http://172.17.0.2:9200/_cat/health?v
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent 
1497352537 11:15:37  elasticsearch green           5         5      0   0    0    0        0             0                  -                100.0% 

```

### 3.10 测试一下用客户端创建索引
> 创建一个 `number_of_shards`为8,`number_of_replicas`为1 的名叫twitter的index。
> 参照***[testClient的git地址](https://github.com/sprintDragon/experiment/tree/master/experiment-elasticsearch)***

```java
public class ClientTest {
    private static Client client = null;//client一定要是单例，单例，单例！不要在应用中构造多个客户端！

    public static void main(String[] args) {
        try {
            client = getClient("elasticsearch", "172.17.0.3", 9300);//client一定要是单例，单例，单例！不要在应用中构造多个客户端！
            createIndex("twitter", "tweet");
            System.out.println("#" + client);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (client != null) {
                client.close();
            }
        }
    }

    /**
     * 创建es client 一定要是单例，单例，单例！不要在应用中构造多个客户端！
     * clusterName:集群名字
     * nodeIp:集群中节点的ip地址
     * nodePort:节点的端口
     *
     * @return
     * @throws UnknownHostException
     */
    public static synchronized Client getClient(String clusterName, String nodeIp, int nodePort) throws UnknownHostException {
        //设置集群的名字
        Settings settings = Settings.settingsBuilder()
                .put("cluster.name", clusterName)
                .put("client.transport.sniff", false)
//                .put("number_of_shards", 1)
//                .put("number_of_replicas", 0)
                .build();

        //创建集群client并添加集群节点地址
        Client client = TransportClient.builder().settings(settings).build()
//                .addTransportAddress(new InetSocketTransportAddress("192.168.200.195", 9370))
//                .addTransportAddress(new InetSocketTransportAddress("192.168.200.196", 9370))
//                .addTransportAddress(new InetSocketTransportAddress("192.168.200.197", 9370))
//                .addTransportAddress(new InetSocketTransportAddress("192.168.200.198", 9370))
                .addTransportAddress(
                        new InetSocketTransportAddress(InetAddress.getByName(nodeIp),
                                nodePort));

        return client;
    }

    /**
     * 创建索引
     * 注意：在生产环节中通知es集群的owner去创建index
     * @param indexName
     * @param documentType
     * @throws IOException
     */
    private static void createIndex(String indexName, String documentType) throws IOException {
        final IndicesExistsResponse iRes = client.admin().indices().prepareExists(indexName).execute().actionGet();
        if (iRes.isExists()) {
            client.admin().indices().prepareDelete(indexName).execute().actionGet();
        }
        client.admin().indices().prepareCreate(indexName).setSettings(Settings.settingsBuilder().put("number_of_shards", 8).put("number_of_replicas", 1)).execute().actionGet();
        XContentBuilder mapping = jsonBuilder()
                .startObject()
                .startObject(documentType)
//                     .startObject("_routing").field("path","tid").field("required", "true").endObject()
                .startObject("_source").field("enabled", "true").endObject()
                .startObject("_all").field("enabled", "false").endObject()
                .startObject("properties")
                .startObject("user")
                .field("store", true)
                .field("type", "string")
                .field("index", "not_analyzed")
                .endObject()
                .startObject("message")
                .field("store", true)
                .field("type","string")
                .field("index", "analyzed")
                .field("analyzer", "standard")
                .endObject()
                .startObject("price")
                .field("store", true)
                .field("type", "float")
                .endObject()
                .startObject("nv1")
                .field("store", true)
                .field("type", "integer")
                .field("index", "no")
                .field("null_value", 0)
                .endObject()
                .startObject("nv2")
                .field("store", true)
                .field("type", "integer")
                .field("index", "not_analyzed")
                .field("null_value", 10)
                .endObject()
                .startObject("tid")
                .field("store", true)
                .field("type", "string")
                .field("index", "not_analyzed")
                .endObject()
//                               .startObject("location")
//                                    .field("store", true)
//                                  .field("type", "geo_point")
//                                  .field("lat_lon", true)
//                                  .field("geohash", true)
//                                  .field("geohash_prefix", true)
//                                  .field("geohash_precision", 7)
//                               .endObject()
//                               .startObject("shape")
//                                    .field("store", true)
//                                  .field("type", "geo_shape")
//                                  .field("geohash", true)
//                                  .field("geohash_prefix", false)
//                                  .field("geohash_precision", 7)
//                               .endObject()
                .startObject("endTime")
                .field("type", "date")
                .field("store", true)
                .field("index", "not_analyzed")
                //2015-08-21T08:35:13.890Z
                .field("format", "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd'T'HH:mm:ss.SSSZ")
                .endObject()
                .startObject("date")
                .field("type", "date")
//                                  .field("store", true)
//                                  .field("index", "not_analyzed")
                //2015-08-21T08:35:13.890Z
//                                  .field("format", "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd'T'HH:mm:ss.SSSZ")
                .endObject()
                .endObject()
                .endObject()
                .endObject();
        client.admin().indices()
                .preparePutMapping(indexName)
                .setType(documentType)
                .setSource(mapping)
                .execute().actionGet();
    }


}
```

### 3.11 查看head集群信息，成功了。
![图 es-head-cluster](https://psiitoy.github.io/img/blog/esdocker/es-head-cluster.png)


### 3.12 停止并删除所有docker容器
> 加`-f`参数强制停止删除

```bash
$ docker rm -f $(docker ps -aq)
b1b17d0f41eb
55d9259ff921
ff4d42575213
66ca0270ef44
ddfbb7ef65bc

```

---------------
