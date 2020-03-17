---
title: OutOfMemoryError异常
date: 2020-03-10T15:23
author: codezhw
summary: 这是阅读《深入理解Java虚拟机:JVM高级特性与最佳实践(第3版)》的读书笔记，记录学习过程以及一些自己的理解。
categories: JVM
tags: 
	- Java
	- 读书笔记
---





# OutOfMemoryError异常





## Java堆异常



我们都知道Java的堆用于存储对象实例，想要得到堆溢出异常，我们只要不断的创建对象就可以，GC Roots到对象之间有可达路径来避免垃圾回收机制清除这些对象，那么随着对象数量的增加，总容量触及最大堆的容量限制后就会产生内存溢出异常，同时通过参数设置避免堆的自动扩充。

VM参数:-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=C:\Users\USERzhw

```java
public class HeapOom {
    public static void main ( String[] args ) {
        ArrayList <OomObject> list = new ArrayList <> ( );

        while (true) {
            list.add ( new OomObject ( ) );
        }
    }

    static class OomObject {
    }
}
```

![运行结果](https://i.loli.net/2020/03/10/VR2Nb7m8EnqKhvZ.png)

可以看到成功抛出了内存溢出异常，位于Java中的堆空间。

---



## 虚拟机栈和本地方法栈溢出



```java
public class StacksOf {

    private int stackLength = 1;

    public static void main ( String[] args ) {
        StacksOf stacksOf = new StacksOf ( );
        try {
            stacksOf.StackLeak ( );
        } catch (Throwable e) {
            System.out.println ( "stackLength = " + stacksOf.stackLength );
            throw e;
        }
    }

    public void StackLeak () {
        stackLength++;
        StackLeak ( );
    }
}
```



![结果](https://i.loli.net/2020/03/10/khv1WmFPRSZrKpU.png)

---



## 方法区和运行时常量池溢出



