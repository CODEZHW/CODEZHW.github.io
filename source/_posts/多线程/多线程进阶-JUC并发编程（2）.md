---

title: 多线程进阶-JUC并发编程（2）
date: 2020-02-22T17:51:51.5
author: codezhw
summary: 这是一篇关于java多线程的进阶文章，会全面的介绍java.util.concurrent （简称JUC)下的内容，也会介绍一下实际生成过程中的情况，提高对java多线程的理解。
categories: Java
tags: 
    - 多线程
    - JUC
---

# 多线程进阶-JUC并发编程（2）

## 1. 线程池技术



线程池：三大方法、7大参数、4种拒绝策略
程序的运行，本质：占用系统的资源！ 优化资源的使用！=>池化技术
线程池、连接池、内存池、对象池///.....  创建、销毁。十分浪费资源
池化技术：事先准备好一些资源，有人要用，就来我这里拿，用完之后还给我。

### 1.1 线程池的好处:

* 1、降低资源的消耗

* 2、提高响应的速度 

* 3、方便管理。

线程复用、可以控制最大并发数、管理线程

![](https://raw.githubusercontent.com/CODEZHW/t/master/img/20200222171143.png)

线程池：三大方法

```java
/**
 * Exectors 工具类、三大方法
 * 使用了线程池之后，使用线程池来创建
 *
 * @authoer : zhw
 * @Date: 2020/2/22
 * @Description: java_study
 * @version: 1.0
 */
public class demo1 {
    public static void main ( String[] args ) {
        //ExecutorService threadPool = Executors.newSingleThreadExecutor ( );// 单个线程
        //ExecutorService threadPool = Executors.newFixedThreadPool ( 5 ); //创建一个固定的线程池的大小
        ExecutorService threadPool = Executors.newCachedThreadPool (); //可伸缩的线程
        try {
            for (int i = 0; i < 10; i++) {
                //使用线程池,来创建线程
                threadPool.execute ( ()->{
                    System.out.println ( Thread.currentThread ( ).getName ( ) + ": ok" );
                } );
            }
        } catch (Exception e) {
            e.printStackTrace ( );
        } finally {
            threadPool.shutdown ();
        }

    }
}
```

> Executors源码以及ThreadPoolExecutor源码

```java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }


public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
    

public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}



public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
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



### 1.2 线程池四种拒绝策略

>  * new ThreadPoolExecutor.AbortPolicy ()  满人了，还有人进来，不处理这个人，抛出异常。
>  *  new ThreadPoolExecutor.CallerRunsPolicy () 哪里来的，去哪里（交给谁执行）
>  *   new ThreadPoolExecutor.DiscardPolicy ()  队列满了 不会抛出异常 会放弃任务
>  *  new ThreadPoolExecutor.DiscardOldestPolicy () 队列满了，会尝试去和最早的竞争 也不会抛出异常 一个尝试性的过程
>  



### 1.3 7大参数

> * int corePoolSize, // 核心线程池大小                          
>
> * int maximumPoolSize, // 大核心线程池大小                          
>
> * long keepAliveTime, // 超时了没有人调用就会释放                          
>
> * TimeUnit unit, // 超时单位                          
>
> * BlockingQueue<Runnable> workQueue, // 阻塞队列               
>
> * ThreadFactory threadFactory, // 线程工厂：创建线程的，一般 不用动                          
>
> * RejectedExecutionHandler handle // 拒绝策略



> 手动创建线程池

```java
package hut.demo7.demo8;

import java.util.concurrent.*;

/**
 * Exectors 工具类、三大方法
 * 使用了线程池之后，使用线程池来创建
 *
 * @authoer : zhw
 * @Date: 2020/2/22
 * @Description: java_study
 * @version: 1.0
 */
public class demo1 {
    public static void main ( String[] args ) {
        ExecutorService threadPool =new ThreadPoolExecutor (
                2,
                5,
                3,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue <> ( 3 ),
                Executors.defaultThreadFactory (),
                new ThreadPoolExecutor.DiscardOldestPolicy () ); //可伸缩的线程
        //new ThreadPoolExecutor.AbortPolicy ()  满人了，还有人进来，不处理这个人，抛出异常。
        //new ThreadPoolExecutor.CallerRunsPolicy () 哪里来的，去哪里（交给谁执行）
        //new ThreadPoolExecutor.DiscardPolicy ()  队列满了 不会抛出异常 会放弃任务
        //new ThreadPoolExecutor.DiscardOldestPolicy () 队列满了，会尝试去和最早的竞争 也不会抛出异常 一个尝试性的过程
        try {
            //最大承载： deque + max
            for (int i = 1; i <= 9; i++) {
                //使用线程池,来创建线程
                threadPool.execute ( ()->{
                    System.out.println ( Thread.currentThread ( ).getName ( ) + ": ok" );
                } );
            }
        } catch (Exception e) {
            e.printStackTrace ( );
        } finally {
            threadPool.shutdown ();
        }

    }
}
```

>  小结和拓展

池的大的大小如何去设置！
了解：IO密集型，CPU密集型：（调优）





