---
layout: post
title: "[总结]jvm总结"
date: 2016-07-25 21:15:06 
categories: 
    - 总结
tags:
    - jvm
---

jvm总结，涉及到JVM结构，垃圾回收，类加载机制等。

<!--more-->

## 一、JVM简介

### 1.1 结构概括

* 下面分别给出中-英文的结构图

![图 jvm-jiegou](https://psiitoy.github.io/img/blog/jvm/jvm-jiegou.png)

![图 jvmjiegou](https://psiitoy.github.io/img/blog/jvm/jvmjiegou.png)

* 如上图所示，首先Java源代码文件(.java后缀)会被Java编译器编译为字节码文件(.class后缀)，
然后由JVM中的类加载器加载各个类的字节码文件，加载完毕之后，交由JVM执行引擎执行。在整个程序执行过程中，
JVM会用一段空间来存储程序执行期间需要用到的数据和相关信息，这段空间一般被称作为Runtime Data Area（运行时数据区），
也就是我们常说的JVM内存。因此，在Java中我们常常说到的内存管理就是针对这段空间进行管理（如何分配和回收内存空间）。

> 在JVM规范中虽然规定了程序在执行期间运行时数据区应该包括图中这几部分，但是至于具体如何实现并没有做出规定，不同的虚拟机厂商可以有不同的实现方式。

### 1.2 工作过程

* 首先我们熟悉一下一个一般性的 Java 程序的工作过程。

> 一个 Java 源程序文件，会被编译为字节码文件（以 class 为扩展名），每个java程序都需要运行在自己的JVM上，
然后告知 JVM 程序的运行入口，再被 JVM 通过字节码解释器加载运行。那么程序开始运行后，
都是如何涉及到各内存区域的呢？

*　概括地说来，JVM初始运行的时候都会分配好Method Area（方法区）和Heap（堆），而JVM 每遇到一个线程，
就为其分配一个Program Counter Register（程序计数器）, VM Stack（虚拟机栈）和Native Method Stack 
（本地方法栈），当线程终止时，三者（虚拟机栈，本地方法栈和程序计数器）所占用的内存空间也会被释放掉。
这也是为什么我把内存区域分为线程共享和非线程共享的原因，非线程共享的那三个区域的生命周期与所属线程相同，
而线程共享的区域与JAVA程序运行的生命周期相同，所以这也是系统垃圾回收的场所只发生在线程共享的区域
（实际上对大部分虚拟机来说知发生在Heap上）的原因。

```java
public class JVMShowcase {  
//静态类常量,  
public final static String ClASS_CONST = "I'm a Const";  
//私有实例变量  
private int instanceVar=15;  
public static void main(String[] args) {  
//调用静态方法  
runStaticMethod();  
//调用非静态方法  
JVMShowcase showcase=new JVMShowcase();  
showcase.runNonStaticMethod(100);  
}  
//常规静态方法  
public static String runStaticMethod(){  
return ClASS_CONST;  
}  
//非静态方法  
public int runNonStaticMethod(int parameter){  
int methodVar=this.instanceVar * parameter;  
return methodVar;  
}  
}

```

- 例子 下面举个典型的例子来进一步说明(以上类作为例子说明JVM运行过程)
 + 第 1 步 、***向操作系统申请空闲内存***。内存写上“Java 占用”标签
 + 第 2 步，***分配内存内存***。JVM 分配内存。JVM 获得到 64M 内存，就开始得瑟了，首先给 heap 分个内存，然后给栈内存也分配好。
 + 第 3 步，***文件检查和分析class 文件***。若发现有错误即返回错误。
 + 第 4 步，***加载类***。由于没有指定加载器，JVM 默认使用 bootstrap 加载器，就把 rt.jar 下的所有类都加载到了堆类存的Method Area，JVMShow 也被加载到内存中。我们来看看Method Area区域，如下图：（这时候包含了 main 方法和 runStaticMethod方法的符号引用，因为它们都是静态方法，在类加载的时候就会加载）
 
![图 jvm-use1](https://psiitoy.github.io/img/blog/jvm/jvm-use1.png)
 > Heap 是空，Stack 是空，因为还没有对象的新建和线程被执行。
 
 + 第 5 步、执行方法。执行 main 方法。执行启动一个线程，开始执行 main 方法，在 main 执行完毕前，方法区如下图所示：
 （public final static String ClASS_CONST = "I'm a Const";  ）
 
![图 jvm-use2](https://psiitoy.github.io/img/blog/jvm/jvm-use2.png)
> 在 Method Area 加入了 CLASS_CONST 常量，它是在第一次被访问时产生的（runStaticMethod方法内部）。

> 堆内存中有两个对象 object 和 showcase 对象，如下图所示：（执行了JVMShowcase showcase=new JVMShowcase();  ）

![图 jvm-use3](https://psiitoy.github.io/img/blog/jvm/jvm-use3.png)
> 为什么会有 Object 对象呢？是因为它是 JVMShowcase 的父类，***JVM 是先初始化父类***，然后再初始化子类，甭管有多少个父类都初始化。

> 在栈内存中有三个栈帧，如下图所示：

![图 jvm-use4](https://psiitoy.github.io/img/blog/jvm/jvm-use4.png)
>于此同时，还创建了一个程序计数器指向下一条要执行的语句。

 + 第 6 步，释放内存。释放内存。运行结束，JVM 向操作系统发送消息，说“内存用完了，我还给你”，运行结束。

> 参考[深入理解JVM之JVM内存区域与内存分配](http://www.cnblogs.com/hellocsl/p/3969768.html?utm_source=tuicool&utm_medium=referral)
> 传送门[Java 内存分配全面浅析](http://blog.csdn.net/shimiso/article/details/8595564)
 

## 二、JVM区域划分

### 2.1 五个区域

1) 程序计数器（Program Counter Register）
* 几乎不占内存，不发生OOM
* 作用可以看做是当前线程执行的字节码的位置指示器。
* 分支、循环、跳转、异常处理和线程恢复等基础功能都需要依赖这个计算器来完成

2) 虚拟机栈（VM Stacks）
* 每个线程包含一个栈区，栈中只保存基础数据类型的对象和自定义对象的引用(不是对象)，对象都存放在堆区中
* 类的方法是该类的所有对象共享的，只有一套，对象使用方法的时候方法才被压入栈，方法不使用则不占用内存。
* 栈分为3个部分：基本类型变量区、执行环境上下文、操作指令区(存放操作指令)。
* 由编译器自动分配释放，存放函数的参数值，局部变量的值等。
* 只要线程一结束，该栈就 Over，所以不存在垃圾回收。

> 每个线程执行每个方法的时候都会在栈中申请一个栈帧，每个栈帧包括局部变量区和操作数栈，用于存放此次方法调用过程中的临时变量、参数和中间结果

3) 本地方法栈（Native Method Stacks）
* 用于支持native方法的执行，存储每个native方法调用的状态
> 本地方法栈与Java栈的作用和原理非常相似。区别只不过是Java栈是为执行Java方法服务的，而本地方法栈则是为执行本地方法（Native Method）服务的。
JVM规范中，并没有对本地方发展的具体实现方法以及数据结构作强制规定，虚拟机可以自由实现它。在HotSopt虚拟机中直接就把本地方法栈和Java栈合二为一。

4) 堆（Java Heap）
- 存储的全部是对象，每个对象都包含一个与之对应的class的信息。(class的目的是得到操作指令)
- 堆中不存放基本类型和对象引用，只存放对象本身。
- 只是保存对象实例的属性值，属性的类型和对象本身的类型标记等，并不保存对象的方法（以帧栈的形式保存在Stack中）。
- 程序员基本不用去关心空间释放的问题，Java的垃圾回收机制会自动进行处理。
- 如果从内存回收的角度看，由于现在收集器基本都是采用的分代收集算法。(新生代，老年代)
 + new对象 优先从新生代分配内存,Eden空间不足，会把存活对象移植到Survivor中。
 + 再细致一点的有Eden 空间、From Survivor 空间、To Survivor ，Old。
 + 如果从内存分配的角度看，线程共享的Java 堆中可能划分出多个线程私有的分配缓冲区（Thread Local Allocation Buffer，TLAB）。
 + 无论哪个区域，存储的都仍然是对象实例
 
5) 方法区（Method Area）
- Object Class Data(加载类的类定义数据) 是存储在方法区的。除此之外，常量、静态变量、JIT(即时编译器)编译后的代码也都在方法区。
- 全局变量和静态变量的存储是放在一块的，初始化的全局变量和静态变量在一块区域， 未初始化的全局变量和未初始化的静态变量在相邻的另一块区域。
- jdk6及其以前的版本中，方法区可以理解为永久区(Perm).
 + 永久区可以使用参数-XX:PermSize和-XX:MaxPermSize制定。默认情况下16-64MB
 + 如果你项目中使用代理模式或者CGLIB的话可能在运行的时候生成大量的类，如果这样，需要设置一下永久区的大小，防止永久区内存溢出。
- jdk8以后永久区被移除了(jdk7就逐步开始移除工作)，取而代之的是元数据区
 + 元数据区可以使用-XX:MaxMetaspaceSize制定
 + 元数据区使用的堆外直接内存
 + 元数据区需要指定大小，元数据区发生溢出，虚拟机一样抛出异常，否则耗尽所有可用的系统内存。

> 传送门[常量池的一道题](http://www.cnblogs.com/DreamSea/archive/2011/11/20/2256396.html)

### 2.2 堆外内存

* 堆外内存，也叫直接内存（Direct Memory），并不是虚拟机运行时数据区的一部分，也不是Java
虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用，而且也可能导致
OutOfMemoryError 异常出现，所以我们放到这里一起讲解。
> Netty的消息透传实现了zero-copy，使用的就是这块区域。因为透传不需要完整解析消息，只需要知道消息要转发给下游哪个系统就足够了。所以透传时，我们可以只解析出部分消息，消息整体还原封不动地放在Direct Buffer里，最后直接将它写入到连接下游系统的Channel中。所以应用层的Zero Copy实现就分为两部分：Direct Buffer配置和Buffer的零拷贝传递。

* OOM
> 如果使用，容易被忽视，超出系统或者操作系统级别的限制(JVM内存+堆外内存)，一样会`OutOfMemoryError`。

## 三、JVM深入理解

### 3.1 哪些区域是共享的，哪些是私有的？

* 堆和方法区是所有线程共享的，栈、本地方法栈、程序计数器是线程私有的(随线程生命周期销毁)。

### 3.2 方法区存的什么，是否会回收？

1) 存的什么 (class+ static变量 + 常量池)
* 在方法区中，存储了每个类的信息（包括类的名称、方法信息、字段信息）、静态变量、常量以及编译器编译后的代码等。
在Class文件中除了类的字段、方法、接口等描述信息外，还有一项信息是常量池，用来存储编译期间生成的字面量和符号引用(在类和接口被加载到JVM后，对应的运行时常量池就被创建出来)。

2) 常量池中的内容不变吗
- `在运行期间`也可将新的常量放入运行时常量池中，比如`String的intern方法`。

3) 是否会GC (会，条件苛刻)
- 满足三个条件会GC
+ 1.该类的所有实例都已经被回收
+ 2.加载该类的ClassLoader已经被回收
+ 3.该类的Class对象没有在任何地方被引用（包括Class.forName反射访问）
> Java 虚拟机规范不要求这个区域实现GC

### 3.3 内存溢出的场景

1) 堆溢出
* 创建过多对象
> 下面的程中我们限制Java 堆的大小为20MB，不可扩展（将堆的最小值-Xms 参数与最大值-Xmx 参数设置为一样即可避免堆自动扩展），通过参数-XX:+HeapDump
`OnOutOfMemoryError` 可以让虚拟机在出现内存溢出异常时Dump 出当前的内存堆转储快照以便事后进行分析。

```java
public class HeapOutOfMemory {

    public static void main(String[] args) {
       List<TestCase> cases = new ArrayList<TestCase>();
       while(true){
           cases.add(new TestCase());
       }
    }
}

```

2) 栈溢出
* 递归层数过深
> 当递归深度过大时, 就会耗尽栈空间, 进而导致了 `StackOverflowError` 异常.

```java
public class StackOomTest {

    public static void main(String[] args) {
        digui();
    }


    /**
     * java.lang.StackOverflowError
     */
    public static void digui() {
        digui();
    }

}

```

3) 方法区
> 注意, 因为 JDK7以后 已经移除了永久代, JDK8取而代之的是 metaspace, 因此在 JDK8 中, 下面两个例子都不会导致 `java.lang.OutOfMemoryError: PermGen space` 异常.
* jdk7以前可以调用`String的intern方法`模拟内存溢出
* 代理模式或者CGLIB的话可能在运行的时候生成大量的类可能造成内存溢出。

```java
public class PermGenOomTest {

    /**
     * 方式一:运行时常量池溢出
     * 在 Java 1.6 以及之前的 HotSpot JVM 版本时, 有永久代的概念, 即 GC 的分代收集机制是扩展至方法区的. 在方法区中, 有一部分内存是用于存储常量池, 因此如果代码中常量过多时, 就会耗尽常量池内存, 进而导致内存溢出.那么如何添加大量的常量到常量池呢? 这时就需要依靠 String.intern() 方法了. String.intern() 方法的作用是: 若此 String 的值在常量池中已存在, 则这个方法返回常量池中对应字符串的引用; 反之将此 String 所包含的值添加到常量池中, 并返回此 String 对象的引用. 在 JDK 1.6 以及之前的版本中, 常量池分配在永久代中, 因此我们可以通过设置参数 “-XX:PermSize” 和 “-XX:MaxPermSize” 来间接限制常量池的大小.
     * 注意, 上面所说的 String.intern() 方法和常量池的内存分布仅仅针对于 JDK 1.6 及之前的版本, 在 JDK 1.7 或以上的版本中, 由于去除了永久代的概念, 因此内存布局稍有不同.
     */
    public static void stringIntern(){
        List<String> list = new ArrayList<String>();
        int i = 0;
        while (true) {
            list.add(String.valueOf(i++).intern());
        }
    }

    /**
     *  方式二：调小永久带内存java -XX:PermSize=1M -XX:MaxPermSize=1M
     *  JAVA 1.8同样无效
     */

    /**
     * 方式三：创建大量的Class相关信息
     * 方法区作用是存放 Class 的相关信息, 例如类名, 类访问修饰符, 字段描述, 方法描述等. 因此如果方法区过小, 而加载的类过多, 就会造成方法区的内存溢出.
     *
     */
    /**
     *
     * 在 方法区的内存溢出 内存溢出一节中, 我们提到, JDK8 没有了永久代的概念, 因此那两个例子在 JDK8 下没有实现预期的效果. 那么在 JDK8 下, 是否有类似方法区内存溢出之类的错误呢? 当然有的. 在 JDK8 中, 使用了 MetaSpace 的区域来存放 Class 的相关信息, 因此当 MetaSpace 内存空间不足时, 会抛出 java.lang.OutOfMemoryError: Metaspace 异常.
     * Exception in thread "main" java.lang.OutOfMemoryError: Metaspace
     *
     * //jdk 1.8 VM Args: -XX:MaxMetaspaceSize=10M
     * @param args
     */

    public static void main(String[] args) {
        while (true) {
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(PermGenOomTest.class);
            enhancer.setUseCache(false);
            enhancer.setCallback(new MethodInterceptor() {
                public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                    return methodProxy.invokeSuper(o, objects);
                }
            });

            enhancer.create();
        }
    }



}

```

### 3.4 内存逃逸
* 逃逸分析(Escape Analysis)是目前Java虚拟机中比较前沿的优化技术。逃逸分析的基本行为就是分析对象动态作用域：当一个对象在方法中被定义后，它可能被外部方法所引用，例如作为调用参数传递到其他地方中，称为方法逃逸。
> 传送门[JVM内存逃逸](http://www.jianshu.com/p/7418a4451974)

### 3.5 访问堆上的对象有几种方式？

1) 指针直接访问
* 栈上的引用保存的就是指向堆上对象的指针，一次就可以定位对象，访问速度比较快。
  但是当对象在堆中被移动时（垃圾回收时会经常移动各个对象），栈上的指针变量的值
  也需要改变。目前JVM HotSpot采用的是这种方式。

![图 jvm-d1](https://psiitoy.github.io/img/blog/jvm/jvm-d1.png)

2) 句柄间接访问
* 栈上的引用指向的是句柄池中的一个句柄，通过这个句柄中的值再访问对象。因此句柄
  就像二级指针，需要两次定位才能访问到对象，速度比直接指针定位要慢一些，但是当
  对象在堆中的位置移动时，不需要改变栈上引用的值。
  
![图 jvm-d2](https://psiitoy.github.io/img/blog/jvm/jvm-d2.png)

## 四、JVM垃圾回收机制

> 推荐，传送门[相对全面的GC总结](http://blog.csdn.net/iter_zc/article/details/41746265)

### 4.1 范围：要回收哪些区域？

* Java方法栈、本地方法栈以及PC计数器随方法或线程的结束而自然被回收，
所以这些区域不需要考虑回收问题。Java堆和方法区是GC回收的重点区域

### 4.2 如何判断对象已死

1) 引用计数法

* 引用计数法就是通过一个计数器记录该对象被引用的次数，方法简单高效，
但是解决不了循环引用的问题。比如对象A包含指向对象B的引用，对象B
也包含指向对象A的引用，但没有引用指向A和B，这时当前回收如果采用的
是引用计数法，那么对象A和B的被引用次数都为1，都不会被回收。

> 下面是循环引用的例子，在Hotspot JVM下可以被正常回收，可以证实JVM
采用的不是简单的引用计数法。通过-XX:+PrintGCDetails输出GC日志。

```java
public class ReferenceCount {

	final static int MB = 1024 * 1024;
	
	byte[] size = new byte[2 * MB];
	
	Object ref;
	
	public static void main(String[] args) {
		ReferenceCount objA = new ReferenceCount();
		ReferenceCount objB = new ReferenceCount();
		objA.ref = objB;
		objB.ref = objA;
		
		objA = null;
		objB = null;
		
		System.gc();
		System.gc();
	}

}

```
> [Full GC (System) [Tenured: 2048K->366K(10944K), 0.0046272 secs] 4604K->366K(15872K), [Perm : 154K->154K(12288K)], 0.0046751 secs] [Times: user=0.02 sys=0.00, real=0.00 secs]
 
2) 根搜索

* 通过选取一些根对象作为起始点，开始向下搜索，如果一个对象到根对象
  不可达时，则说明此对象已经没有被引用，是可以被回收的。可以作为根的
  对象有：栈中变量引用的对象，类静态属性引用的对象，常量引用的对象等。
  因为每个线程都有一个栈，所以我们需要选取多个根对象。
  
![图 jvm-gc-root](https://psiitoy.github.io/img/blog/jvm/jvm-gc-root.png)

### 4.3 何时开始GC？

Minor GC（新生代回收）的触发条件比较简单，Eden空间不足就开始进行Minor GC(Eden区+有对象的Survivor(S0)区进行垃圾回收to S1区,此时Eden区被清空，S0也为空。)
回收新生代。而Full GC（老年代回收，一般伴随一次Minor GC）则有几种触发条件：

（1）老年代空间不足

（2）PermSpace空间不足

（3）统计得到的Minor GC晋升到老年代的平均大小大于老年代的剩余空间

这里注意一点：PermSpace并不等同于方法区，只不过是Hotspot JVM用PermSpace来
实现方法区而已，有些虚拟机没有PermSpace而用其他机制来实现方法区。

### 4.4 对象的空间分配和晋升

（1）对象优先在Eden上分配

（2）大对象直接进入老年代

虚拟机提供了-XX:PretenureSizeThreshold参数，大于这个参数值的对象将直接分配到
老年代中。因为新生代采用的是标记-复制策略，在Eden中分配大对象将会导致Eden区
和两个Survivor区之间大量的内存拷贝。

（3）长期存活的对象将进入老年代

对象在Survivor区中每熬过一次Minor GC，年龄就增加1岁，当它的年龄增加到一定程度
（默认为15岁）时，就会晋升到老年代中。

### 4.5 内存泄漏与系统崩溃

1) 系统崩溃前的一些现象(举例说明)：
- `每次垃圾回收的时间越来越长`，由之前的10ms延长到50ms左右，FullGC的时间也有之前的0.5s延长到4、5s
- `FullGC的次数越来越多`，最频繁时隔不到1分钟就进行一次FullGC
- `年老代的内存越来越大`并且每次FullGC后年老代没有内存被释放
- 之后系统会`无法响应新的请求`，逐渐到达OutOfMemoryError的临界值。

2) 回归问题

#### Q：为什么崩溃前垃圾回收的时间越来越长？
A:根据内存模型和垃圾回收算法，垃圾回收分两部分：内存标记、清除（复制），标记部分只要内存大小固定时间是不变的，变的是复制部分，因为每次垃圾回收都有一些回收不掉的内存，所以增加了复制量，导致时间延长。所以，垃圾回收的时间也可以作为判断内存泄漏的依据

#### Q：为什么Full GC的次数越来越多？
A：因此内存的积累，逐渐耗尽了年老代的内存，导致新对象分配没有更多的空间，从而导致频繁的垃圾回收

#### Q:为什么年老代占用的内存越来越大？
A:因为年轻代的内存无法被回收，越来越多地被Copy到年老代

### 4.6 CMS VS G1

> 传送门[CMS收集器和G1收集器优缺点](http://blog.csdn.net/qq_25396633/article/details/72972008)

## 五、JVM参数调优

## 5.1 有关年轻代的JVM参数

1)-XX:NewSize和-XX:MaxNewSize

用于设置年轻代的大小，建议设为整个堆大小的1/3或者1/4,两个值设为一样大。

2)-XX:SurvivorRatio

用于设置Eden和其中一个Survivor的比值，这个值也比较重要。

3)-XX:+PrintTenuringDistribution

这个参数用于显示每次Minor GC时Survivor区中各个年龄段的对象的大小。

4).-XX:InitialTenuringThreshol和-XX:MaxTenuringThreshold

用于设置晋升到老年代的对象年龄的最小值和最大值，每个对象在坚持过一次Minor GC之后，年龄就加1。

在JVM启动参数中，可以设置跟内存、垃圾回收相关的一些参数设置，默认情况不做任何设置JVM会工作的很好，但对一些配置很好的Server和具体的应用必须仔细调优才能获得最佳性能。通过设置我们希望达到一些目标：
GC的时间足够的小
GC的次数足够的少
发生Full GC的周期足够的长
  前两个目前是相悖的，要想GC时间小必须要一个更小的堆，要保证GC次数足够少，必须保证一个更大的堆，我们只能取其平衡。
   （1）针对JVM堆的设置一般，可以通过-Xms -Xmx限定其最小、最大值，为了防止垃圾收集器在最小、最大之间收缩堆而产生额外的时间，我们通常把最大、最小设置为相同的值
   （2）年轻代和年老代将根据默认的比例（1：2）分配堆内存，可以通过调整二者之间的比率NewRadio来调整二者之间的大小，也可以针对回收代，比如年轻代，通过 -XX:newSize -XX:MaxNewSize来设置其绝对大小。同样，为了防止年轻代的堆收缩，我们通常会把-XX:newSize -XX:MaxNewSize设置为同样大小
   （3）年轻代和年老代设置多大才算合理？这个我问题毫无疑问是没有答案的，否则也就不会有调优。我们观察一下二者大小变化有哪些影响
更大的年轻代必然导致更小的年老代，大的年轻代会延长普通GC的周期，但会增加每次GC的时间；小的年老代会导致更频繁的Full GC
更小的年轻代必然导致更大年老代，小的年轻代会导致普通GC很频繁，但每次的GC时间会更短；大的年老代会减少Full GC的频率
如何选择应该依赖应用程序对象生命周期的分布情况：如果应用存在大量的临时对象，应该选择更大的年轻代；如果存在相对较多的持久对象，年老代应该适当增大。但很多应用都没有这样明显的特性，在抉择时应该根据以下两点：（A）本着Full GC尽量少的原则，让年老代尽量缓存常用对象，JVM的默认比例1：2也是这个道理 （B）通过观察应用一段时间，看其他在峰值时年老代会占多少内存，在不影响Full GC的前提下，根据实际情况加大年轻代，比如可以把比例控制在1：1。但应该给年老代至少预留1/3的增长空间
  （4）在配置较好的机器上（比如多核、大内存），可以为年老代选择并行收集算法： -XX:+UseParallelOldGC ，默认为Serial收集
  （5）线程堆栈的设置：每个线程默认会开启1M的堆栈，用于存放栈帧、调用参数、局部变量等，对大多数应用而言这个默认值太了，一般256K就足用。理论上，在内存不变的情况下，减少每个线程的堆栈，可以产生更多的线程，但这实际上还受限于操作系统。
  （4）可以通过下面的参数打Heap Dump信息
-XX:HeapDumpPath
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-Xloggc:/usr/aaa/dump/heap_trace.txt
    通过下面参数可以控制OutOfMemoryError时打印堆的信息
-XX:+HeapDumpOnOutOfMemoryError
 请看一下一个时间的Java参数配置：（服务器：Linux 64Bit，8Core×16G）
 
 JAVA_OPTS="$JAVA_OPTS -server -Xms3G -Xmx3G -Xss256k -XX:PermSize=128m -XX:MaxPermSize=128m -XX:+UseParallelOldGC -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/usr/aaa/dump -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:/usr/aaa/dump/heap_trace.txt -XX:NewSize=1G -XX:MaxNewSize=1G"
经过观察该配置非常稳定，每次普通GC的时间在10ms左右，Full GC基本不发生，或隔很长很长的时间才发生一次
通过分析dump文件可以发现，每个1小时都会发生一次Full GC，经过多方求证，只要在JVM中开启了JMX服务，JMX将会1小时执行一次Full GC以清除引用，关于这点请参考附件文档。

//todo

## 六、JVM类加载机制

> 传送门[JVM类加载过程](http://blog.csdn.net/zhangliangzi/article/details/51319033),[类加载器与双亲委派模型](http://blog.csdn.net/zhangliangzi/article/details/51338291)