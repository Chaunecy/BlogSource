---
title: Java源码分析 - ReentrantLock
date: 2018-05-08 09:31:56
tags:
	- Java
	- 并发
	- 源码分析
---



## javadoc说明

> A reentrant mutual exclusion {@link Lock} with the same basic behavior and semantics as the implicit monitor lock accessed using {@code synchronized} methods and statements, but with extended capabilities.

可重入锁：一个可重入的互相排斥的锁，和隐式地使用**synchronized**方法获得的锁有着相同的基本行为与语义。

<!-- more -->

## 构造方法

### 非公平锁

默认构造方法，使用非公平锁策略

```java
    public ReentrantLock() {
        sync = new NonfairSync();
    }
```

### 公平锁

重载构造方法，可以指定使用公平锁或者非公平锁

``` java
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

至于**FairSync()**和**NonfairSync()**具体是什么，我们稍后再看。

## 加锁：Lock

### 函数说明

- 如果锁没有另一个线程持有锁，则立即返回，并把锁的持有计数设为1。
- 如果当前线程已经获取到锁，那么锁持有计数加1，函数立即返回。
- 如果锁被另一个线程持有，那么出于线程调度的原因当前线程会disabled，并在获取到锁之前保持休眠；获取到锁后把锁的持有计数设为1。

### 代码解释

``` java
    public void lock() {
        sync.lock();
    }
```

这里用到了**sync**的**lock()**方法，来看看它是什么。

**sync**是一个**Sync**类的实例。

```java
	private final Sync sync;
```

看看它的定义

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
    // something omitted
    abstract void lock();
    // something omitted
}
```

在构造方法里，我们初始化了**sync**字段，如果是公平锁，就使用**FairSync()**，如果是非公平锁，就使用**NonFairSync()**。它们都是**Sync**的子类，分别实现了不同策略的**lock()**方法。

先看**NonFairSync**。

```java
	static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
        final void lock() {
            // 如果没有线程占有锁
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
```

它的加锁策略是先执行**compareAndSetState(0, 1)**，这个方法是在**AbstractQueuedSynchronizer**中定义的，就是**Sync**所继承的类。解释一下这个函数，它使用CAS（Compare and Swap）机制，先把expect与要被修改的值进行比对：如果一致，执行修改，true；不一致，false。

在这里，**compareAndSetState(0, 1)**表示：锁的持有计数为0，我把它设置成1，如果成功，就能获取到锁**setExclusiveOwnerThread(Thread.currentThread())**；如果失败，就执行**acquire(1)**。

``` java
    protected final boolean compareAndSetState(int expect, int update) {
        // See below for intrinsics setup to support this
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }
```

这里又用到了**acquire(1)**，我们看看它是什么。

它也是在**Sync**的父类**AbstractQueuedSynchronizer**中定义的：

``` java
    public final void acquire(int arg) {
        // 获取到锁直接返回，被中断则执行后续操作
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```

它会先调用由子类实现的**tryAcquire**，而在**NonFairSync**中，它是返回了**Sync**类中的**nonfairTryAcquire(acquires)**。实现如下

``` java
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            // 锁的持有计数
            int c = getState();
            // 此时锁被还了回来
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            // 锁被自己持有
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            // 其它情况，锁被占有
            return false;
        }
```

**tryAcquire**失败后会执行**acquireQueued(addWaiter(Node.EXCLUSIVE), arg)**，这里用到了两个函数，我们先看**addWaiter(Node.EXCLUSIVE)**。其中**Node.EXCLUSIVE**表示排他的锁。

``` java
	private Node addWaiter(Node mode) {
    	// 以互斥锁的形式，把当前线程封装进Node
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            // 设置成功，直接返回；
            // 设置失败，执行enq。
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        enq(node);
        return node;
    }

    private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
```

其中Node的定义为（不完全）：

``` java
    static final class Node {
        /** Marker to indicate a node is waiting in shared mode */
        static final Node SHARED = new Node();
        /** Marker to indicate a node is waiting in exclusive mode */
        static final Node EXCLUSIVE = null;

        volatile Node prev;

        volatile Node next;

        volatile Thread thread;

        Node nextWaiter;

        final boolean isShared() {
            return nextWaiter == SHARED;
        }

        final Node predecessor() throws NullPointerException {
            Node p = prev;
            if (p == null)
                throw new NullPointerException();
            else
                return p;
        }

        Node() {    // Used to establish initial head or SHARED marker
        }

        Node(Thread thread, Node mode) {     // Used by addWaiter
            this.nextWaiter = mode;
            this.thread = thread;
        }

        Node(Thread thread, int waitStatus) { // Used by Condition
            this.waitStatus = waitStatus;
            this.thread = thread;
        }
    }
```

再看**acquireQueued**函数，

``` java
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            // 如果在等待过程中被中断了，就返回true，
            // 以执行selfInterrupt()函数，见acquire(1);
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                // 当前node的先代是head，而且当前正在运行的线程获取到了锁
                // node就变成了head。
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                // 阻塞线程，等待唤醒或中断
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed) //出于某种原因，获取失败
                cancelAcquire(node);
        }
    }

	// head不保存thread、prev
    private void setHead(Node node) {
        head = node;
        node.thread = null;
        node.prev = null;
    }

```

在进一步的分析之前先再看一下Node的其他字段

``` java
        /** waitStatus value to indicate thread has cancelled */
        static final int CANCELLED =  1; // 线程被取消了
        /** waitStatus value to indicate successor's thread needs unparking */
        static final int SIGNAL    = -1; // 唤醒后继节点的线程
        /** waitStatus value to indicate thread is waiting on condition */
        static final int CONDITION = -2;
        /**
         * waitStatus value to indicate the next acquireShared should
         * unconditionally propagate
         */
        static final int PROPAGATE = -3;

        volatile int waitStatus;

```

**shouldParkAfterFailedAcquire**

``` java
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        // 前继节点为SIGNAL，可以安全地park后继节点
        if (ws == Node.SIGNAL)
            return true;
        // 前继节点为CANCELLED，所有这些状态的节点去掉
        if (ws > 0) {
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
            // 其他情况，将前继节点状态设置为SIGNAL。
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
```

还是AQS中的函数**parkAndCheckInterrupt()**

``` java
    private final boolean parkAndCheckInterrupt() {
        // 当前线程休眠，除非被唤醒、中断，或是这个调用无缘由地return
        LockSupport.park(this);
        return Thread.interrupted();
    }
	// 中断状态会被清除，就是说，第一次调用为true，接着再调用一次就会是false
    public static boolean interrupted() {
        return currentThread().isInterrupted(true);
    }
```

到这里这个acquire函数就基本讲完了。

``` java
    public final void acquire(int arg) {
        
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```

剩下的还有**cancelAcquire(node)**，和**selfInterrupt()**。

**selfInterrupt()**的实现倒也简单

``` java
    static void selfInterrupt() {
        Thread.currentThread().interrupt();
    }
```

**cancelAcquire(node)**就比较复杂了

``` java
    private void cancelAcquire(Node node) {
        // Ignore if node doesn't exist
        if (node == null)
            return;

        node.thread = null;

        // Skip cancelled predecessors
        Node pred = node.prev;
        while (pred.waitStatus > 0)
            node.prev = pred = pred.prev;

        // predNext is the apparent node to unsplice. CASes below will
        // fail if not, in which case, we lost race vs another cancel
        // or signal, so no further action is necessary.
        Node predNext = pred.next;

        // Can use unconditional write instead of CAS here.
        // After this atomic step, other Nodes can skip past us.
        // Before, we are free of interference from other threads.
        node.waitStatus = Node.CANCELLED;

        // If we are the tail, remove ourselves.
        if (node == tail && compareAndSetTail(node, pred)) {
            compareAndSetNext(pred, predNext, null);
        } else {
            // If successor needs signal, try to set pred's next-link
            // so it will get one. Otherwise wake it up to propagate.
            int ws;
            if (pred != head &&
                ((ws = pred.waitStatus) == Node.SIGNAL ||
                 (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
                pred.thread != null) {
                Node next = node.next;
                if (next != null && next.waitStatus <= 0)
                    compareAndSetNext(pred, predNext, next);
            } else {
                unparkSuccessor(node);
            }

            node.next = node; // help GC
        }
    }
```

好了，以上就是加锁涉及到的大部分代码。

### 小结

回顾一下加锁的过程，其实就是lock()函数的说明

- 如果锁没有另一个线程持有锁，则立即返回，并把锁的持有计数设为1。
- 如果当前线程已经获取到锁，那么锁持有计数加1，函数立即返回。
- 如果锁被另一个线程持有，那么出于线程调度的原因当前线程会disabled，并在获取到锁之前保持休眠；获取到锁后把锁的持有计数设为1。

## 解锁：unlock

### 函数说明

- 如果当前线程是锁的持有者，那么锁的持有计数减去给定数值
- 如果持有计数变为0，锁会被释放
- 如果当前线程不是锁的持有者，会抛出异常

### 代码解释

``` java
    public void unlock() {
        sync.release(1);
    }
```





