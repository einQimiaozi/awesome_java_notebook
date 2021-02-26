## AQS是什么

1.AQS全称：AbstractQueuedSynchronizer，抽象队列同步器

2.本质就是juc包下lock之类的同步锁、同步器的一个实现框架

3.内部实现的关键点：先进先出队列，state状态(锁的状态)，exclusiveOwnerThread：AQS继承了AbstractOwnableSynchronizer，而其中有个属性exclusiveOwnerThread，用来记录当前独占锁的线程是谁；

state的定义
```java
private volatile int state;
```

state的修改方法使用cas
```java
protected final boolean compareAndSetState(int expect, int update) {
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }
```

等待队列(也叫CHL队列，同步队列)的头尾节点的定义，不需要序列化(aqs应该是实现了序列化接口的)

```java
private transient volatile Node head;

private transient volatile Node tail;
```

CHL队列由链表实现，以自旋的方式获取资源，是可阻塞的先进先出的双向队列。通过自旋和CAS操作保证节点插入和移除的原子性。当有线程获取锁失败，就被添加到队列末尾。

4.一般称AQS为同步器

## AQS的两种模式

1.独占模式：该模式只有一个线程能执行，如ReentrantLock

2.共享模式：共享，多个线程可同时执行，如Semaphore/CountDownLatch

## AQS的核心

1.同步队列：如果一个线程获取锁失败，则会被转换成节点放入同步队列中

2.条件队列：如果一个线程获取到锁，但是无法执行任务(比如某并发容器，在容器为空时进行取操作会导致阻塞)，那么就会进入当前锁的条件队列并且将锁释放，条件队列是建立在锁基础上的，并且指出现在独占模式里

3.state：state这个东西本质就是信号量，线程获取锁之后state+1。state使用了线程可见+cas的方法保证操作的原子性和唯一性

4.exclusiveOwnerThread：当线程给state+1之后还需要到exclusiveOwnerThread这个报道

# AQS的原理

![aqs](https://github.com/einQimiaozi/awesome_java_notebook/blob/main/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E9%AB%98%E5%B9%B6%E5%8F%91/source/aqs.jpg)

1.线程a首先尝试获取锁

2.如果成功则将state+1,并且将自己设为当前获取锁的线程

3.如果调用了await方法，则将自己放入条件队列

4.如果获取锁失败则放入工作队列，然后不断自旋获取锁,得不到锁就会阻塞等待被唤醒
