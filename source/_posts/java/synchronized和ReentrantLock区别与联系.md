title: synchronized和ReentrantLock区别与联系
date: 2016-05-03 10:09:57
tags: [synchronized,ReentrantLock]
categories: [java]
---
1. synchronized是在JVM层面上实现的，不但可以通过一些监控工具监控synchronized的锁定，而且在代码执行时出现异常，JVM会自动释放锁定，但是使用ReentrantLock则不行，ReentrantLock是通过自己写代码实现加锁和释放的，要保证锁定一定会被释放，就必须将unLock()放到finally{}中
2. synchronized 采用了悲观锁的机制，ReentrantLock采用了乐观锁的机制，通过原子类来实现

3. ReentrantLock能做到synchronized能做到的所有事情，还包括如下
3.1. lock()方法,如果获取了锁立即返回，如果别的线程持有锁，当前线程则一直处于阻塞状态，直到获取锁
3.2. tryLock()方法，如果获取了锁立即返回true，如果别的线程正持有锁，立即返回false
3.3. tryLock(long timeout,TimeUnit unit)，如果获取了锁定立即返回true，如果别的线程正持有锁，会等待参数给定的时间，在等待的过程中，如果获取了锁定，就返回true，如果等待超时，返回false
3.4. lockInterruptibly(),如果获取了锁定立即返回，如果没有获取锁定，当前线程处于阻塞状态，直到或者锁定，或者当前线程被别的线程中断
3.5. ReentrantLock可以使用公平锁，synchronized不行。
  ``ReentrantLock fairLock = new ReentrantLock(true);``
4. 在资源竞争不是很激烈的情况下，Synchronized的性能要优于ReetrantLock，但是在资源竞争很激烈的情况下，Synchronized的性能会下降几十倍，但是ReetrantLock的性能能维持常态；
