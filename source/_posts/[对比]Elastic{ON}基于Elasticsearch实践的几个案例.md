---
layout: post
title: "[对比]Elastic{ON}基于Elasticsearch实践的几个案例"
date: 2017-08-23 21:15:06 
categories: 
    - 对比
tags:
    - es
---

本文总结了 `苏宁`、`美团点评`、`58到家`、`百度` 在Elastic{ON}大会上分享的，基于Elasticsearch实践的几个案例。感谢这些热衷分享的工程师们的开源精神。

<!--more-->

## 一、基于Kibana和ES的苏宁实时日志分析平台[苏宁]

### 1.1 架构

`Flume`(采集⽇志)+`Kafka`(数据通道,`ES river`插件消费)+`Elasticsearch`+`Kibana`的部署架构。

### 1.2 总结

- 第一次升级主要优化
  + 提出client节点(网关节点)的概念并分别部署到VM，client节点(只负责检索)，data节点(负责索引) 。
  + 关闭`_all`字段(节省存储空间、提升索引速度)。
  + 按业务进行垂直拆分
  
- 第二次升级主要优化
  + 使用`tribe`做多集群路由
  + `data`节点全部使用物理机
  + 按路径可降级
  + 按分析类型(统计型、检索型)进行集群拆分
  + 统计型集群使用`SSD`
  
- 第三次升级主要优化
  + 同时支持按业务、数据类型两个维度划分集群。
  + 接入流程自动化
  + `Cold/Hot` 节点不同硬件配置，历史数据低峰期迁移冷节点。
  + Exception数据单独存储，长期保留。
  + 多`kafka`
  
------------------------------  
  
## 二、搜索平台化实践之路[美团点评]

### 2.1 架构

- 集群部署的架构为
  + [user]业务应用集群
  + [man]数据库集群
  + [man]汇总慢日志集群
  + [man]集群监控集群
  + 各集群节点全部部署agent，同时对业务及群进行日志采集(含慢日志)

### 2.2 总结

- 2016Q3
  + 统一管理平台
  + Java客户端组件
  + 集群多写组件
  + 权限插件
  
- 2016Q4
  + 客户端读写监控
  + 慢查询监控
  + 集群准实时监控
  + 支持ES5.x版本
  
- 2017Q1
  + Sql客户端和操作界面
  + 多数据源自动化导入ES
  + 统计管理理    
  
## 三、58到家ES服务化实践[58到家]

### 3.1 架构

中间件化架构
  + `写`操作调用集中式服务(RPC+ORM+@ANO)
  + `读`操作经过DSLHandler进行优化(分布式)
  + 权限模块定位客户端连接那个集群。

### 3.2 总结
 
总结
- 服务化构建
> RPC，ORM，约束
 
- 地理理类集群
> 读写密集，减小数据集、内存、业务折中
 
- 商品类集群
> 查询快，隔离、路由、减少字段、定期优化
 
- 日志类集群
> 写频繁，批量、减少副本、角色分配、索引拆分

## 四、百度对Elasticsearch的优化改进[百度]

### 4.1 功能增强

- 功能改进
  + 分布式SQL查询层
  + 权限管理
  + Online schema change
  + `DistributedLog` 数据一致性
  + 多集群数据同步
  + 多租户资源隔离
  
### 4.2 总结
  
- 分布式SQL查询层
  + 提供标准SQL接口
  + 兼容MySQL协议
  + 兼容原始HTTP协议

- 权限管理系统
  + 如果是HttpClient-ES Engine走`ActionFilter`->AuthService
  + 如果是MysqlClient-SQL Enging走Analyzer->AuthService

- Online schema change (Baidu-ES reindex)
  + 每个分片内部本地`reindex`
  + 增加mapping version，插入时使用最新mapping，并使用internal version，防止覆盖，后台线程定期检测Mapping version增加，自动reindex
  + 动态调整reindex速度
  + 节点宕机、分片迁移、reindex自动恢复
  + 无需新建index，无需额外网络IO和磁盘空间
  + 支持在线服务，支持update
  + 查看每个分片reindex进度
  
- DistributedLog 数据一致性
  + 需求背景
    * 糯米、钱包等业务对ES服务可靠性要求很高
    * 不能容忍脑裂、数据不一致、丢数据等情况的发生
  + 需解决的问题
    * 元数据一致性（脑裂）
    * 强一致写
    * 强一致读
  + 解决方案：DistributedLog
    * Twitter 开源分布式日志服务
    * 高可用，高性能，强一致，可扩展
    
- 多集群数据同步
> Es Mirror Maker分发(主备) + version乐观锁(解决主备切时的冲突)

- 多租户资源隔离
  + 需求背景
    * 多个业务使用公共集群，CPU、内存、IO、JVM等相互影响
    * 云化部署
  + 设计实现
    * 每台物理机启动多个ES进程组成大集群
    * `cgroup` 对每个ES进程进行CPU、内存、IO等隔离
    * 引入tenement 概念，分配不同的ES节点为每个租户创建自己的虚拟集群
    * Allocation filter 限制租户的index只能创建在自己的节点上
    * 每个租户分配不同DB，隔离访问权限
    * username@tenement，租户命名空间隔离租户信息
    * 根据租户ID隔离settings，templates、nodes等