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

2.共享模式：共享，多个线程可同时执行，如Semaphore/CountDownLatch，还有ReentrantLock在自己的线程重复拿锁的时候

## AQS的核心

1.同步队列：如果一个线程获取锁失败，则会被转换成节点放入同步队列中，同步队列的结构采用双向链表

2.条件变量：条件变量可以看作也是一个锁，使用await方法上锁，signal和signalAll方法释放，每个条件变量对应一个条件队列，每个锁可以拥有多个条件变量

2.条件队列：如果一个线程获取到锁，但是无法执行任务(比如某并发容器，在容器为空时进行取操作会导致阻塞)，那么就会进入当前锁的条件队列并且将锁释放，条件队列是建立在锁(条件变量)基础上的，并且只出现在独占模式里，条件队列的结构采用单链表

3.state：state这个东西本质就是信号量，线程获取锁之后state+1。state使用了线程可见+cas的方法保证操作的原子性和唯一性

4.exclusiveOwnerThread：当线程给state+1之后还需要到exclusiveOwnerThread这个报道

# AQS的原理

![aqs](https://github.com/einQimiaozi/awesome_java_notebook/blob/main/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E9%AB%98%E5%B9%B6%E5%8F%91/source/aqs.jpg)

下面是几种aqs获取锁的方式，这些方式来自与node类，node类就是AQS在进入同步队列后被转化成的结点

```java
//这几个方法都是尝试获取锁
public final void acquire(int arg) {}//独占方式
protected boolean tryAcquire(int arg) {}
public final void acquireShared(int arg) {}//共享方式
public final void acquireInterruptibly(int arg){}//独占方式
public final void acquireSharedInterruptibly(int arg){}//共享方式
```

注意，tryAcquire()和它对应的锁释放方法tryRelease()需要自己在继承的子类里实现(tryAcquireShared方法也是，好像try开头的方法都要自己实现)

aqs的acquire等方法加锁的底层是通过Locksupport实现的，也就是unsafe，系统级实现，所以其实你看不到源码

## 具体流程

1.线程a首先调用acquire方法，acquire方法调用tryAcquire方法尝试修改state

2.如果失败则将线程包装成节点丢入同步队列并做自旋尝试获取锁，如果自旋获取不到则进入阻塞等待被唤醒，唤醒的条件是该节点的前驱节点发出信号(就是前驱搞完了，告诉后面你可以开始抢了)

3.如果成功的话，则开始执行线程，并将自己放入exclusiveOwnerThread，state+1

4.如果执行线程时发现线程调用了new出来的条件变量的await方法，则将自己放入该条件变量的条件队列

5.如果其他线程调用了signal或signalAll(这个方法不建议用，因为它会一次性唤醒该条件队列中所有的线程，会造成惊群)，那么该线程被转入同步队列，也就是说，一个线程只能处在条件队列or处在同步队列，二选一，不能影分身

## 独占模式和共享模式的区别

大部分流程都差不多，一个比较主要的区别就是独占模式下state只能是0or1,当state为1的时候其他线程都拿不到锁，而共享模式下state的值可以根据一个上限累加，未达到上限时就可以实现一个锁被多个线程获取，这也是为什么共享模式不需要条件队列，因为你一个线程就算阻塞也不影响其他线程去拿锁，所以阻塞就阻塞呗
