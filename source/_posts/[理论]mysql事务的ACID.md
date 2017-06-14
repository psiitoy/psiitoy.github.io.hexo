---
layout: post
title: "[理论]mysql事务的ACID"
date: 2016-07-25 21:15:06 
categories: 
    - 理论
tags:
    - mysql
    - 事务
---

mysql事务的ACID

MySQL中隔离级别RC与RR的区别


<!--more-->
-------------------------------

个人理解
RU（读未提交） 存在并发问题(脏读) RC（读已提交 不可重复读） RR(MVCC 可重复读) 串行(读写都加锁)
RU 可见性问题 还没提交 别的事务就可见了 根本不可用！ 不谈
RC 提交之后别的事务可见。但是事务中多次查询统一条件但是结果不一样，不可重复读。
RR MVCC 数据发生更改时会产生一个副本包含生效时间跟失效时间，如果别的事务读到该条数据那么会对比当前时间跟副本的生效时间是否落后（是否又发生变更了），如果变更了那么事务失效。所谓的CAS乐观锁
串行 库锁 只能由一个事务执行


1. 数据库事务ACID特性
 
数据库事务的4个特性：
 
原子性(Atomic): 事务中的多个操作，不可分割，要么都成功，要么都失败； All or Nothing.
 
一致性(Consistency): 事务操作之后, 数据库所处的状态和业务规则是一致的; 比如a,b账户相互转账之后，总金额不变；
 
隔离性(Isolation): 多个事务之间就像是串行执行一样，不相互影响;
 
持久性(Durability): 事务提交后被持久化到永久存储.
 
2. 隔离性
 
其中 隔离性 分为了四种：
 
READ UNCOMMITTED：可以读取未提交的数据，未提交的数据称为脏数据，所以又称脏读。此时：幻读，不可重复读和脏读均允许；
 
READ COMMITTED：只能读取已经提交的数据；此时：允许幻读和不可重复读，但不允许脏读，所以RC隔离级别要求解决脏读；
 
REPEATABLE READ：同一个事务中多次执行同一个select,读取到的数据没有发生改变；此时：允许幻读，但不允许不可重复读和脏读，所以RR隔离级别要求解决不可重复读；
 
SERIALIZABLE: 幻读，不可重复读和脏读都不允许，所以serializable要求解决幻读；
 
3. 几个概念
 
脏读：可以读取未提交的数据。RC 要求解决脏读；
 
不可重复读：同一个事务中多次执行同一个select, 读取到的数据发生了改变(被其它事务update并且提交)；
 
可重复读：同一个事务中多次执行同一个select, 读取到的数据没有发生改变(一般使用MVCC实现)；RR各级级别要求达到可重复读的标准；
 
幻读：同一个事务中多次执行同一个select, 读取到的数据行发生改变。也就是行数减少或者增加了(被其它事务delete/insert并且提交)。SERIALIZABLE要求解决幻读问题；
 
这里一定要区分 不可重复读 和 幻读：
 
不可重复读的重点是修改:
 
同样的条件的select, 你读取过的数据, 再次读取出来发现值不一样了
 
幻读的重点在于新增或者删除:
 
同样的条件的select, 第1次和第2次读出来的记录数不一样
 
从结果上来看, 两者都是为多次读取的结果不一致。但如果你从实现的角度来看, 它们的区别就比较大：
 
对于前者, 在RC下只需要锁住满足条件的记录，就可以避免被其它事务修改，也就是 select for update, select in share mode; RR隔离下使用MVCC实现可重复读；
 
对于后者, 要锁住满足条件的记录及所有这些记录之间的gap，也就是需要 gap lock。
 
而ANSI SQL标准没有从隔离程度进行定义，而是定义了事务的隔离级别，同时定义了不同事务隔离级别解决的三大并发问题：

Isolation Level
	

Dirty Read
	

Unrepeatable Read
	

Phantom Read

Read UNCOMMITTED
	

YES
	

YES
	

YES

READ COMMITTED
	

NO
	

YES
	

YES

READ REPEATABLE
	

NO
	

NO
	

YES

SERIALIZABLE
	

NO
	

NO
	

NO
4. 数据库的默认隔离级别
 
除了MySQL默认采用RR隔离级别之外，其它几大数据库都是采用RC隔离级别。
 
但是他们的实现也是极其不一样的。Oracle仅仅实现了RC 和 SERIALIZABLE隔离级别。默认采用RC隔离级别，解决了脏读。但是允许不可重复读和幻读。其SERIALIZABLE则解决了脏读、不可重复读、幻读。
 
MySQL的实现：MySQL默认采用RR隔离级别，SQL标准是要求RR解决不可重复读的问题，但是因为MySQL采用了gap lock，所以实际上MySQL的RR隔离级别也解决了幻读的问题。那么MySQL的SERIALIZABLE是怎么回事呢？其实MySQL的SERIALIZABLE采用了经典的实现方式，对读和写都加锁。
 
5. MySQL 中RC和RR隔离级别的区别
 
MySQL数据库中默认隔离级别为RR，但是实际情况是使用RC 和 RR隔离级别的都不少。好像淘宝、网易都是使用的 RC 隔离级别。那么在MySQL中 RC 和 RR有什么区别呢？我们该如何选择呢？为什么MySQL将RR作为默认的隔离级别呢？
 
5.1 RC 与 RR 在锁方面的区别
 
1> 显然 RR 支持 gap lock(next-key lock)，而RC则没有gap lock。因为MySQL的RR需要gap lock来解决幻读问题。而RC隔离级别则是允许存在不可重复读和幻读的。所以RC的并发一般要好于RR；
 
2> RC 隔离级别，通过 where 条件过滤之后，不符合条件的记录上的行锁，会释放掉(虽然这里破坏了“两阶段加锁原则”)；但是RR隔离级别，即使不符合where条件的记录，也不会是否行锁和gap lock；所以从锁方面来看，RC的并发应该要好于RR；
 
5.2 RC 与 RR 在复制方面的区别
 
1> RC 隔离级别不支持 statement 格式的bin log，因为该格式的复制，会导致主从数据的不一致；只能使用 mixed 或者 row 格式的bin log; 这也是为什么MySQL默认使用RR隔离级别的原因。复制时，我们最好使用：binlog_format=row
 
2> MySQL5.6 的早期版本，RC隔离级别是可以设置成使用statement格式的bin log，后期版本则会直接报错；
 
5.3 RC 与 RR 在一致性读方面的区别
 
简单而且，RC隔离级别时，事务中的每一条select语句会读取到他自己执行时已经提交了的记录，也就是每一条select都有自己的一致性读ReadView; 而RR隔离级别时，事务中的一致性读的ReadView是以第一条select语句的运行时，作为本事务的一致性读snapshot的建立时间点的。只能读取该时间点之前已经提交的数据。
 
具体可以参加：MySQL 一致性读 深入研究
 
5.4 RC 支持半一致性读，RR不支持
 
RC隔离级别下的update语句，使用的是半一致性读(semi consistent)；而RR隔离级别的update语句使用的是当前读；当前读会发生锁的阻塞。
 
1> 半一致性读：
 
A type of read operation used for UPDATE statements, that is a combination of read committed and consistent read. When an UPDATE statement examines a row that is already locked, InnoDB returns the latest committed version to MySQL so that MySQL can determine whether the row matches the WHERE condition of the UPDATE. If the row matches (must be updated), MySQL reads the row again, and this time InnoDB either locks it or waits for a lock on it. This type of read operation can only happen when the transaction has the read committed isolation level, or when the innodb_locks_unsafe_for_binlog option is enabled.
 
简单来说，semi-consistent read是read committed与consistent read两者的结合。一个update语句，如果读到一行已经加锁的记录，此时InnoDB返回记录最近提交的版本，由MySQL上层判断此版本是否满足 update的where条件。若满足(需要更新)，则MySQL会重新发起一次读操作，此时会读取行的最新版本(并加锁)。semi-consistent read只会发生在read committed隔离级别下，或者是参数innodb_locks_unsafe_for_binlog被设置为true(该参数即将被废弃)。
 
对比RR隔离级别，update语句会使用当前读，如果一行被锁定了，那么此时会被阻塞，发生锁等待。而不会读取最新的提交版本，然后来判断是否符合where条件。
 
半一致性读的优点：
 
减少了update语句时行锁的冲突；对于不满足update更新条件的记录，可以提前放锁，减少并发冲突的概率。
 
Oracle中的update好像有“重启动”的概念。 