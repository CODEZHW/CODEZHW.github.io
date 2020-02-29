---
title: Java8新特性:函数式编程以及Stream流式计算
date: 2020-02-27T13:30
author: codezhw
top: true
summary: 这是一篇关于Java8新特性，函数式接口以及Stream流式计算的文章，作为新时代的程序必须掌握的JDK8新特性：Lambda表达式、链式编程、函数式接口、Stream流式计算，本文主要介绍了函数式接口以及Stream流式计算, Lambda表达式和链式编程夹带讲解。
categories: Java
tags: 
- 函数式接口
- Stream流式计算


---

# Java8新特性:black_heart:函数式接口以及Stream流式计算

## 1.四大函数式接口

java 8 四大新特性：  **Lambda表达式**，**链式编程**， **函数式接口**，**Stream流式计算**。

>* 什么是函数式接口
>
>**有且只有一个抽象方法的接口被称为函数式接口，函数式接口适用于函数式编程的场景，Lambda就是Java中函数式编程的体现，可以使用Lambda表达式创建一个函数式接口的对象，一定要确保接口中有且只有一个抽象方法，这样Lambda才能顺利的进行推导。必须要有@FunctionalInterface 注解，可以有默认方法。**

**带有@FunctionalInterface注解的接口就是函数式接口**

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}

// 泛型、枚举、反射 
// lambda表达式、链式编程、函数式接口、Stream流式计算 
// 超级多FunctionalInterface 
// 简化编程模型，在新版本的框架底层大量应用！ 
// foreach(消费者类的函数式接口)
```

![四大常用函数式接口](https://raw.githubusercontent.com/CODEZHW/t/master/img/20200222175528.png)

**主要语法:**

1.  **() ->{} 代表了 lambda的一个表达式**
2.  **单行代码无需写return (无论函数式接口有没有返回值),花括号**
3.  **多行代码必须写花括号,有返回值的一定要写返回值**
4.  **单行代码且有参数的情况下可以不写 ()  如  s->System.out.println(s)**
5.  **(T t)中的参数类型可写可不写**

**函数式编程的思想**

			1. **我们都知道可以用匿名内部类的方式去new 一个接口（Runnable接口的实现）**
   			2. **在JDK8中的官方文档中，lambda表示可以代替匿名内部类去创建函数式接口。**



<center>举例四大函数式接口：</center>

> **Function 函数型接口, 有一个输入参数，有一个输出**

```java
public class demo1 {
    public static void main ( String[] args ) {
        //Function function = new Function  <String, String>() {
        //    @Override
        //    public String apply ( String str ) {
        //        return str;
        //    }
        //};
        Function function = ( str) -> { return str; };
        System.out.println ( function.apply ( "ads" ) );
    }
}

```


> **Predicate断定型接口： 有一个输入参数， 返回值只能是布尔值**。

```java
public class demo2 {
    public static void main ( String[] args ) {
        //判断字符是否为控
        //Predicate <String> predicate = new Predicate <> ( ) {
        //    @Override
        //    public boolean test ( String str ) {
        //        return str.isEmpty ( );
        //    }
        //};
        Predicate <String> predicate = str ->{return str.isEmpty ();};
        System.out.println ( predicate.test ( "123" ) );

    }
}
```


> **Consumer消费型接口，只有输入值，没有返回值**

```java
public class demo3 {

    public static void main ( String[] args ) {
        //Consumer <String> consumer = new Consumer <> ( ) {
        //    @Override
        //    public void accept ( String str ) {
        //        System.out.println ( str );
        //    }
        //};
        Consumer <String> consumer = str -> 	   System.out.println ( str );
        consumer.accept ( "测试Consumer" );
    }
}
```


> **Supplier 供给型接口，没有参数，有返回值**

```java
public class demo4 {
    public static void main ( String[] args ) {
        //Supplier <Object> supplier = new Supplier <> ( ) {
        //    @Override
        //    public Integer get () {
        //        return 1024;
        //    }
        //};
        Supplier  supplier = () -> {return 1024;};
        System.out.println ( supplier.get ( ) );
    }
}
```

---



## 2. 流式计算



>* 什么是Java Stream流
>
>**流提供了一种让我们可以在比集合更高的概念级别上指定计算的数据视图。通过使用流，我们可以说明想要完成什么任务，而不是说明如何去实现它。将操作的调度留给具体实现去做。**
>
>**流遵循了做什么而非怎么做的原则。在流的示例中，我们描述了需要做什么，没有指定该操作应该以什么顺序或者在哪个线程中执行。**



流和集合的差异：

1. **流并不存储元素，这些元素可能存储在底层的集合中，或者是按需生成的。**
2. **流的操作不会修改其数据源。例如，filter方法不会从新的流中移除元素，而是会生成一个新的流，其中不包含被过滤掉的元素。**
3. **流的操作是尽可能惰性执行的。这意味着直至需要其结果时，操作才会执行。**



**Collection中的stream（）方法可以将当前集合转变为Stream流对象**

```java
default Stream<E> stream() {
    return StreamSupport.stream(spliterator(), false);
}
```

**对于数组可以通过 Stream.of（）静态方法去创建**

```java
public static<T> Stream<T> of(T... values) {
    return Arrays.stream(values);
}
```

**Stream**的操作符大体上分为两种：**中间操作符**和**终止操作符**

**中间操作符**

* filter	    元素过滤：对Stream对象按指定的Predicate进行过滤，返回的Stream对象中仅包含未被过滤的元素

* map	        元素一对一转换：使用传入的Function对象对Stream中的所有元素进行处理，返回的Stream对象中的元素为原元素处理后的结果

* mapToInt	元素一对一转换：将原Stream中的使用传入的IntFunction加工后返回一个IntStream对象

* flatMap	 元素一对多转换：对原Stream中的所有元素进行操作，每个元素会有一个或者多个结果，然后将返回的所有元素组合成一个统一的Stream并返回；

* distinct	    去重：返回一个去重后的Stream对象

* sorted	      排序：返回排序后的Stream对象

* peek	         使用传入的Consumer对象对所有元素进行消费后，返回一个新的包含所有原来元素的Stream对象

* limit	      	获取有限个元素组成新的Stream对象返回

* skip	 	 抛弃前指定个元素后使用剩下的元素组成新的Stream返回

**常用的终止操作符**

* collect 收集操作，将所有数据收集起来，这个操作非常重要，官方的提供的Collectors 提供了非常多收集器，可以说Stream 的核心在Collectors。

* count 统计操作，统计最终的数据个数。

* findFirst、findAny 查找操作，查找第一个、查找任何一个 返回的类型为Optional。

* noneMatch、allMatch、anyMatch 匹配操作，数据流中是否存在符合条件的元素 返回值为bool 值。

* min、max 最值操作，需要自定义比较器，返回数据流中最大最小的值。

* reduce 规约操作，将整个数据流的值规约为一个值，count、min、max底层就是使用reduce。

* forEach、forEachOrdered 遍历操作，这里就是对最终的数据进行消费了。

* toArray 数组操作，将数据流的元素转换成数组。



- 举例

```java
//查找所有包含t的元素并进行打印
Stream.of("test", "t1", "t2", "teeeee", "aaaa").filter ( (str )-> { return str.contains ( "t" ) ;}).forEach ( System.out::println );
```

```java
//将所有字符串大写
Stream.of ( "test", "hello", "world", "teeeee", "aaaa" ).map ( String::toUpperCase ).forEach ( System.out::println );
```

```java
//去重
Stream.of ( 1, 2, 3, 4, 5, 1, 1, 2, 4 ).distinct ( ).forEach ( System.out::println );
```

 **Optional**

用于简化Java中对空值的判断处理，以防止出现各种空指针异常。
Optional实际上是对一个变量进行封装，它包含有一个属性value，实际上就是这个变量的值。



用于创建一个空的Optional对象；其value属性为Null。
如：

```java
Optional o = Optional.empty();
```

根据传入的值构建一个Optional对象;
传入的值必须是非空值，否则如果传入的值为空值，则会抛出空指针异常。
使用：

```java
o = Optional.of("test"); 
```



```java
String S = "test";
Optional.of ( S ).ifPresent ( (str)->{
    System.out.println ( "str不为空，str = " + str );
} );
```



```java
//变量为空时提供默认值
Optional.of ( "" ).ofNullable ( "default" ).stream ().forEach ( System.out::println );
```



关于lambda表达式可以看：这篇文章《》https://juejin.im/post/5d2d15825188253d7201d297#heading-4