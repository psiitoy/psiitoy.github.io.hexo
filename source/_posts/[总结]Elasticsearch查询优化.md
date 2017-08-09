---
layout: post
title: "[总结]Elasticsearch查询优化"
date: 2016-07-25 21:15:06 
categories: 
    - 总结
tags:
    - es
---

Elasticsearch查询优化

<!--more-->


es中的查询请求有两种方式，一种是简易版的查询，另外一种是使用JSON完整的请求体，叫做结构化查询（DSL）。

        GetResponse gResponse = client.prepareGet(indexName, typeName, docId)
                .execute()
                .actionGet();
