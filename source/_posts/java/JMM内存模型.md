---
title: JVM内存模型和Volatile
date: 2020-02-26 15:33:51.5
author: codezhw
top: true
cover: true
summary: 这是一篇关于java多线程的进阶文章，会全面的介绍java.util.concurrent （简称JUC)下的内容，也会介绍一下实际生成过程中的情况，提高对java多线程的理解。
categories: Java
tags: 
    - 多线程

---



# JVM内存模型和Volatile



##  1. JMM

> 什么是JMM

JMM ： Java内存模型，不存在的东西，概念！约定！  

**关于JMM的一些同步的约定：**

* 1、线程解锁前，必须把共享变量立刻刷回主存。
* 2、线程加锁前，必须读取主存中的新值到工作内存中！
* 3、加锁和解锁是同一把锁  

线程  工作内存  、主内存



```java
public class JMMDemo {
    private static int num = 0;
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

---



## 2. Volatile



>  

## 3. 指令重排



什么是指令重排： 计算机并不是按照我们写的程序顺序去执行的

源代码-> 编译器优化的重排->指令并行也会重排->内存系统也会重排->执行

处理器在进行指令重排的时候，考虑：数据之间的依赖性

int x = 1; // 1 
int y = 2; // 2 
x = x + 5; // 3 
y = x * x; // 4
我们所期望的：1234  但是可能执行的时候回变成 2134  1324 可不可能是  4123

## 4.深入理解CAS

> 什么是CAS

## 5.各种锁



- **公平锁： 非常公平， 不能够插队，必须先来后到！**

- **非公平锁：非常不公平，可以插队 （默认都是非公平）**

```javajava
//默认非公平锁
public ReentrantLock() {
    sync = new NonfairSync();
}
```

****

- **可重入锁**