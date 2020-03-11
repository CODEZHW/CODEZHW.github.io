---
title: if..else表达式优化策略
date: 2020-03-09T20:23
author: codezhw
summary: 
categories: Java
tags: 
	- 优化
---





# 前情提要

**if...else** 是所有高级编程语言都有的必备功能。但现实中的代码往往存在着过多的 if...else。如果我们日常工作中需要进行以一个逻辑比较复杂的判断，写完后会嵌套好几层if..else，这样代码的可读性和可维护性非常差，下面介绍几种优化if..else的策略，提高我们代码的可读性和简洁程。

# 1. 取反条件，提前return



这是我们日常中经常遇到，但是又容易忽略的情况。

- 优化前：

~~~java
if(condition){
    doSomething()
}else{
    return xxx ;
}
~~~

- 优化后：

~~~java
if（!condition）{
    return xxx;
}
doSomething()
~~~



# 2. 条件三目运算符

这在Java基础中学过，但是缺很少运用到实际中。



- 优化前

~~~java
int  num ;
if(condition){
    num = 80;
}else{
    num = 100;
}
~~~



- 优化后

~~~java
int num = condition ? 80 : 100;
~~~



# 3. 数组优化（表驱动法）

这里举例我们想要得到一年各个月份的天数。

- 优化前

~~~java
int getDays(int month){
    if (month == 1)  return 31;
    if (month == 2)  return 29;
    if (month == 3)  return 31;
    if (month == 4)  return 30;
    if (month == 5)  return 31;
    if (month == 6)  return 30;
    if (month == 7)  return 31;
    if (month == 8)  return 31;
    if (month == 9)  return 30;
    if (month == 10)  return 31;
    if (month == 11)  return 30;
    if (month == 12)  return 31;
}

~~~



- 优化后

~~~java
int monthDays[12] = {31, 29, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
int getDays(int month){
    return monthDays[--month];
}
~~~



# 4.表驱动法：

**表驱动法**，又称之为表驱动、表驱动方法。表驱动方法是一种使你可以在表中查找信息，而不必用很多的逻辑语句（if或Case）来把它们找出来的方法。以下的demo，把map抽象成表，在map中查找信息，而省去不必要的逻辑语句。



- 优化前

~~~java
if (param.equals(value1)) {
    dosomething1(someParams);
} else if (param.equals(value2)) {
    dosomething2(someParams);
} else if (param.equals(value3)) {
    dosomething3(someParams);
}
~~~



- 优化后

~~~java
Map<?, Function<?> action> actionMappings = new HashMap<>(); // 这里泛型 ? 是为方便演示，实际可替换为你需要的类型

// 初始化
actionMappings.put(value1, (someParams) -> { dosomething1(someParams)});
actionMappings.put(value2, (someParams) -> { dosomething2(someParams)});
actionMappings.put(value3, (someParams) -> { dosomething3(someParams)});

// 省略多余逻辑语句
actionMappings.get(param).apply(someParams);
~~~

这里非常的精髓，将我们的HashMap ,Function<T, R> 和Lambda 结合起来。



# 5.优化逻辑结构，让正常流程走主干



- 优化前

~~~java
public double getAdjustedCapital(){
    if(_capital <= 0.0 ){
        return 0.0;
    }
    if(_intRate > 0 && _duration >0){
        return (_income / _duration) *ADJ_FACTOR;
    }
    return 0.0;
}


~~~



- 优化后

~~~java
public double getAdjustedCapital(){
    if(_capital <= 0.0 ){
        return 0.0;
    }
    if(_intRate <= 0 || _duration <= 0){
        return 0.0;
    }
 
    return (_income / _duration) *ADJ_FACTOR;
}



//再优化
public double getAdjustedCapital(){
    if(_capital <= 0.0 ||_intRate <= 0 || _duration <= 0 ){
        return 0.0;
    }

    return (_income / _duration) *ADJ_FACTOR;
}
~~~

感觉这里的思想和取反条件，提前return差不多。



# 6. Optional



Java 代码中的一部分 if...else 是由非空检查导致的。因此，降低这部分带来的 if...else 也就能降低整体的 if...else 的个数。



- 优化前

```java
String str = "Hello World!";
if (str != null) {
    System.out.println(str);
} else {
    System.out.println("Null");
}
```



- 优化后

```java
Optional<String> strOptional = Optional.of("Hello World!");
strOptional.ifPresentOrElse(System.out::println, () -> System.out.println("Null"));
```

# 7.

# 总结

Java 8 为我们提供了强大的能力，它的新特性是划时代的（Java 8 之后的版本中新特性都不够亮眼），包括 Lambda 表达式和 Stream 流，我们通过它们可以写出简洁又高效的代码。

但是我们需要注意不能过度可以的去使用这些方法，过度不恰当的使用可能会造成我们代码的混乱，不可读性，甚至低效。

函数式编程出现的目的可不仅仅是为了减少冗余代码，它是为了解放生产力——言外之意就是说，代码复杂点没关系，只要可用可靠。编程的目标不是产生尽可能少的代码，而是产生易于维护的、高性能的系统。