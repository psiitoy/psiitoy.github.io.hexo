---
layout: post
title: "[源码]Elasticsearch源码3(线程池)"
date: 2017-08-11 20:15:06 
categories: 
    - 源码
tags:
    - es
---

Elasticsearch源码3(线程池)

<!--more-->

## 一、前言

其实就是在`java.util.concurrent.ThreadPoolExecutor`基础上做了封装，我们就看下和JVM线程池的区别。

## 二、Executors and EsExecutors

### 2.1 首先先看JVM线程池

> 注意:如果 corePoolSize < workerNum < maximumPoolSize时 是优先入队列的，队列满才addWorker。
 
![图 threadpool](https://psiitoy.github.io/img/blog/essourcecode/threadpool.jpg)

### 2.2 对比的看都是如何对ThreadPoolExecutor进行的构造

* 以下为JVM线程池

```java
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
    
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }

    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }

```

* 以下为ES线程池

```java
    public static EsThreadPoolExecutor newFixed(String name, int size, int queueCapacity, ThreadFactory threadFactory) {
        BlockingQueue<Runnable> queue;
        if (queueCapacity < 0) {
            queue = ConcurrentCollections.newBlockingQueue();
        } else {
            queue = new SizeBlockingQueue<>(ConcurrentCollections.<Runnable>newBlockingQueue(), queueCapacity);
        }
        return new EsThreadPoolExecutor(name, size, size, 0, TimeUnit.MILLISECONDS, queue, threadFactory, new EsAbortPolicy());
    }
    
    public static EsThreadPoolExecutor newCached(String name, long keepAliveTime, TimeUnit unit, ThreadFactory threadFactory) {
        return new EsThreadPoolExecutor(name, 0, Integer.MAX_VALUE, keepAliveTime, unit, new SynchronousQueue<Runnable>(), threadFactory, new EsAbortPolicy());
    }    
    
    public static EsThreadPoolExecutor newScaling(String name, int min, int max, long keepAliveTime, TimeUnit unit, ThreadFactory threadFactory) {
        ExecutorScalingQueue<Runnable> queue = new ExecutorScalingQueue<>();
        // we force the execution, since we might run into concurrency issues in offer for ScalingBlockingQueue
        EsThreadPoolExecutor executor = new EsThreadPoolExecutor(name, min, max, keepAliveTime, unit, queue, threadFactory, new ForceQueuePolicy());
        queue.executor = executor;
        return executor;
    }

```

* 区别于差异
1) fixed(无限制)
- 使用SizeBlockingQueue(内部封装了LinkedTransferQueue无锁队列)代替LinkedBlockingQueue。
- ES 当`queueCapacity`<0 时队列设为无界。

2) cached(定长)
- 同jvm`cachedThreadPool`一样，只是拒绝策略自定义了EsAbortPolicy代替默认的AbortPolicy。
> 其实由于`maximumPoolSize`是MAX，所以基本不会发生拒绝(addWorker失败)。

3) scaling(可变大小)
- 这里创建的是一个无限制的queue，并且pool size的大小在最小最大值之间浮动。对于reject的处理是ForceQueuePolicy，即是将之放入queue中，这里的queue是无限制的。

es中可以通过`GET /_cat/thread_pool`来查看线程池状态信息。

## 二、es中使用的线程池

- CACHED
  + GENERIC：通用的操作，比如node的discovery，上面也说了，默认keep alive时间是5min;

- FIXED
  + LISTENER：主要用作java client的执行，默认大小halfProcMaxAt10；
  + GET：用作get操作，默认大小availableProcessors，queue_size为1000；
  + INDEX：用作index或delete操作，默认大小availableProcessors，queue_size为200；
  + BULK：用作bulk操作，默认大小为availableProcessors，queue_size为50；
  + SEARCH：用作count或是search操作，默认大小((availableProcessors * 3) / 2) + 1；queue_size为1000；
  + SUGGEST：用作suggest操作，默认大小availableProcessors，queue_size为1000；
  + PERCOLATE：用作percolate，默认大小为availableProcessors，queue_size为1000；
  + FORCE_MERGE：用作force_merge操作(2.1之前叫做optimize)，默认大小为1；

- SCALING
  + MANAGEMENT：用作ES的管理，比如集群的管理；默认大小5，keep alive时间为5min；
  + FLUSH：用作flush操作，默认大小为halfProcMaxAt5，keep alive时间为5min；
  + REFRESH：用作refresh操作，默认大小为halfProcMaxAt10，keep alive时间为5min；
  + WARMER：用作index warm-up操作，默认大小为halfProcMaxAt5，keep alive时间为5min；
  + SNAPSHOT:用作snapshot操作，默认大小为halfProcMaxAt5，keep alive时间为5min；
  + FETCH_SHARD_STARTED：用作fetch shard开始操作，默认大小availableProcessors * 2，keep alive时间为5min；
  + FETCH_SHARD_STORE：用作fetch shard存储操作，默认大小availableProcessors * 2，keep alive时间为5min；