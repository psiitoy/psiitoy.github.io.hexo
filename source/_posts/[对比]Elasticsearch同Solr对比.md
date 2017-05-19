---
layout: post
title: "[对比]Elasticsearch同Solr对比"
date: 2016-04-27 21:15:06 
description: "Elasticsearch同Solr对比"
categories: 
    - 对比
tags:
    - es
    - solr
---

Elasticsearch同Solr对比

<!--more-->


二者安装都很简单；
    Solr 利用 Zookeeper 进行分布式管理，而 Elasticsearch 自身带有分布式协调管理功能;
    Solr 支持更多格式的数据，而 Elasticsearch 仅支持json文件格式；
    Solr 官方提供的功能更多，而 Elasticsearch 本身更注重于核心功能，高级功能多有第三方插件提供；
    Solr 在传统的搜索应用中表现好于 Elasticsearch，但在处理实时搜索应用时效率明显低于 Elasticsearch。

Solr 是传统搜索应用的有力解决方案，但 Elasticsearch 更适用于新兴的实时搜索应用。

1. solr查询快，但更新索引时慢（即插入删除慢），用于电商等查询多的应用；

2.ES建立索引快（即查询慢），即实时性查询快，用于facebook新浪等搜索。



