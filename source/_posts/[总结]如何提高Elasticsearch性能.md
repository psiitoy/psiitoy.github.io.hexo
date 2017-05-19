---
layout: post
title: "[总结]如何提高Elasticsearch性能"
date: 2016-04-27 21:15:06 
description: "如何提高Elasticsearch性能"
categories: 
    - 总结
tags:
    - es
---

如何提高Elasticsearch性能

<!--more-->

1. solr查询快，但更新索引时慢（即插入删除慢），用于电商等查询多的应用；

2.ES建立索引快（即查询慢），即实时性查询快，用于facebook新浪等搜索。


- 如何提高ES性能：
- 1.ES更新操作尽量提供批处理，减少写入次数；(bulk)
- 2.按商家维度分多套集群， 在集群里再按照商家维度分片；
- 3.热点集中问题解决方案：路由规则可定制化由ES团队负责；
- 4.提供数据重建reindex功能，要求mapping配置source为true；(方便重建数据)
- 5.ES团队要求，业务方需要提供至少10台物理机用以部署；
- 6.深分页条数建议控制在1万条以内，做分页用scroll方式实现，禁止跳转到大页码；
