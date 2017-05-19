---
layout: post
title: "[总结]java锁"
date: 2016-07-25 21:15:06 
description: "java锁总结"
categories: 
    - 总结
tags:
    - lock
---

java锁总结

<!--more-->

Ps：说白了ReentrantLock就是基于Sync的，而Sync就是一种AQS，其中核心机制AQS都实现好了。

3.    Sync及AQS的核心实现（源码级别）

AQS核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源的设置为锁定状态。
如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，
即将暂时获取不到锁的线程加入到队列中。



wait()  释放cpu,释放占有的对象锁(线程进入等待池)
sleep() 释放cpu,不释放锁
notify() 调用后，不会立即释放锁，而是继续执行代码直到sync块中的代码全部执行完毕才会释放锁。
notifyAll() 环形所有wait的线程。

wait() notify()必须在同步块中使用(synchronized)。
await() signal() 是condition的方法。
park() unpark() 指定某一线程立即执行

一、synchronized的实现：

synrhronized关键字简洁、清晰、语义明确，因此即使有了Lock接口，使用的还是非常广泛。
其应用层的语义是可以把任何一个非null对象 作为"锁"，当synchronized作用在方法上时，锁住的便是对象实例（this）；
当作用在静态方法时锁住的便是对象对应的Class实例，因为 Class数据存在于永久带，因此静态方法锁相当于该类的一个全局锁；
当synchronized作用于某一个对象实例时，锁住的便是对应的代码块。在 HotSpot JVM实现中，锁有个专门的名字：对象监视器。 

在java虚拟机中，每个对象和类在逻辑上都是和一个监视器相关联的。 
对于对象来说，相关联的监视器保护对象的实例变量。 
对于类来说，监视器保护类的类变量。


公平 非公平
当有线程竞争锁时，该线程会首先尝试获得锁，这对于那些已经在队列中排队的线程来说显得不公平，这也是非公平锁的由来，
与synchronized实现类似，这样会极大提高吞吐量。 如果已经存在Running线程，则新的竞争线程会被追加到队尾，
具体是采用基于CAS的Lock-Free算法，因为线程并发对Tail调用CAS可能会 导致其他线程CAS失败，解决办法是循环CAS直至成功。



synchronized和lock比较

从代码层角度来说：

Lock是基于在语言层面实现的锁，Lock锁可以被中断，支持定时锁，虽然我们总是在一个finally块中释放锁，
但是其实我们可以很随意的释放锁，如果安全的话。Synchronized是基于JVM实现的，我们称之为对象的内置锁，
Java中的每一个对象都可以作为锁。对于同步方法，锁是当前实例对象。对于静态同步方法，锁是当前对象的Class对象。
对于同步方法块，锁是Synchonized括号里配置的对象。当一个线程访问同步代码块时，它首先必须得到锁，
退出或抛出异常时必须释放锁。JVM基于进入和退出Monitor对象来实现方法同步和代码块同步，但两者的实现细节不一样。
代码块同步是使用monitorenter和monitorexit指令实现，而方法同步是使用另外一种方式实现的，
细节在JVM规范里并没有详细说明，但是方法的同步同样可以使用这两个指令来实现。
monitorenter指令是在编译后插入到同步代码块的开始位置，而monitorexit是插入到方法结束处和异常处，
每个monitorenter必须有一个monitorexit对应。任何对象都有一个 monitor 与之关联，当且一个monitor 被持有后，
它将处于锁定状态。线程执行到 monitorenter 指令时，将会尝试获取对象所对应的 monitor 的所有权，即尝试获得对象的锁。
其实无论是Lock还是Synchronized，他们加锁的机制，使用的机器指令，系统调用都基本一致。

 

效率上的区别：

当竞争不是很激烈的时候Synchronized使用的是轻量级锁或者偏向锁，这两种锁都能有效减少轮询或者阻塞的发生，
相比与Lock仍旧要将未获得锁的线程放入等待队列阻塞带来的上下文切换的开销，此时Synchronized效率会高些，
当竞争激烈的时候Synchronized会升级为重量级锁，由于Synchronized的出对速度相比Lock要慢，所以Lock的效率会更高些。
一般对于数据结构设计或者框架的设计都倾向于使用Lock而非Synchronized。


monitor跟锁的关系
同步块加上monitorenter(将会尝试获取对象所对应的 monitor 的所有权) monitorexit
