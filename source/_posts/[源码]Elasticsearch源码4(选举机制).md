---
layout: post
title: "[源码]Elasticsearch源码4(选举机制)"
date: 2017-08-12 20:15:06 
categories: 
    - 源码
tags:
    - es
---

本文简述了ES选举中应用相对于Paxos简单了许多的[Bully](https://en.wikipedia.org/wiki/Bully_algorithm)算法，感谢[elasticsearch的master选举机制](http://www.cnblogs.com/zziawanblog/p/6577383.html),[zenDiscovery和master选举](http://www.cnblogs.com/zziawanblog/p/6583107.html)的精彩介绍

<!--more-->

## 一、前言

## 二、Bully and Paxos

### 2.1 Bully

- Bully 

  + 对所有可以成为master的节点根据nodeId排序，每次选举每个节点都把自己所知道节点排一次序，然后选出第一个（第0位）节点，暂且认为它是master节点。
  
  + 如果对某个节点的投票数达到一定的值（可以成为master节点数n/2+1）并且该节点自己也选举自己，那这个节点就是master。否则重新选举。
  
  + 对于brain split问题，需要把候选master节点最小值设置为可以成为master节点数n/2+1（quorum）


- 通过比较节点能“看到”的候选master数量和配置的最小值来确定是否可以进行选举，如果数量不够会导致选举不能进行，这样就可以保证集群不会被分裂。下面以一个图(图片来自于elasticsearch官网)来说明:

![图 es-bully](https://psiitoy.github.io/img/blog/essourcecode/es-bully.jpg)

### 2.2 Paxos

![图 fle-flow](https://psiitoy.github.io/img/blog/essourcecode/fle-flow.png)


