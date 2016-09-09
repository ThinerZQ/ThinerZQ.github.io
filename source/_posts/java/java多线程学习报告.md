title: Java 多线程学习
date: 2016-07-09 09:08:06
tags: [多线程]
categories: [java]
---
1. ConcurrentHashMap：线程安全HashMap，分段锁。put(),remove()加锁，size()三次后再加锁

2. CopyOnWriteArrayList：线程安全，读无锁的ArrayList,使用ReentrantLock, 每次add，remove都是新创建一个数组，复制元素到新的数组，最后切换全局数组引用。适用于多读少写
3. CopyOnWriteArraySet：通过CopyOnWriteSet实现，add()的时候会遍历整个数组判断数组是否存在这个值，因此性能略低于CopyOnWriteArrayList

4. ArrayBlockingQueue： 基于数组，ReentrantLock,Condition实现的有界队列。add(E e):队满，抛异常。put(E e):队满，阻塞。offer(E e):队满，返回false。remove()，take()，poll()。和上面对应。

5. LinkedBlockingQueue：基于链表，读写锁分离，遍历同时锁。此按FIFO （先进先出） 排序元素，吞吐量通常要高于ArrayBlockingQueue。

6. SynchronousQueue：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue。

7. PriorityBlockingQueue：一个具有优先级的无限阻塞队列。

8. Atomic：支持原子操作的类，由硬件指令cmpxchg提供CAS支持（简单来说，先使用一个期望值和一个变量的当前值进行比较，如果当前变量的值与期望的值相等，就使用一个新值替换当前变量的值）,比synchronize性能高出2倍。实现类有AtomicBoolean，AtomicInteger，AtomicIntegerArray，AtomicLong，AtomicLongArray，AtomicReference，AtomicReferenceArray等。需要注意的是ABA问题，可以通过AtomicStampedReference解决。

9.  线程池
线程池就是有一堆线程，随时可以取一个线程出来用。学习线程池最好的方法就是从线程池的构造函数开始看。一个线程池的关注点在构造函数中的几个参数。常用的Executor类的几个方法都是返回一个的ThreadPoolExecutor，这些ThreadPoolExecutor的构造函数的参数各不相同。
```java
public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,  BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```
  1. corePoolSize： 线程池维护线程的最少数量：当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的prestartAllCoreThreads方法，线程池会提前创建并启动所有基本线程。

  2. maximumPoolSize：线程池维护线程的最大数量：线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。如果使用了无界的任务队列这个参数就没什么效果。

  3. keepAliveTime： 线程池维护线程所允许的空闲时间：线程池的工作线程空闲后，保持存活的时间。所以如果任务很多，并且每个任务执行的时间比较短，可以调大这个时间，提高线程的利用率。

  4. unit： 线程池维护线程所允许的空闲时间的单位：可选的单位有天（DAYS），小时（HOURS），分钟（MINUTES），毫秒(MILLISECONDS)，微秒(MICROSECONDS, 千分之一毫秒)和毫微秒(NANOSECONDS, 千分之一微秒)。

  5. workQueue： 线程池所使用的缓冲队列。

  6. threadFactory：线程池创建线程的工厂：可以通过线程工厂给每个创建出来的线程设置更有意义的名字。

  7. handler： 线程池对拒绝任务的处理策略：AbortPolicy：直接抛出异常。
  CallerRunsPolicy：只用调用者所在线程来运行任务。
  DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。
  DiscardPolicy：不处理，丢弃掉。
  当然也可以根据应用场景需要来实现RejectedExecutionHandler接口自定义策略。如记录日志或持久化不能处理的任务。

10. Executors：对ThreadPoolExecutor再封装。

  1. nexFixedThreadPool(int) :直接创建固定大小的线程池，keepAlive=0，超过LinkedBlockingQueue的最大范围拒绝

  2. newSingleThreadExecutor:线程只有1个，其他的等待

  3. newCachedThreadPool:corePoolSize=0,keepAlive=60s,SynchronousQueue,最大线程未Integer.Max_Value

  4. newScheduledThreadPool(int):DelayedWorkQueue:会对新加入的任务按照执行时间排序

11. FutureTask：异步获取执行结果，保证只执行一次，可取消任务。
12. Semaphore：控制某资源同时被访问的个数的类，通过构造函数可以达到公平访问和非公平访问，默认非公平。如果把资源那个数设置为1,可以达到锁的效果。
13. CountDownLatch：用于多个线程同时开始某一个动作，每个线程执行的时候，计数器-1，如果计数器为0，位于latch.await()后面代码才会执行。一般用来等待任务执行完毕
14. CyclicBarrier: 和CountDownLatch刚好反过来，它是当await()的数量达到一定，才继续往后执行，另外还能设置一个回调：当await()达到一定数量后，开始执行回调的Runable对象

15. ReentrantLock：ReentrantLock提供了公平锁和非公平锁，lock().trylock(),trylockInterrupted, tryLock(time)等方法

16. Condition： 一般和RenentrantLock配合使用，可在同一个锁上启用多个Condition,用于等待，通知操作。

17. 和ReentrantLock没有任何继承关系	，提供了读写锁分离。读的时候如果没有线程写就直接获取，读操作是无阻赛的。写的时候如果有其他线程在读或者写就会被阻塞。在同一线程中，持有读锁后不能调用写锁的lock方法（会死锁）：读锁不可升级。同一个线程中，持有写锁后，可以调用读锁的lock方法，之后如果调用写锁的unlock(0方法，当前锁降级为读锁。适用于读多写少
