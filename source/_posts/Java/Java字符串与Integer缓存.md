---
title: Java字符串与Integer缓存
date: 2020-03-17T20:23
author: codezhw
summary: Java字符串’s1==s2‘与’s1.equals(s2)‘有什么区别吗？这是一个老生常谈的问题，这次从自己的角度去分析这个问题加深一下理解，顺便谈一谈Integer缓存问题。
categories: Java
tags: 
	- 源码
---



# Java字符串



- S1==S2和S1.equals(S2)有什么区别吗？

我们都知道在Java语义中，`==`判断的对象的内存地址是否相等。（暂时不去探究JVM怎么解析==这个语法的以及JVM怎么把“”之间的内容解析成字符串）



那现在问题来了：

```java
String s1 = new String ( "123" );
String s2 = new String ( "123" );
var s3 = "123";
var s4 = "123";

System.out.println ( "s1: 内存地址 => " + System.identityHashCode ( s1 ) + "  s1: 内存地址 => " + System.identityHashCode ( s2 ) + "  s1 == s2 : " + (s1 == s2) );

System.out.println ( "s3: 内存地址 => " + System.identityHashCode ( s3 ) + "  s4: 内存地址 => " + System.identityHashCode ( s4 ) + "  s3 == s4 : " + (s3 == s4) );
```

**我们不能通过s.hashcode()方法得到对象的内存地址，因为String重写了这个方法，它返回对象的hash值。**

你觉得结果会是怎样呢？

![结果](https://gitee.com//CODEZHW/blogimage/raw/master/img/20200317205955.png)



很容易验证我们心中所想：

- **s1 != s2 因为他们是通过new关键字创建出来的，存储在Java堆中对象的内存地址不相等。**
- **s3 == s3 因为JVM解析“”将里面的内容存放在字符串常量池中，因为是同一个字符串位于常量池中，所以他们的引用相同，内存地址也就相同**



**Java在JDK1.7之后将字符串常量池放入堆中。**



# Integer缓存



```java
Integer integer1 = 1;
Integer integer2 = 1;
System.out.println (integer1 == integer2 );

Integer integer3 = 128;
Integer integer4 = 129;
System.out.println (integer3 == integer4 );
```



结果：true

​	 false



Integer integer1 = 1;

会通过自动装箱的机制：

Integer integer1 = 1; 相当于 Integer integer1 = Integer.valueOf(1);

我们来看valueOf()的源码

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

可以看到当i的范围在i >= IntegerCache.low && i <= IntegerCache.hig之间时，会在

IntegerCache.cache[]数组中去寻找。

IntegerCache.low = -128；

IntegerCache.hig = +127；

cache[]中存储的也是-128~+127

```java

//IntegerCache 静态代码块中的一段 high = 127 low = -128
int size = (high - low) + 1;

// Use the archived cache if it exists and is large enough
if (archivedCache == null || size > archivedCache.length) {
    Integer[] c = new Integer[size];
    int j = low;
    for(int i = 0; i < c.length; i++) {
        c[i] = new Integer(j++);
    }
    archivedCache = c;
}
cache = archivedCache;
```

由此可见，当Integer对象的值在-128~+127之间，它不会创建对象会从Integer缓存中提取。当Integer超过这个范围时会直接创建对象。