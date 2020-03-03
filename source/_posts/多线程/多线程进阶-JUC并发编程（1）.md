---

title: 多线程进阶-JUC并发编程（1）
date: 2020-02-22T15:51:51.5
author: codezhw
summary: 这是一篇关于java多线程的进阶文章，会全面的介绍java.util.concurrent （简称JUC)下的内容，也会介绍一下实际生成过程中的情况，提高对java多线程的理解。
categories: Java
tags: 
    - 多线程
    - JUC

---



# 多线程进阶-JUC并发编程（1）


## 1、什么是JUC



​		在 Java 5.0 提供了 java.util.concurrent （简称JUC ）包,在此包中增加了在并发编程中很常用的实用工具类，用于定义类似于线程的自定义子系统，包括线程池、异步 IO 和轻量级任务框架。提供可调的、灵活的线程池。还提供了设计用于多线程上下文中的 Collection 实现等。

![](https://raw.githubusercontent.com/CODEZHW/t/master/img/juc.png)

## 2、线程和进程 



 ###  2.1 进程

进程：一个程序，qq.exe , Music.exe  程序的集合。
 一个进程往往可以包含多个线程，至少包含一个！

 Java默认有几个线程?

 2 个   mian、GC

### 2.2 线程

线程：开了一个进程 Typora，写字，自动保存（线程负责的） 

对于Java而言：Thread、Runnable、Callable 

Java 真的可以开启线程吗？ 

开不了

> 下面是Thread类的start()方法的源码（我们开启线程的方式new Thread().start()）

```java
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

// native 为本地方法，他调用了底层的C++去开启线程
private native void start0();
```

Java是运行在虚拟机上的，无法操控硬件。

### 2.3 并发与并行

并发（多线程操作同一个资源）

* CPU 一核 ，模拟出来多条线程（多个人一起行走）

并行（多个人一起行走）

* CPU 多核 ，多个线程可以同时执行； 线程池

并发编程的本质：充分利用CPU的资源 

### 2.4 线程有几个状态

```java
public enum State {
    /**
     * 创建
     */
    NEW,

    /**
     * 运行
     */
    RUNNABLE,

    /**
     * 阻塞
     */
    BLOCKED,

    /**
     * 等待
     */
    WAITING,

    /**
     * 超时等待
     */
    TIMED_WAITING,

    /**
     * 终止
     */
    TERMINATED;
}
```

### 2.5 wait /sleep 的区别

1. 来自不同的类

   wait => Object

   sleep => Thread 

   

2. 关于锁的释放 

   wait 会释放锁，sleep 睡觉了，抱着锁睡觉，不会释放！

   

3. 使用的范围是不同的 

   wait 必须使用在同步代码块中

   sleep 可以再任何地方睡 

## 3、 Lock锁

### 3.1传统的synchronized锁



> 下列代码模拟了一个买票业务
>
> * Ticket ：资源类，业务中多线程去操作的资源类
> * main： 开启三个线程，采用了lambdom表达式的方法创建了三个线程 A B C

```java
/**
 * 买票问题，初步展示synchronized版的多线程问题
 * @authoer : zhw
 * @Date: 2020/2/19
 * @Description: java_study
 * @version: 1.0
 */
public class SaleTicket {
    public static void main ( String[] args ) {
        Ticket ticket = new Ticket ( );

        new Thread ( () -> {
            for (int i = 0; i < 30; i++) {
                ticket.sale ( );
            }
        }, "A" ).start ( );
        new Thread ( () -> {
            for (int i = 0; i < 50; i++) {
                ticket.sale ( );
            }
        }, "B" ).start ( );
        new Thread ( () -> {
            for (int i = 0; i < 20; i++) {
                ticket.sale ( );
            }
        }, "C" ).start ( );

    }
}


class Ticket  {
    private int number = 50;

    public synchronized void sale () {
        if (number > 0) {
            System.out.println ( Thread.currentThread ( ).getName ( ) + "卖出了第" + (number--) +票,剩余：" + number );
        }
    }
}
```

> 这是不加上synchronized执行的结果可以看出在没有锁的情况下，多个线程同时对统一资源进行操作造成的结果混乱。

![](https://raw.githubusercontent.com/CODEZHW/t/master/img/result.png)

```java
/**
 * 买票问题，初步展示JUC版的多线程问题 (lock版本)
 (JUC:java.util.concurrent包)
 * @authoer : zhw
 * @Date: 2020/2/19
 * @Description: java_study
 * @version: 1.0
 */
public class SaleTicketdemo2 {
    public static void main ( String[] args ) {
        Ticket1 ticket = new Ticket1 ( );
        new Thread ( ()->{for (int i = 0 ; i<40; i++) ticket.sale ();},"A" ).start ();
        new Thread ( ()->{for (int i = 0 ; i<40; i++) ticket.sale ();},"B" ).start ();
        new Thread ( ()->{for (int i = 0 ; i<40; i++) ticket.sale ();},"C" ).start ();

    }
}


class Ticket1 {
    //属性 方法
    private int number = 50;
    Lock lock = new ReentrantLock ( );

    public void sale () {
        lock.lock ( ); //加锁

        try {
            //业务
            if (number > 0) {
                System.out.println ( Thread.currentThread ( ).getName ( ) + "卖出了第" + (number--) + "票,剩余：" + number );
            }
        } catch (Exception e) {
            e.printStackTrace ( );
        } finally {
            //解锁
            lock.unlock ();
        }
    }
}
```

Lock三部曲 

* 1、new ReentrantLock(); 

* 2、 lock.lock(); // 加锁 

* 3、finally=>  lock.unlock(); // 解锁

### 3.2synchronized和lock锁的区别

* 1、Synchronized   内置的Java关键字，  Lock 是一个Java类*
*  2、Synchronized  无法判断获取锁的状态，Lock  可以判断是否获取到了锁
* 3、Synchronized  会自动释放锁，lock 必须要手动释放锁！如果不释放锁，死锁 
* 4、Synchronized   线程 1（获得锁，阻塞）、线程2（等待，傻傻的等）；Lock锁就不一定会等待下 去；
* 5、Synchronized    可重入锁，不可以中断的，非公平；Lock ，可重入锁，可以 判断锁，非公平（可以 自己设置）；
* 6、Synchronized     适合锁少量的代码同步问题，Lock  适合锁大量的同步代码！  

> 锁是什么，如何判断锁的是谁！

## 4、生产者和消费者问题

> 生产者和消费者问题 Synchronized版

```java
package hut.demo7.demo2;


/**
 * 线程之间的通信问题：生产者和消费者问题！ 等待唤醒，通知唤醒。
 * 线程交替执行， A B 同事操作一个变量 NUM = 0
 * A num + 1
 * B num - 1
 *
 * @authoer : zhw
 * @Date: 2020/2/19
 * @Description: java_study
 * @version: 1.0
 */
public class A {
    public static void main ( String[] args ) {
        Data data = new Data ( );
        new Thread ( () -> {
            for (int i = 1; i < 10; i++) {
                try {
                    System.out.println ( "第" + i + "次执行add()操作" );
                    data.add ( );
                } catch (InterruptedException e) {
                    e.printStackTrace ( );
                }
            }
        }, "A" ).start ( );
        new Thread ( () -> {
            for (int i = 1; i < 10; i++) {
                try {
                    System.out.println ( "第" + i + "次执行dec()操作" );
                    data.dec ( );
                } catch (InterruptedException e) {
                    e.printStackTrace ( );
                }
            }
        }, "B" ).start ( );
        new Thread ( () -> {
            for (int i = 1; i < 10; i++) {
                try {
                    System.out.println ( "第" + i + "次执行add()操作" );
                    data.add ( );
                } catch (InterruptedException e) {
                    e.printStackTrace ( );
                }
            }
        }, "C" ).start ( );
        new Thread ( () -> {
            for (int i = 1; i < 10; i++) {
                try {
                    System.out.println ( "第" + i + "次执行dec()操作" );
                    data.dec ( );
                } catch (InterruptedException e) {
                    e.printStackTrace ( );
                }
            }
        }, "D" ).start ( );
    }
}

//资源类 通用：等待，业务，通知
class Data {
    private int number = 0;

    public synchronized void add () throws InterruptedException {
        while (number != 0) {
            //等待
            this.wait ( );
        }
        number++;
        //通知其他线程，加一完毕了
        System.out.println ( Thread.currentThread ( ).getName ( ) + "-> :" + number );
        this.notifyAll ( );
    }

    public synchronized void dec () throws InterruptedException {

        while (number == 0) {
            //等待
            this.wait ( );
        }
        number--;
        //通知其他线程，减一完毕了
        System.out.println ( Thread.currentThread ( ).getName ( ) + "-> :" + number );
        this.notifyAll ( );
    }
}
```

>可能存在的问题：ABCD四个线程同时存在可能会产生虚假唤醒.
>
>解决方案： 将判断业务逻辑的if改为while.

> juc版本的生产者消费者问题

```java
package hut.demo7.demo2;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @authoer : zhw
 * @Date: 2020/2/19
 * @Description: java_study
 * @version: 1.0
 */
public class B {
    public static void main ( String[] args ) {
        Data2 data2 = new Data2 ( );
        new Thread ( () -> {
            for (int i = 1; i < 10; i++) {
                try {
                    System.out.print ( "第" + i + "次执行add()操作：" );
                    data2.add ( );
                } catch (InterruptedException e) {
                    e.printStackTrace ( );
                }
            }
        }, "A" ).start ( );
        new Thread ( () -> {
            for (int i = 1; i < 10; i++) {
                try {
                    System.out.print ( "第" + i + "次执行dec()操作：" );
                    data2.dec ( );
                } catch (InterruptedException e) {
                    e.printStackTrace ( );
                }
            }
        }, "B" ).start ( );
    }
}
class Data2 {
    private int number = 0;
    Lock lock = new ReentrantLock (  );
    Condition condition = lock.newCondition ( );
    public  void add () throws InterruptedException {
        lock.lock ();
        try {
            while (number != 0) {
                //等待
                condition.await ();
            }
            number++;
            //通知其他线程，加一完毕了
            System.out.println ( Thread.currentThread ( ).getName ( ) + "-> :" + number );
            condition.signalAll ();
        } catch (InterruptedException e) {
            e.printStackTrace ( );
        } finally {
            lock.unlock ();
        }
    }

    public  void dec () throws InterruptedException {
        lock.lock ();
        try {
            while (number == 0) {
                //等待
                condition.await ();
            }
            number--;
            //通知其他线程，减一完毕了
            System.out.println ( Thread.currentThread ( ).getName ( ) + "-> :" + number );
            condition.signalAll ();
        } catch (InterruptedException e) {
            e.printStackTrace ( );
        } finally {
            lock.unlock ();
        }
    }
}
```

任何一个新的技术，绝对不是仅仅只是覆盖了原来的技术，优势和补充！

> Condition 精准的通知和唤醒线程 

```java
package hut.demo7.demo2;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @authoer : zhw
 * @Date: 2020/2/19
 * @Description: java_study
 * @version: 1.0
 */
public class C {
    public static void main ( String[] args ) {

        Data3 data3 = new Data3 ( );
        new Thread ( () -> {
            for (int i = 0; i < 5; i++) {
                data3.printA ( );
            }
        }, "A" ).start ( );
        new Thread ( () -> {
            for (int i = 0; i < 5; i++) {
                data3.printB ( );
            }
        }, "B" ).start ( );
        new Thread ( () -> {
            for (int i = 0; i < 5; i++) {
                data3.printC ( );
            }
        }, "C" ).start ( );
    }
}

class Data3 {
    private Lock lock = new ReentrantLock ( );
    private Condition condition1 = lock.newCondition ( );
    private Condition condition2 = lock.newCondition ( );
    private Condition condition3 = lock.newCondition ( );
    private int number = 1; //1A 2B 3C

    public void printA () {
        lock.lock ( );
        try {
            //业务 判断 执行
            while (number != 1) {
                condition1.await ( );
            }
            number=2;
            System.out.println ( Thread.currentThread ( ).getName ( ) + "=> A" );
            condition2.signal ( );
        } catch (Exception e) {
            e.printStackTrace ( );
        } finally {
            lock.unlock ( );
        }

    }

    public void printB () {
        lock.lock ( );
        try {
            //业务 判断 执行
            while (number != 2) {
                condition2.await ( );
            }
            number=3;
            System.out.println ( Thread.currentThread ( ).getName ( ) + "=> B" );
            condition3.signal ( );
        } catch (Exception e) {
            e.printStackTrace ( );
        } finally {
            lock.unlock ( );
        }
    }

    public void printC () {
        lock.lock ( );
        try {
            //业务 判断 执行
            while (number != 3) {
                condition3.await ( );
            }
            number=1;
            System.out.println ( Thread.currentThread ( ).getName ( ) + "=> C" );
            condition1.signal ( );
        } catch (Exception e) {
            e.printStackTrace ( );
        } finally {
            lock.unlock ( );
        }
    }
}
```

