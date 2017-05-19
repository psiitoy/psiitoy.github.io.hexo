---
layout: post
title: "[总结]canal otter总结"
date: 2017-02-15 09:36:06 
description: "otter总结"
categories: 
    - 总结
tags:
    - otter
    - canal
---

otter总结

<!--more-->

otter原理描述：

 
1.基于Canal开源产品，获取数据库增量日志数据。 什么是Canal, 请点击
2.典型管理系统架构，manager(web管理)+node(工作节点)
     a. manager运行时推送同步配置到node节点
     b. node节点将同步状态反馈到manager上
3.基于zookeeper，解决分布式状态调度的，允许多node节点之间协同工作.

---------------
组件解释：
　　canal：
　　什么是canal？  otter之前开源的一个子项目，开源链接地址：http://github.com/alibaba/canal
　　定位：基于数据库增量日志解析，提供增量数据订阅&消费，目前主要支持了mysql
          工作原理：
          原理相对简单：类似MYSQL原有的主从复制机制。
          1.canal模拟mysql slave 的交互协议，伪装自己为mysql slave，向mysql master发送dump协议
          2.mysql master收到dump请求，开始推送binary log给slave（也就是canal）
          3.canal解释binary log 对象（原始为byte流）
          相关文档：
          See the wiki page for : wiki文档  https://github.com/alibaba/canal/wiki
          
          
          
          
          
     
 Issues
 pipeline与映射关系是一对一的模式好，还是一对多的关系好，为什么？
 当然1对多的好啊，减少对mysql库dump binlog的链接
 
 单向同步，将数据同步发送到kafka
 可以使用canal
 
 我配置的双A，两边同时修改同一条数据，会出现循环修改，请问一下原因？
 有影响的，不支持事务的引擎，binlog生成会和retl_mark的事务分离，导致无法判断是否是回环同步
 
 通过源码只能同步库名与表名相同的表结构，有没有其他方式支持库名或表名不同的表结构同步。
 DDL同步目前有约束，要求库名和表名都一致
 
 您好： 可在扩展功能自己写jdbc进行数据库查询及处理 ， 请问 能 提供一下目前系统 源和目标的 jdbc connect 如何获取方法吗？ 谢谢 ！
 有的，继承DataSourceFetcherAware
 
 一个mysql包含10个要同步的DB，以下哪种方案更好？
 1、10个channel，每个channel都是 1个pipeline，1个canal，1个db映射 （一共建立10个canal）
 2、1个channel， 1个pipile, 1个canal , 10个db映射(只有1个canal)
 2更好
 
 otter那里可以指定位置开始同步吗
 可以，配置管理-》canal配置，位点自定义设置勾中复选框，填入位点信息即可。