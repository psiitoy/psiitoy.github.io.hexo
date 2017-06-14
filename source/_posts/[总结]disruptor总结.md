---
layout: post
title: "[总结]disruptor总结"
date: 2016-07-25 21:15:06 
categories: 
    - 总结
tags:
    - disruptor
    - 高并发
    - 开源框架
---

disruptor总结

<!--more-->

有说
1. 正如前面说的，预分配RingBuffer，防止GC。
2. 去掉锁，因为锁太耗时了，可以用CAS指令或者内存屏障达到这一目的。
3. 缓存填充，这样可以避免伪共享。cpu从不直接读取主存，都是通过cache间接访问主存。cache line是能被cache处理的
最小单元，典型的大小为32,64,128bytes

也有说
1.padding 避免伪共享 槽挂载的对象 槽 生产者seq 消费者每个seq 都作了padding 避免false sharing
2.无锁，入环时候 无锁 因为都是seq++ 做cas 失败的重试 非阻塞 (多线程访问都是通过seq，cas取代了锁)
3.避免槽gc 新产生的对象挂载到槽上覆盖就行，跟传统阻塞队列比较，减少gc负担。
4.队列采用数组结构（环形数组） 直接取余获取对象，取代了链表结构，寻址更快。

生产过快 或者消费过快 但是 还是都会阻塞!还是通过锁！
阻塞策略如下
BlockingWaitStrategy notify
这个策略的内部适用一个锁和条件变量来控制线程的执行和等待（Java基本的同步方法）。
SleepingWaitStrategy 
它的方式是循环等待并且在循环中间调用LockSupport.parkNanos(1)来睡眠
YieldingWaitStrategy
会循环等待sequence增加到合适的值。循环中调用Thread.yield()允许其他准备好的线程执行。
BusySpinWaitStrategy 
循环不让出cpu，同时也是对部署环境要求最高的策略。


---------------------------------------------

给一个最简单的disruptor，single consumer single producer的实现，
不需要同步锁：共享的变量：
int volatile readBarrier;
Data[] ring = new Data[A_BIG_NUMBER];
Producer
 线程:int writeCount;void produce(Data newItem) 
 {  ring[++writeCount % ring.size() ] = newItem;  readBarrier++;}
 Consumer 线程:void run() 
 {  int currentRead;  while (true) {   
  if (readBarrier > currentRead) {       
   consume (ring [ currentRead % ring.size() ];      
     currentRead++;    }  }}
     关键点在于：Consumer线程是只读的，所以理论上并不需要锁的参与，
     只要控制好readBarrier增量的时机，而Consumer线程只要一直轮询这个变量即可。


------------------------------------------

网上译文不够看啊，而且版本都升了，要想完全弄懂必须攻克源码 以下个人理解 本质上是个ringbuffer, 
buffer(就是数组)做过优化防止JVM伪共享，lock free 是通过CAS自旋(JUC用烂的技巧没啥)，
注意不是wait free，多线程并发获取buffer中的序号，在这里需要CAS，把事件放入槽中，
工作线程调度是交给jdk 线程池，只要buffer中有事件，就不停提交给线程池所以就是一个牛逼的临界区，无锁解决多线程读写


-------------------------------------------

1 一个64位的全局变量用来产生sequence number. writer每次写之前都有acquire一个unique的sequence number. 
方法就是把这个变量atomic的加一。2 writer用sequence取膜后得到要写位置，然后检查最小的reader的sequence
 number(每个reader都有私有的sequence number,记录自己消费到哪里了），保证不会outpace任何reader。
 写完后，要cas一个global的sequence number让写入内容可见。（if myseq == Gseq then Gseq ++ else retry).
 3每个reader有自己私有的sequence number, 读之前和Gseq比较避免超过writers.