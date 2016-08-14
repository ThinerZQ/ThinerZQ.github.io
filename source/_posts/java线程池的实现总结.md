title: java线程池的实现总结
date: 2016-05-03 16:19:38
tags:
 线程池
categories:
 java
---
# Thread 和Runnable 的区别
学了这么久的多线程，始终有一点迷惑
>往线程池里面提交的时候，提交的是一个实现了Runnable接口的对象，本来提交的时候就是一个Runnable,那么怎么实现线程复用。

产生这个迷惑的原因是没有弄明白Thread 和Runnable的区别。

java中真正的线程执行类是Thread  

Runnable（可运行的）只是一个接口，中间定义了run()方法，实现了这个接口的类，实现了run()方法，通过把这个实现类交给Thread对象去运行。  

thread.start()方法里调用了start0()这个native函数来实现.start()函数的注释上写着，java虚拟机会自动调用这个线程的run()方法，然并卵。  

请看thread里面的run()怎么写的，如果target(一个Runnable实例对象)不为空，他就调用其中的run()方法。我相信start0()这个native方法里面肯定取调用了run()方法。见如下的代码

也可以把Runnable理解成只是JDK给出的一个标识性接口而已。看网上说了那么多都是浮云。
<!--more -->

```java
/**
    * Causes this thread to begin execution; the Java Virtual Machine
    * calls the <code>run</code> method of this thread.
    * <p>
    * The result is that two threads are running concurrently: the
    * current thread (which returns from the call to the
    * <code>start</code> method) and the other thread (which executes its
    * <code>run</code> method).
    * <p>
    * It is never legal to start a thread more than once.
    * In particular, a thread may not be restarted once it has completed
    * execution.
    *
    * @exception  IllegalThreadStateException  if the thread was already
    *               started.
    * @see        #run()
    * @see        #stop()
    */
    public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
        group.add(this);

        boolean started = false;
        try {
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }
    private native void start0();

    /**
     * If this thread was constructed using a separate
     * <code>Runnable</code> run object, then that
     * <code>Runnable</code> object's <code>run</code> method is called;
     * otherwise, this method does nothing and returns.
     * <p>
     * Subclasses of <code>Thread</code> should override this method.
     *
     * @see     #start()
     * @see     #stop()
     * @see     #Thread(ThreadGroup, Runnable, String)
     */
    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
    ```

# 自己写一个线程池
大家都知道要用线程池，也有一些讲Executor框架的博客之类的，咋一看理解起来不成问题，自己准备参照网上的一些例子，写一个最简单的线程池
>线程池说白了，一个List池子里面有一堆的<font color=red>Thread对象</font>，然后提交一个<font color=red>Runable对象</font>,这时候，<font color=red>从List池子里面取一个Thread对象来调用Runnable对象的run()方法而已</font>!   Thread才是真正的线程对象，他的创建才是需要开销的。

## 线程池类
```java
/**
 * Created by Intellij IDEA
 * Date: 16-5-3
 * Time: 下午2:39
 * User: thinerzq
 * Github: https://www.github.com/ThinerZQ
 * Blog: http://www.thinerzq.me
 * Email: thinerzq@gmail.com
 */

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

/**
 * 自定义线程池，学习线程池的原理
 */
public class MyThreadPool {

    //单例模式
    private static MyThreadPool instance = null;

    //空闲线程队列
    private List<MyThread> idelThreadList;

    private List<MyThread> workerThreadList;

    //总的线程数量
    private int threadCount;


    private boolean isShutdown=false;

    private MyThreadPool(){
        idelThreadList = Collections.synchronizedList(new ArrayList<>());
        workerThreadList = Collections.synchronizedList(new ArrayList<>());
        threadCount = 0 ;
    }

    //单例模式获取线程池实例
    public static synchronized MyThreadPool getInstance(){
        if (instance == null){
            instance = new MyThreadPool();
        }
        return instance;
    }

    public synchronized void  submit(Runnable target){

        MyThread workThread = null;
        if (idelThreadList.size() > 0){

            //随机取一个线程执行任务,这里取第1个空闲线程
            workThread = idelThreadList.get(0);
            idelThreadList.remove(workThread);
            //开始工作了
            workThread.setTarget(target);
        }else{
            //创建新的线程来执行任务
            threadCount ++;
            workThread = new MyThread(instance,target);
            workerThreadList.add(workThread);
            workThread.start();
        }


    }
    public boolean shutdown(){
        isShutdown = true;
        for (int i = 0; i < idelThreadList.size(); i++) {
            idelThreadList.get(i).shutdown();
        }
        return true;
    }

    public synchronized void rePushToIdeaThreadList(MyThread myThread){

        if (!isShutdown){
            idelThreadList.add(myThread);
        }else
            myThread.shutdown();
    }
    public synchronized void pullToWorkThreadList(MyThread myThread){
        if (!isShutdown){
            workerThreadList.remove(myThread);
        }else{
            myThread.shutdown();
        }
    }
    //等到线程池执行完所有任务，不一定准确，没想到好的实现方式
    public void awaitTermination() throws InterruptedException {

        while (workerThreadList.size()!=0){
            if (workerThreadList.size() == 0){
                Thread.sleep(1);
            }
        }
    }

    public int getThreadCount() {
        return threadCount;
    }


}
```

## 线程池中的线程类
为什么需要定义自己的线程？ 因为线程池里面的线程，执行完提交的任务之后，还需要复用（不能run()方法执行完就结束了）。所以需要自定线程来管理当前线程的生命周期。
```java
/**
 * Created by Intellij IDEA
 * Date: 16-5-3
 * Time: 下午2:42
 * User: thinerzq
 * Github: https://www.github.com/ThinerZQ
 * Blog: http://www.thinerzq.me
 * Email: thinerzq@gmail.com
 */
public class MyThread extends Thread {

    //当前线程所在的线程池
    private MyThreadPool pool;

    //当前线程需要执行的任务
    private Runnable target;

    //当前线程是不是结束了
    private boolean isShutDown = false;

    //当前线程是不是空闲的
    private boolean isIdle = false;

    public MyThread(MyThreadPool myThreadPool,Runnable target){
        this.pool = myThreadPool;
        this.target = target;
    }


    @Override
    public void run() {
        //当前线程的处理逻辑,只要当前现成没有结束，就不退出
        while (!isShutDown){

            //
            isIdle = false;

            if (target != null){
                //运行任务，单纯的调用target的run()方法
                target.run();
            }

            isIdle = true;
            pool.pullToWorkThreadList(this);
            try {
                //线程运行任务结束了，重新放入线程池
                pool.rePushToIdeaThreadList(this);
                synchronized (this){
                    //线程空闲，需要等待任务到来
                    wait();
                }
            }catch (Exception e){
                //放入线程池失败
                e.printStackTrace();
            }
            isIdle = false;
        }
    }
    public synchronized void setTarget(Runnable target){
        this.target = target;
        notify();
    }
    public void shutdown(){
        this.isShutDown = true;
        //要关闭的时候，要把wait()的本线程唤醒，然后再让其运行完。
        notify();
    }

    public boolean isIdle() {
        return isIdle;
    }
}
```
只要上面两个类，一个最基本的线程池就建立好了（每次提交任务的时候，如果有空闲线程就使用空闲线程，否则创建新的线程），每次只要使用submit()方法提交一个任务就好了。

下面使用一个例子，来测试一下
## 例子
### 两个任务类
```java
public class TaskThreadA implements Runnable {
    @Override
    public void run() {
        System.out.println("我是任务A,现在被执行");
    }
}
public class TaskThreadB implements Runnable {
    @Override
    public void run() {
        System.out.println("我是任务B,现在被执行");
    }
}
```
### Client类
```java
public class Client {

    public static void main(String[] args) throws InterruptedException {

        long start = System.currentTimeMillis();
        for (int i = 0; i < 100; i++) {
            Runnable runnable = null;
            if (i%2 == 0 )
                runnable = new TaskThreadA();
            else
                runnable = new TaskThreadB();
            MyThreadPool.getInstance().submit(runnable);
        }
        //等待所有任务执行完成
        MyThreadPool.getInstance().awaitTermination();

        long end = System.currentTimeMillis();
        System.out.println("time consume : "+ (end-start)+" ms");
        System.out.println("一共启动了: "+MyThreadPool.getInstance().getThreadCount()+" 个线程");

    }
}
```
### 结果
 我们提交了100个任务，可是最后只启动了42个线程，这就是线程池。Executor框架中的几种线程池都差不多是这种套路，只是控制策略根加的多。
```txt
...
我是任务B,现在被执行
我是任务B,现在被执行
我是任务A,现在被执行
我是任务B,现在被执行
我是任务A,现在被执行
我是任务A,现在被执行
我是任务B,现在被执行
我是任务A,现在被执行
我是任务B,现在被执行
我是任务B,现在被执行
我是任务A,现在被执行
我是任务B,现在被执行
我是任务A,现在被执行
我是任务B,现在被执行
我是任务A,现在被执行
我是任务B,现在被执行
time consume : 11 ms
一共启动了: 42 个线程
```
