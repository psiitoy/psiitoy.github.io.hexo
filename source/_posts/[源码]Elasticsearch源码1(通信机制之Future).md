---
layout: post
title: "[源码]Elasticsearch源码(1通信机制之Future)"
date: 2017-08-09 20:15:06 
categories: 
    - 源码
tags:
    - es
---


<!--more-->

## 1、 前言

首先分析客户端的RPC调用部分，使用了Future模式。

### 1.1 Future模式

分析代码前先简单介绍下Future模式
From 《Java 高并发程序设计》
　　Future模式，核心思想是异步调用，就是当调用一个方法时，这个函数可能执行得很慢，就需要等待，
但是有时候并不着急要这个结果，所以选择不去傻傻等待，而是做其他的事情。就好比”双十一”购物，你买到了想要的东西，
那么你不可能等待它到货，然后才做另一件事情，你可能想继续购物其他的商品。而对于已经购买得商品，会生成一个订单，
你只需要等待这个订单的快递通知（notify）就行了。

一个最基本的future模式代码片段

```java
class Demo{
     public Future request() {
        final Future future = new Future();
    
        new Thread() {
            public void run() {
                // 下面这个动作可能是耗时的
                RealSubject subject = new RealSubject();    //构建过程可能是远程调用
                future.setRealSubject(subject);
            }
        }.start();
    
        return future;
     } 
}
     
```

### 1.2 线程池的中的Future(FutureTask)

类比的我们看一下线程池的future实现，场景是把一个回调任务交给线程池然后返回给你一个Future，当任务执行完成释放阻塞。
```java
public abstract class AbstractExecutorService implements ExecutorService {
    public <T> Future<T> submit(Callable<T> task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task);
        execute(ftask);
        return ftask;
    }

    protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
        return new FutureTask<T>(callable);
    }
}       

```

线程池的submit过程后续单独讲,继续跟进FutureTask的run，发现最终调用set方法并且遍历通知Waitor。
```java
public class FutureTask<V> implements RunnableFuture<V> {
    public void run() {
        if (state != NEW ||
            !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                         null, Thread.currentThread()))
            return;
        try {
            Callable<V> c = callable;
            if (c != null && state == NEW) {
                V result;
                boolean ran;
                try {
                    result = c.call();
                    ran = true;
                } catch (Throwable ex) {
                    result = null;
                    ran = false;
                    setException(ex);
                }
                if (ran)
                    set(result);
            }
        } finally {
            // runner must be non-null until state is settled to
            // prevent concurrent calls to run()
            runner = null;
            // state must be re-read after nulling runner to prevent
            // leaked interrupts
            int s = state;
            if (s >= INTERRUPTING)
                handlePossibleCancellationInterrupt(s);
        }
    }
    
    protected void set(V v) {
        if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
            outcome = v;
            UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state 最终状态NORMAL
            finishCompletion(); //通知waitor
        }
    }    
    
   protected void setException(Throwable t) {
        if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
            outcome = t;
            UNSAFE.putOrderedInt(this, stateOffset, EXCEPTIONAL); // final state最终状态EXCEPTIONAL
            finishCompletion();
        }
    }   
    
   private void finishCompletion() {
        // assert state > COMPLETING;
        for (WaitNode q; (q = waiters) != null;) {
            if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {
                for (;;) {
                    Thread t = q.thread;
                    if (t != null) {
                        q.thread = null;
                        LockSupport.unpark(t);
                    }
                    WaitNode next = q.next; //遍历等待链表
                    if (next == null)
                        break;
                    q.next = null; // unlink to help gc
                    q = next;
                }
                break;
            }
        }

        done();

        callable = null;        // to reduce footprint
    }    
}

```

FutureTask是一种RunnableFuture，执行的是本地线程调用run()去同步的set(v)，
而我们需要做的是要等待Netty的异步回调。so我们需要的是另外的Future实现。

## 2、 ES中的Future模式

### 2.1 场景

我们需要的场景是(先不讨论ES是怎么设计的，我们自己设计一个Future)
1) client -> get();//阻塞
2) Netty异步接收Response然后cast to V 调用requestId对应的回调responseHandler 
3) 然后再调用responseHandler 的代理listener 
4) listener的实现是一个Future 需要做set(v) then notify get()的过程。(get可能是多处，那么肯定是要共享锁)


看了上面四条，是不是脑子里马上想到以下几个方案可以实现(线程之间通信)：
1) BlockingQueue，然后1对应的take(),3对应的发起put()。(源自2)
2) Condition，然后1对应的循环+条件+await(),3对应的发起signal()。(源自3)
3) 利用共享模式的Aqs，自己基于cas和等待队列实现。
4) 同步代码块
```java
public class A{
    public Object get() {
        synchronized (this) { // 旋锁
            while (!isDone) { // 是否有结果了
                wait(); //没结果是释放锁，让当前线程处于等待状态
            }
        }
    }
    
    private void setDone(Response res) {
        this.res = res;
        isDone = true;
        synchronized (this) { //获取锁，因为前面wait()已经释放了callback的锁了
            notifyAll(); // 唤醒处于等待的线程
        }
    }    
}

```

Es选择的是3。BaseFuture->Sync->Aqs。
当然了我们万能的guava的[AbstractFuture](http://blog.csdn.net/xxcupid/article/details/51912207)同样给出了实现，
jdk的异步非阻塞的[CompletableFuture](http://blog.csdn.net/zero__007/article/details/50571703)也给出了实现。

作者本人也基于CompletableFuture对源生es做了改造。es自己实现的原因见[BaseFuture链接](https://github.com/elastic/elasticsearch/commits/master/core/src/main/java/org/elasticsearch/common/util/concurrent/BaseFuture.java)

### 2.2 源码

回到正题，我们继续看源码，Client向网关节点A发起异步请求，拿到ActionFuture(继承自Future)。代码如下

```java
class AbstractClient {
    @Override
    public final <Request extends ActionRequest, Response extends ActionResponse, RequestBuilder extends ActionRequestBuilder<Request, Response, RequestBuilder>> ActionFuture<Response> execute(Action<Request, Response, RequestBuilder> action, Request request) {
        PlainActionFuture<Response> actionFuture = PlainActionFuture.newFuture();// 初始化future
        execute(action, request, actionFuture);// code1 发起异步请求
        return actionFuture;// code2 返回还没准备好的数据(等待Netty的异步回调)
    }
}

public class TransportActionNodeProxy<Request extends ActionRequest, Response extends ActionResponse> extends AbstractComponent {
    public void execute(final DiscoveryNode node, final Request request, final ActionListener<Response> listener) {
        /**
         * 请求校验(匿名内部类)
         * @see GetRequest#validate(),SearchRequest#validate()
         *
         */
        ActionRequestValidationException validationException = request.validate();
        if (validationException != null) {
            listener.onFailure(validationException);
            return;
        }
        // 通过netty的channel将数据写入
        transportService.sendRequest(node, action.name(), request, transportOptions, new ActionListenerResponseHandler<Response>(listener) {//code3
            @Override
            public Response newInstance() {
                return action.newResponse();
            }
        });
    }
}    
    
```

code1最终会执行code3

code2返回的future在其调用get()之前都是非阻塞的，而code3只是发起了请求并把requestId注册回调函数，
只有handleResponse的时候调用该resquestId的回调才会set(v) 填充数据并释放阻塞。

TransportService(Es通信过程中存请求上下文，以及RPC方法映射，即request对应action的地方)，
的 clientHandlers就是异步回调池(根据requestId拿到回调执行),存放的就是requestId以及对应的
TransportResponseHandler，与code1的request是一一对应的。
而ActionListener就是声明的回调接口，onResponse最终会执行set(v)，而onFailure会执行setException(e),把code2赋值。

```java
public class TransportService extends AbstractLifecycleComponent<TransportService> {
    final ConcurrentMapLong<RequestHolder> clientHandlers = ConcurrentCollections.newConcurrentMapLongWithAggressiveConcurrency();
}

public interface ActionListener<Response> {

    /**
     * A response handler.
     */
    void onResponse(Response response);

    /**
     * A failure handler.
     */
    void onFailure(Throwable e);
}

public abstract class ActionListenerResponseHandler<Response extends TransportResponse> extends BaseTransportResponseHandler<Response> {

    private final ActionListener<Response> listener;

    public ActionListenerResponseHandler(ActionListener<Response> listener) {
        this.listener = listener;
    }

    @Override
    public void handleResponse(Response response) {
        listener.onResponse(response);
    }

    @Override
    public void handleException(TransportException e) {
        listener.onFailure(e);
    }

    @Override
    public String executor() {
        return ThreadPool.Names.SAME;
    }
}

public abstract class BaseTransportResponseHandler<T extends TransportResponse> implements TransportResponseHandler<T> {

}

//code1中PlainActionFuture的父类AdapterActionFuture(适配器)，我们只看两个方法
public abstract class AdapterActionFuture<T, L> extends BaseFuture<T> implements ActionFuture<T>, ActionListener<L> {
    @Override
    public T actionGet() {
        try {
            return get();   //code4
        } catch (InterruptedException e) {
            throw new IllegalStateException("Future got interrupted", e);
        } catch (ExecutionException e) {
            throw rethrowExecutionException(e);
        }
    }
        
    @Override
    public void onResponse(L result) {
        set(convert(result));   //code5
    }
        
    @Override
    public void onFailure(Throwable e) {
        setException(e);    //code6
    }
        
    protected abstract T convert(L listenerResponse);
}

```

我们java客户端调用的场景通常是这样的

```java
public class TestGet{
    public static void main(String[] args) {
        GetResponse sResponse = Tool.CLIENT.prepareGet(indexName, typeName, id).get();
        logger.info(sResponse);
        Tool.CLIENT.close();
    }    
} 

```

继续跟code4,5,6发现其实现是在父类BaseFuture，且具体实现其实是交给了内部类Sync(继承自AQS)

```java
public abstract class BaseFuture<V> implements Future<V> {
    private final Sync<V> sync = new Sync<>();    

    @Override
    public V get() throws InterruptedException, ExecutionException {
        return sync.get();
    }

    protected boolean set(@Nullable V value) {
        boolean result = sync.set(value);
        if (result) {
            done();
        }
        return result;
    }
    
    protected boolean setException(Throwable throwable) {
        boolean result = sync.setException(checkNotNull(throwable));
        if (result) {
            done();
        }

        return result;
    }
    
    static final class Sync<V> extends AbstractQueuedSynchronizer {
        
        V get() throws CancellationException, ExecutionException,
                InterruptedException {

            // Acquire the shared lock allowing interruption.
            acquireSharedInterruptibly(-1);     //code7
            return getValue();  //code8
        }
        
        boolean set(@Nullable V v) {
            return complete(v, null, COMPLETED);    
        }
        
        boolean setException(Throwable t) {
            return complete(null, t, COMPLETED);    
        }
        
        boolean cancel() {
            return complete(null, null, CANCELLED);     
        }
        
        @Override
        protected int tryAcquireShared(int ignored) {   //对应的code11
            if (isDone()) {
                return 1;
            }
            return -1;
        }        
        
        @Override
        protected boolean tryReleaseShared(int finalState) {
            setState(finalState);   //设置状态为完成
            return true;
        }        

        private boolean complete(@Nullable V v, @Nullable Throwable t,
                                 int finalState) {
            boolean doCompletion = compareAndSetState(RUNNING, COMPLETING); //code9  cas防止并发
            if (doCompletion) {
                this.value = v;
                this.exception = t;
                releaseShared(finalState);  //code10    
            } else if (getState() == COMPLETING) {//别人也在操作 置完成
                acquireShared(-1);  //code11  park自己 等待别人完成   
            }
            return doCompletion;
        }
        
       boolean isDone() {
            return (getState() & (COMPLETED | CANCELLED)) != 0;
        }        
        
    }
}

```

AQS park 和 unpark的过程

```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
     /**
     * Acquires in shared interruptible mode.
     * @param arg the acquire argument
     */
    private void doAcquireSharedInterruptibly(int arg)  //对应的code7阻塞
        throws InterruptedException {
        final Node node = addWaiter(Node.SHARED);   //共享锁
        boolean failed = true;
        try {
            for (;;) {  //自旋
                final Node p = node.predecessor();
                if (p == head) {    //如果队首那么尝试一次跳出判定
                    int r = tryAcquireShared(arg);  //code11 跳出条件抽象方法子类实现
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
    
    private void doReleaseShared() {    //释放锁通知所有
        for (;;) {
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;            // loop to recheck cases
                    unparkSuccessor(h);
                }
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            if (h == head)                   // loop if head changed
                break;
        }
    }    

    public final boolean releaseShared(int arg) {   //对应的code10
        if (tryReleaseShared(arg)) {    //抽象方法子类实现 设置状态为完成
            doReleaseShared();      //释放锁
            return true;
        }
        return false;
    }
}

```

那么对应get就是最终会执行code7去阻塞(采用的AQS共享模式)，而异步回调结果的onResponse和onFailure，
就都是对应到了 code9,code10,code11。code9就是设置aqs队列的状态为完成，
code10是针对共享模式的锁释放(比如读锁),code11是发现别的线程正在设置完成中(并发问题)，等待别人设置完成(小概率)。


