---
layout: post
title: "[总结]jmm总结"
date: 2016-07-25 21:15:06 
categories: 
    - 总结
tags:
    - jmm
---

jmm总结

<!--more-->

每个线程都有自己的执行空间(即工作内存)，线程执行的时候用到某变量，首先要将变量从主内存拷贝的自己的工作内存空间，
然后对变量进行操作：读取，修改，赋值等，这些均在工作内存完成，操作完成后再将变量写回主内存；

volatile关键字

当变量被某个线程A修改值之后，其它线程比如B若读取此变量的话，立刻可以看到原来线程A修改后的值



深度解析：
处理器为了提高处理速度，不直接和内存进行通讯，而是先将系统内存的数据读到内部缓存（L1,L2或其他）后再进行操作，
但操作完之后不知道何时会写到内存；如果对声明了Volatile变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，
将这个变量所在缓存行的数据写回到系统内存。但是就算写回到内存，如果其他处理器缓存的值还是旧的，
再执行计算操作就会有问题，所以在多处理器下，为了保证各个处理器的缓存是一致的，就会实现缓存一致性协议，
每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，
就会将当前处理器的缓存行设置成无效状态，当处理器要对这个数据进行修改操作的时候，
会强制重新从系统内存里把数据读到处理器缓存里。



-------------------
Volatile的使用优化
著名的Java并发编程大师Doug lea在JDK7的并发包里新增一个队列集合类LinkedTransferQueue，他在使用Volatile变量时，用一种追加字节的方式来优化队列出队和入队的性能。

追加字节能优化性能？这种方式看起来很神奇，但如果深入理解处理器架构就能理解其中的奥秘。让我们先来看看LinkedTransferQueue这个类，它使用一个内部类类型来定义队列的头队列（Head）和尾节点（tail），而这个内部类PaddedAtomicReference相对于父类AtomicReference只做了一件事情，就将共享变量追加到64字节。我们可以来计算下，一个对象的引用占4个字节，它追加了15个变量共占60个字节，再加上父类的Value变量，一共64个字节。

为什么追加64字节能够提高并发编程的效率呢？ 因为对于英特尔酷睿i7，酷睿， Atom和NetBurst， Core Solo和Pentium M处理器的L1，L2或L3缓存的高速缓存行是64个字节宽，不支持部分填充缓存行，这意味着如果队列的头节点和尾节点都不足64字节的话，处理器会将它们都读到同一个高速缓存行中，在多处理器下每个处理器都会缓存同样的头尾节点，当一个处理器试图修改头接点时会将整个缓存行锁定，那么在缓存一致性机制的作用下，会导致其他处理器不能访问自己高速缓存中的尾节点，而队列的入队和出队操作是需要不停修改头接点和尾节点，所以在多处理器的情况下将会严重影响到队列的入队和出队效率。Doug lea使用追加到64字节的方式来填满高速缓冲区的缓存行，避免头接点和尾节点加载到同一个缓存行，使得头尾节点在修改时不会互相锁定。

那么是不是在使用Volatile变量时都应该追加到64字节呢？不是的。在两种场景下不应该使用这种方式。第一：缓存行非64字节宽的处理器，如P6系列和奔腾处理器，它们的L1和L2高速缓存行是32个字节宽。第二：共享变量不会被频繁的写。因为使用追加字节的方式需要处理器读取更多的字节到高速缓冲区，这本身就会带来一定的性能消耗，共享变量如果不被频繁写的话，锁的几率也非常小，就没必要通过追加字节的方式来避免相互锁定。

-------------------

false-sharing

class {int x ,int y}  x和y被放在同一个高速缓存区，如果一个线程修改x；那么另外一个线程修改y，必须等待x修改完成后才能实施。

当多核CPU线程同时修改在同一个高速缓存行各自独立的变量时，会不自不觉地影响性能，这就发生了伪共享False sharing，伪共享是性能的无声杀手。
这里强调多核，是因为单核CPU模拟出的多线程不会严格意义上同时访问缓存行，所以性能影响不大
解决方便是将高速缓存剩余的字节填充填满(pad)，确保不发生多个字段被挤入一个高速缓存区，下面测试结果图就是和填充后性能比较。
实现字节填充的框架有 Disruptor，在RingBuffer中实现填充。关于Disruptor可见infoQ这个视频，用1毫秒的延时得到100K+ TPS吞吐量

Disruptor


Disruptor没有像JDK的LinkedBlockQueue等那样使用锁，针对CPU高速缓存进行了优化。

<font color=red>原来我们以为多个线程同时写一个类的字段会发生争夺，这是多线程基本原理，所以使用了锁机制，保证这个共用字段(资源)能够某个时刻只能一个线程写，但是这样做的坏处是：有可能发生死锁。</font>

比如1号线程先后访问共享资源A和B；而2号线程先后访问共享资源B和A，因为在资源A和资源B都有锁，那么1号在访问资源A时，资源A上锁了，准备访问资源B，但是无法访问，因为与此同时；而2号线程在访问资源B，资源B锁着呢，正准备访问资源A，发现资源A被1号线程锁着呢，结果彼此无限等待彼此下去，死锁类似逻辑上自指悖论。

所以，锁是坏的，破坏性能，锁是并发计算的大敌。

我们回到队列上，一把一个队列有至少两个线程：生产者和消费者，这就具备了资源争夺的前提，这两个线程一般彼此守在队列的进出两端，表面上好像没有访问共享资源，实际上队列存在两个共享资源：队列大小或指针.

除了共享资源写操作上存在资源争夺问题外，Disruptor的LMAX团队发现<font color=red>Java在多核CPU情况下有伪共享问题：<font>
CPU会把数据从内存加载到高速缓存中 ,这样可以获得更好的性能，高速缓存默认大小是64 Byte为一个区域，<font color=red>CPU机制限制只能一个CPU的一个线程访问(写)这个高速缓存区。</font>

CPU在将主内存中数据加载到高速缓冲时，如果发现被加载的数据不足64字节，那么就会加载多个数据，以填满自己的64字节，悲催就发生了，恰恰hotspot JVM中对象指针等大小都不会超过64字节，这样一个高速缓冲中可能加载了两个对象指针，一个CPU一个高速缓冲，双核就是两个CPU各自一个高速缓冲，那么两个高速缓冲中各有两个对象指针，都是指向相同的两个对象。

<font color=red>因为一个CPU只能访问(写)自己高速缓存区中数据，相当于给这个数据加锁，那么另外一个CPU同时访问自己高速缓冲行中同样数据时将会被锁定不能访问。</font>

这就发生与锁机制类似的性能陷进，`Disruptor的解决办法是填满高速缓冲的64字节，不是对象指针等数据不够64字节吗？那么加一些字节填满64字节，这样CPU将数据加载到高速缓冲时，就只能加载一个了，刚刚好啊。`

所以，尽管两个线程是在写两个不同的字段值，也会因为双核CPU底层机制发生伪装的共享，并没有真正共享，其实还是排他性的独享。

现在我们大概知道RingBuffer是个什么东东了：
1.ring buffer是一个大的数组.
2.RingBuffer里所有指针都是Java longs (64字节) 不断永远向前计数，如后面图，不断在圆环中循环。
3.RingBuffer只有当前序列号，没有终点序列号，其中数据不会被取出后消除，这样以便实现从过去某个序列号到当前序列号的重放，这样当消费者说没有接受到生产者发送的消息，生产者还可以再次发送，这点是一种原子性的“事务”机制。