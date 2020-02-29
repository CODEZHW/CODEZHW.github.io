---
title: 彻底搞懂Synchronized和Volatile
date: 2020-02-29 T15:02:51.5
author: codezhw
top: true
cover: true
summary: 这是详细讲解Java关键字Synchronize和Volatile的文章，会全面介绍Synchronized和Volatile的底层实现以及原理。
categories: Java
tags: 
- 多线程
- JUC
- Synchronize
- Volatiie
    
---





# 彻底搞懂Synchronized和Volatile



## Synchronize基本介绍



> 什么是Synchronized？谈谈你对Synchronized的理解



**Synchronized，是Java中用于解决`并发`情况下`数据同步`访问的一个很重要的关键字。当我们想要保证一个共享资源在同一时间只会被一个线程访问到时，我们可以在代码中使用Synchronized关键字对类或者对象加锁。**

---



**简单理解就是一个`锁`。**

> Synchronized实现同步的基础是什么？

synchronized实现同步的基础：

Java中的每一个对象都可以作为锁。具体表现为以下3种形式。

* **对于普通同步方法，锁是当前实例对象。**
* **对于静态同步方法，锁是当前类的Class对象。**
* **对于同步方法块，锁是Synchonized括号里配置的对象。**

对于普通同步方法和静态同步方法可能会有误解：

实例对象：**是指我们通过new关键字new出来的那个对象**

Class对象：**是每一个类对应的Class对象。也就是不关你 new 几个xx对象它们都属于同一个Class类的对象。**

可以参考八锁问题深刻理解这个三种方式。



---



> Synchronized的三大特性

这里引用《深入理解Java虚拟机》中的一段话：

> **synchronized关键字在需要原子性、可见性和有序性这三种特性的时候都可以作为其中一种解决方案，看起来万能的。的确，大部分并发控制操作都能使用synchronized来完成。**

可以看到synchronized的三大特性：

* **原子性**
* **可见性**
* **有序性**

对于这三个特性先给出概念一下再仔细探究

**原子性**：**原子指化学反应不可再分的基本微粒，原子性是指一个操作是不可中断的，要全部执行完成，要不就都不执行。**

**可见性：** **可见性是指当多个线程同时访问同一个变量时，一旦又线程改变了这个变量的值，其他线程能够立即看到修改的值。**

**有序性：**  **有序性即程序执行的顺序按照代码的先后顺序执行。**

---



那么我们现在知道了Synchronized可以通过修饰代码块和方法，来确保在同一时刻只有一个线程能够访问代码块里面的内容或者方法。

> 那这个过程是怎么实现的呢？

这里给出一个案例。

```java
public class Test1 {
    public static void main ( String[] args ) {
        // 对Synchronized Class对象进行加锁
        synchronized (Test1.class) {
        }
        //静态同步方法，对Synchronized Class对象进行加锁
        m ( );
    }

    public static synchronized void m () {
    }
}
```



我们通过JDK自带的工具,将Test1.class文件反编译成汇编查看一下synchronized的实现。

![代码块](https://i.loli.net/2020/02/29/ctrXF9B4mgDfV8R.png)

![修饰方法](https://i.loli.net/2020/02/29/dlvDbLVQiWt9Jnm.png)

《Java并发编程的艺术》中给出的解释：

> **上面class信息中，对于同步块的实现使用了`monitorenter`和`monitorexit`指令，而同步方法则 是依靠方法修饰符上的`ACC_SYNCHRONIZED`来完成的。无论采用哪种方式，其本质是对一 个对象的监视器`monitor`进行获取，而这个获取过程是排他的，也就是同一时刻只能有一个 线程获取到由synchronized所保护对象的监视器。**
> **任意一个对象都拥有自己的监视器，当这个对象由同步块或者这个对象的同步方法调用时，执行方法的线程必须先获取到该对象的监视器才能进入同步块或者同步方法，而没有获 取到监视器（执行该方法）的线程将会被阻塞在同步块和同步方法的入口处，进入BLOCKED状态。**



![](https://i.loli.net/2020/02/29/F9Zcl8B3EXukUIn.png)

从图中可以看到，任意线程对Object（Object由synchronized保护）的访问，首先要获得 Object的监视器。如果获取失败，线程进入同步队列，线程状态变为BLOCKED。当访问Object的前驱（获得了锁的线程）释放了锁，则该释放操作唤醒阻塞在同步队列中的线程，使其重新尝试对监视器的获取。



在[The Java® Virtual Machine Specification](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.10)中有关于同步方法和同步代码块的实现原理的介绍，我翻译成中文如下：

> 方法级的同步是隐式的。同步方法的常量池中会有一个`ACC_SYNCHRONIZED`标志。当某个线程要访问某个方法的时候，会检查是否有`ACC_SYNCHRONIZED`，如果有设置，则需要先获得监视器锁，然后开始执行方法，方法执行之后再释放监视器锁。这时如果其他线程来请求执行方法，会因为无法获得监视器锁而被阻断住。值得注意的是，如果在方法执行过程中，发生了异常，并且方法内部并没有处理该异常，那么在异常被抛到方法外面之前监视器锁会被自动释放。
>
> 同步代码块使用`monitorenter`和`monitorexit`两个指令实现。可以把执行`monitorenter`指令理解为加锁，执行`monitorexit`理解为释放锁。 每个对象维护着一个记录着被锁次数的计数器。未被锁定的对象的该计数器为0，当一个线程获得锁（执行`monitorenter`）后，该计数器自增变为 1 ，当同一个线程再次获得该对象的锁的时候，计数器再次自增。当同一个线程释放锁（执行`monitorexit`指令）的时候，计数器再自减。当计数器为0的时候。锁将被释放，其他线程便可以获得锁。



不论是monitorenter、monitorexit、ACC_SYNCHRONIZED、Object.wait()、Object.notify()。这些方法都是基于monitor实现的。monitor是HotSpot虚拟机中采用ObjectMonitor实现，由于水平有限，这里不做介绍。



现在来对Synchroized的三大特性做出解释：

---

## Synchroized的三大特性

 **原子性：**

线程是CPU调度的基本单位。CPU有时间片的概念，会根据不同的调度算法进行线程调度。当一个线程获得时间片之后开始执行，在时间片耗尽之后，就会失去CPU使用权。所以在多线程场景下，由于时间片在线程间轮换，就会发生原子性问题。

在Java中，为了保证原子性，提供了两个高级的字节码指令`monitorenter`和`monitorexit`。前面中，介绍过，这两个字节码指令，在Java中对应的关键字就是`synchronized`。

通过`monitorenter`和`monitorexit`指令，可以保证被`synchronized`修饰的代码在同一时间只能被一个线程访问，在锁未释放之前，无法被其他线程访问到。因此，在Java中可以使用`synchronized`来保证方法和代码块内的操作是原子性的。

 **可见性：**

造成不可见性的原因：在Java内存模型中，所有的变量都存在主存中，每条线程拥有自己的工作内存，每一个访问主存的线程都拥有一个主存的拷贝副本，将其放在线程自己的工作内存中，当一个线程对变量进行修改时，先经过工作内存再刷新到主存，另一个持有该变量的线程，再通过自己的工作内存去主存中读取新的数据。

所以当一个线程修改了数据，对另一个持有该数据的线程来说，这次的更新操作对另一个线程是不可见的。

Java中这样规定：

* 线程解锁前,必须把共享变量的最新值刷新到主内存
*  线程加锁时,将清空工作内存中共享变量的值,从而使用共享变量时需要从主内存中重新读取最新的值

 **有序性：**

除了引入了时间片以外，由于处理器优化和指令重排等，CPU还可能对输入代码进行乱序执行，比如load->add->save 有可能被优化成load->save->add 。这就是可能存在有序性问题。

这里需要注意的是，`synchronized`是无法禁止指令重排和处理器优化的。也就是说，`synchronized`无法避免上述提到的问题。

那么，为什么还说`synchronized`也提供了有序性保证呢？

这就要再把有序性的概念扩展一下了。Java程序中天然的有序性可以总结为一句话：如果在本线程内观察，所有操作都是天然有序的。如果在一个线程中观察另一个线程，所有操作都是无序的。

以上这句话也是《深入理解Java虚拟机》中的原句，但是怎么理解呢？周志明并没有详细的解释。这里我简单扩展一下，这其实和`as-if-serial语义`有关。

`as-if-serial`语义的意思指：不管怎么重排序（编译器和处理器为了提高并行度），单线程程序的执行结果都不能被改变。编译器和处理器无论如何优化，都必须遵守`as-if-serial`语义。

这里不对`as-if-serial语义`详细展开了，简单说就是，`as-if-serial语义`保证了单线程中，指令重排是有一定的限制的，而只要编译器和处理器都遵守了这个语义，那么就可以认为单线程程序是按照顺序执行的。当然，实际上还是有重排的，只不过我们无须关心这种重排的干扰。

所以呢，由于`synchronized`修饰的代码，同一时间只能被同一线程访问。那么也就是单线程执行的。所以，可以保证其有序性。



- 可见性实例

```java
//资源类
class H {
    public int num = 0;

}

public static void main ( String[] args ) throws InterruptedException {
        H h = new H ( );
        TimeUnit.SECONDS.sleep ( 5 );
        h.num = 2;
        System.out.println ( Thread.currentThread ( ).getName ( ) +"-> num的值："+ h.num);
        new Thread ( () -> {
            synchronized (h) {
            }
            System.out.println ( Thread.currentThread ( ).getName ( ) +"-> num的值："+ h.num);
            h.num--;
        }, "A" ).start ( );
        try {
            TimeUnit.SECONDS.sleep ( 2 );
        } catch (InterruptedException e) {
            e.printStackTrace ( );
        }
        System.out.println ( Thread.currentThread ( ).getName ( ) +"-> num的值："+ h.num);
    }
```

main-> num的值：2
A-> num的值：2
main-> num的值：1



---



## Volatile基本介绍

>  请你谈谈你对 Volatile 的理解

 Volatile 是轻量级的Synchronized，它的实现确保了可见性以及指令重排，不能够实现同步。

* 1、保证可见性 

* 2、不保证原子性

* 3、禁止指令重排



**Valatile定义：**

> **Java语言规范第3版中对volatile的定义如下：Java编程语言允许线程访问共享变量，为了 确保共享变量能被准确和一致地更新，线程应该确保通过排他锁单独获得这个变量。Java语言 提供了volatile，在某些情况下比锁要更加方便。如果一个字段被声明成volatile，Java线程内存**
> **模型确保所有线程看到这个变量的值是一致的。**
> 

---

## Volatile的三大特性

- **保证可见性**

```java
public class JMMDemo {
    private volatile static int num = 0;
    // 不加 volatile 程序就会死循环！    
    // 加 volatile 可以保证可见性
    // 可见性： 可见性的意思是当一个线程修改一个共享变量时，另外一个线程能读到这个修改的值
    public static void main ( String[] args ) {
        new Thread ( ()->{
            while (num ==0){
            }
        } ).start ();
        try {
            TimeUnit.SECONDS.sleep ( 1 );
        } catch (InterruptedException e) {
            e.printStackTrace ( );
        }
        num = 1;
        System.out.println ( num );
    }
}
```

![没有volatile关键字](https://i.loli.net/2020/02/26/VLwsjHvixOErZn3.png)

![有volatile关键字](https://i.loli.net/2020/02/26/wNpodxMlj7qPz4h.png)



**深入探究一下Volatile 可见性的原因**

```java
public class JMMDemo {
// 模拟一个demo去给volatile 类型变量num赋值
// 我们通过工具查看 对valatile进行写操作时，CPU做的事情
    volatile  int num;
    public static void main ( String[] args ) {
        JMMDemo jmmDemo = new JMMDemo ( );
        jmmDemo.num = 5;
    }
}
```

**0x000001bc005902d9: lock add dword ptr**

通过查IA-32架 构软件开发者手册可知，Lock前缀的指令在多核处理器下会引发了两件事情。

* **将当前处理器缓存行的数据写回到系统内存。**
* **这个写回内存的操作会使在其他CPU里缓存了该内存地址的数据无效。**

volatile的两条实现原则：



> **摘自《Java并发编程的艺术》**
>
> * Lock前缀指令会引起处理器缓存回写到内存。Lock前缀指令导致在执行指令期间，声 言处理器的LOCK#信号。在多处理器环境中，LOCK#信号确保在声言该信号期间，处理器可以 独占任何共享内存[2]。是，在最近的处理器里，LOCK＃信号一般不锁总线，而是锁缓存，毕 竟锁总线开销的比较大。对于Intel486和 Pentium处理器，在锁操作时，总是在总线上声言LOCK#信号。但在P6和目前的处理器中，如果 访问的内存区域已经缓存在处理器内部，则不会声言LOCK#信号。相反它会锁定这块内存区 域的缓存并回写到内存，并使用缓存一致性机制来确保修改的原子性，此操作被称为“缓存锁 定”，**缓存一致性机制会阻止同时修改由两个以上处理器缓存的内存区域数据。**
> * 一个处理器的缓存回写到内存会导致其他处理器的缓存无效。IA-32处理器和Intel 64处 理器使用MESI（修改、独占、共享、无效）控制协议去维护内部缓存和其他处理器缓存的一致 性。在多核处理器系统中进行操作的时候，IA-32和Intel 64处理器能嗅探其他处理器访问系统内存和它们的内部缓存。处理器使用嗅探技术保证它的内部缓存、系统内存和其他处理器的 缓存的数据在总线上保持一致。例如，在Pentium和P6 family处理器中，**如果通过嗅探一个处理器来检测其他处理器打算写内存地址，而这个地址当前处于共享状态，那么正在嗅探的处理器将使它的缓存行无效，在下次访问相同内存地址时，强制执行缓存行填充。**




- **不保证原子性（和事务的原子性一个意思）**

**原子性 : 不可分割**
**线程A在执行任务的时候，不能被打扰的，也不能被分割。要么同时成功，要么同时失败**



```java
public class JMMDemo2 {
    private static int num = 0;

    public synchronized static void add () {
        num++;
    }
    public static void main ( String[] args ) {
        for (int i = 0; i < 20; i++) {
            new Thread ( () -> {
                for (int j = 0; j < 1000; j++) {
                    add ( );
                }
            }, "A" ).start ( );
        }
        while (Thread.activeCount ( ) > 2) {
            Thread.yield ( );
        }
        System.out.println ( Thread.currentThread ( ).getName ( ) + num );
    }
}
```

- **防止指令重排**

**volatile关键字通过内存屏障来防止指令被重排序。**

**为了实现volatile的内存语义，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。然而，对于编译器来说，发现一个最优布置来最小化插入屏障的总数几乎不可能，为此，Java内存模型采取保守策略。**

**下面是基于保守策略的JMM内存屏障插入策略：**

- **在每个volatile写操作的前面插入一个StoreStore屏障。**
- **在每个volatile写操作的后面插入一个StoreLoad屏障。**
- **在每个volatile读操作的后面插入一个LoadLoad屏障。**
- **在每个volatile读操作的后面插入一个LoadStore屏障。**



**volatile的两条语义保证了线程间共享变量的及时可见性，但整个过程并没有保证同步，这是与volatile的使命有关的，创造它的背景就是在某些情况下可以代替synchronized实现可见性的目的，规避synchronized带来的线程挂起、调度的开销。如果volatile也能保证同步，那么它就是个锁，可以完全取代synchronized了。**

## 总结

* **Java中实现多线程共享变量的可见性方法有synchronize 和 volatile 。**

* **synchronize:可以用在方法或者代码块上,能保证可见性,也能保证原子性。**

* **volatitle:用在变量上,只保证可见性,不保证原子性,不加锁,比synchronize轻量级,不会造成线程阻塞.volatitle读相当于加锁,volatitle写相当于解锁。**

