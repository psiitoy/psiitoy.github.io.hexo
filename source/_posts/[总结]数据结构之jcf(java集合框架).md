---
layout: post
title: "[总结]数据结构之jcf(java集合框架)"
date: 2017-06-18 21:15:06 
categories: 
    - 总结
tags:
    - 数据结构
    - jdk
    - jcf
---

本文比较全面的集中整理，总结了java集合框架jcf(Java Collection Framework)。

<!--more-->

## 一、前言

### 1.1 数据结构分类

* 数据元素相互之间的关系称为结构。有四类基本结构：集合、线性结构、树形结构、图状结构。

1) 集合结构:
* 一组`对象`，无其他特点。
2) 线性结构:
* 元素之间存在`一对一`关系常见类型有: 数组,链表,队列,栈。
> 它们之间在操作上有所区别.例如:链表可在任意位置插入或删除元素,而队列在队尾插入元素,队头删除元素,栈只能在栈顶进行插入,删除操作.

3) 树形结构:
* 元素之间存在`一对多`关系,常见类型有:树(有许多特例:二叉树、平衡二叉树、查找树等)
4) 图形结构:
* 元素之间存在`多对多`关系,略。

### 1.2 java中的数据结构JCF(Java Collection Framework)特点

* 底层是数组的大多会有 `初始容量参数initialCapacity`(如ArrayList)，`负载因子loadFactor`(如HashMap)，大于阈值(`当前容量` 或者 带负载因子的是`当前容量*loadFactor`)时会进行扩容。
* RandomAccess(随机访问)标识的List使用for循环做遍历以提升性能。
* 现所有的Set的一些实现都是相应Map的一种封装。
* 并发容器多数不能使用null值。
* 通常含有Concurrent，CopyOnWrite，Blocking的是线程安全的，但是这些线程安全通常是有条件的，所以在使用前一定要仔细阅读文档。

### 1.3 第三方的数据结构(简要介绍)

LRUCache，LongHashMap，AtomicLongMap

> [Apache的数据结构](http://commons.apache.org/collections/)
[大对象的数据结构](https://github.com/HugeCollections/Collections)
[Google的Guava](http://ifeve.com/google-guava/)

## 二、JCF

### 2.1 接口

* 先看java.util包里面最基本的数据结构，先看接口都有什么

```
Collection
|-List
|-Queue
|-Set

```

```
Map
|-SortedMap
|-ConcurrentMap

```

### 2.2 UML类图

* Collection

![图 es5-uml1](https://psiitoy.github.io/img/blog/knowledge/datastructure/uml/jdk-collection.png)

> java.util.concurrent里面实现的并发数据结构

![图 es5-uml1](https://psiitoy.github.io/img/blog/knowledge/datastructure/uml/jdk-collection-concurrent.png)

* Map

![图 es5-uml1](https://psiitoy.github.io/img/blog/knowledge/datastructure/uml/jdk-map.png)

> java.util.concurrent里面实现的并发数据结构

![图 es5-uml1](https://psiitoy.github.io/img/blog/knowledge/datastructure/uml/jdk-map-concurrent.png)

### 2.3 详解

#### 2.3.1 接口详解

##### Collection

Iterable (可迭代，其实现类编译器代理实现了forEach语法糖，代理用户使用迭代器进行外部迭代)
> 参见：[Java for-each循环解惑](http://www.importnew.com/11038.html)

Collection (集合，代表一组Object)
> Collection是最基本的集合接口，一个Collection代表一组Object，即Collection的元素（Elements）。一些Collection允许相同的元素而另一些不行。一些能排序而另一些不行。Java SDK不提供直接继承自Collection的类，Java SDK提供的类都是继承自Collection的“子接口”如List和Set。 
所有实现Collection接口的类都必须提供两个标准的构造函数：无参数的构造函数用于创建一个空的Collection，有一个Collection参数的构造函数用于创建一个新的Collection，这个新的Collection与传入的Collection有相同的元素。后一个构造函数允许用户复制一个Collection。
如何遍历Collection中的每一个元素？不论Collection的实际类型如何，它都支持一个iterator()的方法，该方法返回一个迭代子，使用该迭代子即可逐一访问Collection中每一个元素。

List (有序的Collection,允许有相同的元素。)
> 除了具有Collection接口必备的iterator()方法外，List还提供一个listIterator()方法，返回一个ListIterator接口，和标准的Iterator接口相比，ListIterator多了一些add()之类的方法，允许添加，删除，设定元素，还能向前或向后遍历。

RandomAccess (随机存取，单独的接口，无继承)
> `随机存取`:就是存取第N个数据时，必须先访问前(N-1)个数据。如链表。`顺序存取`:就是存取第N个数据时，不需要访问前(N-1)个数据，直接就可以对第N个数据操作。如数组。
RandomAccess接口是一个标识接口，本身没有提供任何方法。主要的目的是为了标识出那些可以支持快速随机访问的List的实现。例如，根据是否实现RandomAccess接口在变量的时候选择不同的遍历实现，以提升性能。

Set (不能包含重复元素，最多有一个null元素)
> 如果一个Set中的可变元素改变了自身状态导致Object.equals(Object)=true将导致一些问题。
现所有的Set的一些实现都是相应Map的一种封装。

SortedSet (有序，唯一实现类是TreeSet,add不可比较对象会抛ClassCastException，除非在构造传入Comparator)
> SortedSet意思是“根据对象的比较顺序”，而不是“插入顺序”进行排序.

NavigableSet (可导航的Set 扩展了SortedSet，具有了搜索匹配元素的方法)

Queue (fifo先进先出)

Deque (双端队列)

[concurrent] BlockingQueue (阻塞队列，不接受null)
> 接口是在Queue基础上增加了两个操作,1：检索元素时等待队列变为非空,2：存储元素时等待空间变得可用。 
put(E) and take() 阻塞式的(根据类型不一样有的put不受阻)，通过condition做线程之间的通信。
drainTo():一次性从BlockingQueue获取所有可用的数据对象（还可以指定获取数据的个数），通过该方法，可以提升获取数据效率；不需要多次分批加锁或释放锁。
注意不要用Queue的声明add(E) and poll()。
ArrayBlockingQueue和LinkedBlockingQueue是两个最普通也是最常用的阻塞队列，一般情况下，在处理多线程间的生产者消费者问题，使用这两个类足以。
传送门：[阻塞队列的详解](http://www.cnblogs.com/tjudzj/p/4454490.html)

[concurrent] BlockingDeque (双端阻塞队列)

##### Map

Map (key value)
> Map接口提供3种集合的视图，Map的内容可以被当作一组key集合，一组value集合，或者一组key-value映射。

SortedMap

[concurrent] ConcurrentMap

#### 2.3.2 实现详解

##### List 

1) ArrayList (动态数组，查找快o(1)，插入慢，允许null元素)
> ArrayList实现了可变大小的数组。它允许所有元素，包括null。ArrayList没有同步。
  size，isEmpty，get，set方法运行时间为常数。但是add方法开销为分摊的常数，添加n个元素需要O(n)的时间。
  其他的方法运行时间为线性。每个ArrayList实例都有一个容量（Capacity），即用于存储元素的数组的大小。
  这个容量可随着不断添加新元素而自动增加，但是增长算法并没有定义。1.5倍扩容。
  当需要插入大量元素时，在插入前可以调用ensureCapacity方法来增加ArrayList的容量以提高插入效率。
  在ArrayList和Vector中，从一个指定的位置（通过索引）查找数据或是在集合的末尾增加、移除一个元素所花费的时间是一样的，这个时间我们用O(1)表示。但是，如果在集合的其他位置增加或移除元素那么花费的时间会呈线形增长：O(n-i)，其中n代表集合中元素的个数，i代表元素增加或移除元素的索引位置。为什么会这样呢？以为在进行上述操作的时候集合中第i和第i个元素之后的所有元素都要执行位移的操作。这一切意味着什么呢？
  这意味着，你只是查找特定位置的元素或只在集合的末端增加、移除元素，那么使用Vector或ArrayList都可以。如果是其他操作，你最好选择其他的集合操作类。比如，LinkList集合类在增加或移除集合中任何位置的元素所花费的时间都是一样的?O(1)，但它在索引一个元素的使用缺比较慢－O(i),其中i是索引的位置.使用ArrayList也很容易，因为你可以简单的使用索引来代替创建iterator对象的操作。LinkList也会为每个插入的元素创建对象，所有你要明白它也会带来额外的开销。
  add()方法性能的好坏取决于grow()方法的性能。当ArrayList对容量的需求超过当前数组的大小是，会进行数组扩容，扩容的过程中需要大量的数组复制，数组复制调用System.arraycopy（） native方法，操作效率是非常快的。

2) [过时] Vector (线程安全的ArrayList)
> 并发修改并调用Iterator的方法时将抛出ConcurrentModificationException。
2倍扩容。

3) [过时] Stack (继承自Vector，一个后进先出的堆栈。优先应使用 Deque<Integer> stack = new ArrayDeque<Integer>();)
> Stack提供5个额外的方法使得Vector得以被当作堆栈使用。基本的push和pop方法，还有peek方法得到栈顶的元素，empty方法测试堆栈是否为空，search方法检测一个元素在堆栈中的位置。Stack刚创建后是空栈。

4) LinkedList (链表，插入快，查找慢o(n)，允许null元素)
> LinkedList提供额外的get，remove，insert方法在LinkedList的首部或尾部。这些操作使LinkedList可被用作堆栈（stack），队列（queue）或双向队列（deque）。
LinkedList是基于链表实现，因此不需要维护容量大小，但是每次都新增元素都要新建一个Node对象，并进行一系列赋值，
在频繁系统调用中，对系统性能有一定影响。性能测试得出，在列表末尾增加元素，ArrayList比LinkedList性能要好，
因为数组是连续的，在末尾增加元素，只有在空间不足时才会进行数组扩容，大部分情况下追加操作效率还是比较高的。

5) [concurrent] CopyOnWriteArrayList (读多写少的并发场景，写时内存占用较大，数据最终一致。)
> 里面有一个ReentrantLock，每当add时，都锁住，把所有的元素都复制到一个新的数组上。
  只保证历遍操作是线程安全的，get操作并不保证，也就是说如果先得到size，再调用get(size-1)，
  有可能会失效.读的时候不需要加锁，添加的时候是需要加锁的，否则多线程写的时候会Copy出N个副本出来。

------------

##### Set

1) HashSet (基于HashMap实现，无序，根据哈希值查找Entry,能存一个null)

2) LinkedHashSet (插入有序)
> 根据哈希值来判断元素存贮的位置，同时使用链表来维护元素之前的顺序，所以他是有序的。迭代速度比HashSet好，插入删除查（因为需要维护前后元素的关系）

3) TreeSet (基于TreeMap实现，key需要实现comparator或构造比较器，实现排序。)

4) EnumSet

5) [concurrent] CopyOnWriteArraySet (简单包装了CopyOnWriteArrayList，注意这个Set的get的时间复杂度。)
> 实现完全依赖于CopyOnWriteArrayList

6) [concurrent] ConcurrentSkipListSet (包装了一个ConcurrentSkipListMap，参考HashSet。)
> Skip list(跳表）是一种可以代替平衡树的数据结构，默认是按照Key值升序的。Skip list让已排序的数据分布在多层链表中，以0-1随机数决定一个数据的向上攀升与否，通过“空间来换取时间”的一个算法，在每个节点中增加了向前的指针，在插入、删除、查找时可以忽略一些不可能涉及到的结点，从而提高了效率。
  从概率上保持数据结构的平衡比显示的保持数据结构平衡要简单的多。对于大多数应用，用Skip list要比用树算法相对简单。由于Skip list比较简单，实现起来会比较容易，虽然和平衡树有着相同的时间复杂度(O(logn)),但是skip list的常数项会相对小很多。Skip list在空间上也比较节省。一个节点平均只需要1.333个指针（甚至更少）。
  (1) 由很多层结构组成，level是通过一定的概率随机产生的。
  (2) 每一层都是一个有序的链表，默认是升序，也可以根据创建映射时所提供的Comparator进行排序，具体取决于使用的构造方法。
  (3) 最底层(Level 1)的链表包含所有元素。
  (4) 如果一个元素出现在Level i 的链表中，则它在Level i 之下的链表也都会出现。
  (5) 每个节点包含两个指针，一个指向同一链表中的下一个元素，一个指向下面一层的元素。
  ConcurrentSkipListMap具有Skip list的性质 ，并且适用于大规模数据的并发访问。多个线程可以安全地并发执行插入、移除、更新和访问操作。与其他有锁机制的数据结构在巨大的压力下相比有优势。
  
------------

***SortedSet NavigableSet***

1) TreeSet ...

2) [concurrent] ConcurrentSkipListSet ...

------------

##### Queue    

1) PriorityQueue (内部用一个数组来保存元素，但数组是以堆的形式来组织的，因此是有序的。)
> 堆(最小堆)的实现，优先级队列是不同于先进先出队列的另一种队列。每次从队列中取出的是具有最高优先权的元素。

2) LinkedList ...

3) ArrayDeque (内部用一个数组保存元素，有int类型head和tail的。)

4) [concurrent] ConcurrentLinkedQueue

------------

***BlockingQueue***

1) [concurrent] ArrayBlockingQueue (fifo)
> 基于数组实现的，在构造时需要指定容量，并可以选择是否需要公平性，如果公平参数被设置true，等待时间最长的线程会优先得到处理。（实现就是锁的FairSync）
性能上会付出代价，默认都是非公平锁。

2) [concurrent] LinkedBlockingQueue (fifo)
> 基于链表实现的，构造时不传容量`capacity`默认就是MAX_INTEGER大小。

3) [concurrent] DelayQueue (延迟队列，无界，期满才能提取，基于PriorityQueue实现，不允许null)
> 是一个存放Delayed 元素的无界阻塞队列，只有在延迟期满时才能从中提取元素。
该队列的头部是延迟期满后保存时间最长的 Delayed 元素。如果延迟都还没有期满，则队列没有头部，并且poll将返回null。
当一个元素的 getDelay(TimeUnit.NANOSECONDS) 方法返回一个小于或等于零的值时，则期满。
以下是延时接口

```java
public interface Delayed extends Comparable<Delayed> {  
     long getDelay(TimeUnit unit);  
}  

```

4) [concurrent] PriorityBlockingQueue (无界，优先级队列，基于PriorityQueue实现)
> 优先级队列，unfifo，不是先进先出的，包装的PriorityQueue基于堆，没有容量限制。

5) [concurrent] SynchronousQueue (同步队列，无缓冲的等待伪队列)
> 每个 put 必须等待一个 take，反之亦然。同步队列没有任何内部容量，甚至连一个队列的容量都没有。
不能在同步队列上进行 peek，因为仅在试图要取得元素时，该元素才存在。
可以选择公平模式。

6) [concurrent] LinkedTransferQueue (略)

------------

***BlockingDeque***

1) [concurrent] LinkedBlockingDeque

------------

***RandomAccess***

1) ArrayList ...

2) Vector ...

[concurrent] CopyOnWriteArrayList ...

------------

##### Map

1) HashMap (key和value允许为null)
> 通过initial capacity和load factor两个参数调整性能。通常缺省的load factor 0.75较好地实现了时间和空间的均衡。增大load factor可以节省空间但相应的查找时间将增大，这会影响像get和put这样的操作。
要同时复写equals方法和hashCode方法，而不要只写其中一个。
将HashMap视为Collection时（values()方法可返回Collection），其迭代子操作时间开销和HashMap的容量成比例。因此，如果迭代操作的性能相当重要的话，不要将HashMap的初始化容量设得过高，或者load factor过低。
冲突率低，性能好，冲突率高，退化成几个链表，性能极差。
node->treeNode(jdk8) 当某一个点上的链表长度达到8,会转化成红黑树，这样在极端情况下(所有的元素都在一个点上，整个就以链表)，一些操作的时间复杂度有O(n)变成了O(logn)。

2) [过时] Hashtable (同步，key和value都不能为null,遍历使用Enumerator)
> 首先说一下，HashMap和Hashtable的区别:Hashtable的大部分方法都实现了同步，而HashMap没有。因此，HashMap不是线程安全的。其次，Hashtable不允许key或value使用null值，而HashMap可以。第三是内部的算法不同，它们对key的hash算法和hash值到内存索引的映射算法不同。

3) LinkedHashMap (插入有序)
> 在Entry中增加before和after指针把HashMap中的元素串联起来，这样在迭代时，就可以按插入顺序遍历。
在HashMap的基础上，LinkedHashMap内部又增加了一个链表，用于存放元素的顺序。
LinkedHashMap提供了两种类型的顺序，一种是元素插入时的顺序，一种是最近访问的顺序(构造参数accessOrder决定)。

4) WeakHashMap (弱引用，如果key没有引用，那么该key可以被GC回收)
> 是一种改进的HashMap，它对key实行“弱引用”，如果一个key不再被外部所引用，那么该key可以被GC回收。
弱引用的HashMap，正常的HashMap是强引用，即里面的value不会被GC回收，在WeakHashMap<K,V>中，V中最好是WeakReference类型的，用像这样的代码：m.put(key, new WeakReference(value))。
[WeakHashMap 的实际使用场景有哪些](https://www.zhihu.com/question/46648593?from=profile_question_card)

5) TreeMap (红黑二叉树,key需要实现comparator或构造比较器，实现排序)
> 内部是基于红黑树实现，红黑树是一种平衡查找树，其统计性能优于平衡二叉树。
TreeMap取出来的是排序后的键值对。插入、删除需要维护平衡会牺牲一些效率。但如果要按自然顺序或自定义顺序遍历键，那么TreeMap会更好。

6) EnumMap

7) IdentityHashMap (正常的HashMap中比较是用equals方法，这个用的是“==”比较符)

-------------------

***ConcurrentMap***

1) [concurrent] ConcurrentHashMap (高并发，分段锁)
> get是无锁的，put分段锁(segment)。

2) [concurrent] ConcurrentSkipListMap (log(n)的时间复杂度，有点像多级链表保存的，貌似有点像Redis中的SortedSet的实现)

------------

***SortedMap***

1) TreeMap

2) [concurrent] ConcurrentSkipListMap ...

## 三、堆、栈、堆栈的区别

> 详情参见[Java虚拟机的堆、栈、堆栈如何去理解?](https://www.zhihu.com/question/29833675)

### 3.1 名词介绍

1) 堆(heap)

* JVM里的“堆”特指用于存放Java对象的内存区域。所以根据这个定义，Java对象全部都在堆上。
> 要注意，这个“堆”并不是数据结构意义上的堆（Heap (data structure)，一种有序的树），
而是动态内存分配意义上的堆——用于管理动态生命周期的内存区域。
JVM的堆被同一个JVM实例中的所有Java线程共享。它通常由某种自动内存管理机制所管理，
这种机制通常叫做“垃圾回收”（garbage collection，GC）。JVM规范并不强制要求JVM实现采用哪种GC算法。
传送门[垃圾回收机制中，引用计数法是如何维护所有对象引用的?](https://www.zhihu.com/question/21539353/answer/18596488)
上面这些概念如何映射到实际的JVM的内存，还可以跳另一个传送门[老师说字符串常量和静态变量放在data segment中，问一下这里的data segment和常量池是一回事吗？](https://www.zhihu.com/question/28346079/answer/40472621)

2) 栈(Stack)

* 后入先出(LIFO)，函数调用的数据存活时间满足LIFO，所以栈满足。一个方法每被调用一次，入口出会分配栈空间，出口处会释放栈空间，完全由JVM掌控。
> 以Oracle JDK / OpenJDK的HotSpot VM为例，它使用所谓的“mixed stack”——在同一个调用栈里存放Java方法的栈帧与native方法的栈帧，所以每个Java线程其实只有一个调用栈，融合了JVM规范的JVM栈与native方法栈这俩概念。
详见[为什么函数调用要用栈实现?](https://www.zhihu.com/question/34499262/answer/59415153)

3) 堆栈(就是栈)

* 堆栈是栈。坑...

### 3.2 jvm中的堆和栈对比

![图 duiandzhan](https://psiitoy.github.io/img/blog/jvm/duiandzhan.png)

> 属jvm范畴，另文详解[jvm总结](https://psiitoy.github.io/2016/07/25/[%E6%80%BB%E7%BB%93]jvm%E6%80%BB%E7%BB%93/)

> 关于堆和栈，参见[Java中的堆和栈的区别](https://link.zhihu.com/?target=http%3A//droidyue.com/blog/2014/12/07/differences-between-stack-and-heap-in-java/)

## 四、Collections和Arrays

### 4.1 简要介绍

* Collections和Arrays是***有关集合操作的工具类(负责包装、算法等)***
> 这两个类提供了封装器实现（Wrapper Implementations），“封装器”（Wrapper），它提供了一些方法可以把一个集合转换成一个特殊的集合。
同时Collections类提供了丰富的静态方法比如“折半查找”、“排序”等经典算法，用好可以起来事半功倍。

### 4.2 具体方法

* Collections提供的部分算法
binarySearch：折半查找。
sort：排序，这里是一种类似于快速排序的方法，效率仍然是O(n * log n)，但却是一种稳定的排序方法。
reverse：将线性表进行逆序操作，这个可是从前数据结构的经典考题哦！
rotate：以某个元素为轴心将线性表“旋转”。
swap：交换一个线性表中两个元素的位置。

* Collections提供的封装器
unmodifiableXXX：转换成只读集合，这里XXX代表六种基本集合接口：Collection、List、Map、Set、SortedMap和SortedSet。如果你对只读集合进行插入删除操作，将会抛出UnsupportedOperationException异常。
synchronizedXXX：转换成同步集合。
```
public static <T> Set<T> synchronizedSet(Set<T> s);

```

singleton：创建一个仅有一个元素的集合，这里singleton生成的是单元素Set，
singletonList和singletonMap分别生成单元素的List和Map。
空集：由Collections的静态属性EMPTY_SET、EMPTY_LIST和EMPTY_MAP表示。

## 五、几个对比(部分内容冗余前文)

### 5.1 ArrayList vs LinkedList

1) 底层实现
* 数组 vs 链表

2) 时间复杂度(前面是ArrayList)
* get() O(1) vs O(n) (前者是RandomAccess，后者可以直接查头尾O(1))
* add(E) 不稳定的O(1) vs O(1) (但是前者在到达容量时会面临扩容，所以不是O(1))
* add(index,E) O(n) vs O(n) (前者是元素移动，后者是遍历)
* remove(index) O(n) vs O(1) (前者是元素移动，后者指针操作)

> 性能测试得出，在列表末尾增加元素，ArrayList比LinkedList性能要好，因为数组是连续的，
在末尾增加元素，只有在空间不足时才会进行数组扩容，大部分情况下追加操作效率还是比较高的(System.arraycopy()是native)。

3) 对象与GC
* 前者预留空间会浪费，后者每次都新增元素都要新建一个Node对象，并进行一系列赋值，在频繁系统调用中，对系统性能有一定影响。

4) 共性
* 非线程安全，允许null

### 5.2 ArrayBlockingQueue VS LinkedBlockingQueue

1) 锁 
* ArrayBlockingQueue的put和take共用一把锁，而LinkedBlockingQueue是各以用一把锁。
> 意味着LinkedBlockingQueue可并行存取。(性能差异可不计Doug Lea)

```java
public class ArrayBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {
        /** Main lock guarding all access */
        final ReentrantLock lock;
        /** Condition for waiting takes */
        private final Condition notEmpty;
        /** Condition for waiting puts */
        private final Condition notFull;
}

public class LinkedBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {
    /** Lock held by take, poll, etc */
    private final ReentrantLock takeLock = new ReentrantLock();
    /** Wait queue for waiting takes */
    private final Condition notEmpty = takeLock.newCondition();
    /** Lock held by put, offer, etc */
    private final ReentrantLock putLock = new ReentrantLock();
    /** Wait queue for waiting puts */
    private final Condition notFull = putLock.newCondition();
}

```

2) 对象与GC 
* ArrayBlockingQueue在插入或删除元素时不会产生或销毁任何额外的对象实例，
而LinkedBlockingQueue则会生成一个额外的Node对象。

3) 公平锁
* ArrayBlockingQueue时，我们还可以控制对象的内部锁是否采用公平锁，默认采用非公平锁。

4) 容量
* 数组vs链表，ArrayBlockingQueue构造必须指定容量，LinkedBlockingQueue有空构造默认容量Integer.MAX_VALUE，
后者有内存溢出的风险。