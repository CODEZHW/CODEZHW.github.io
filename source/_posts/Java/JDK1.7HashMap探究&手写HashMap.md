---
title: JDK1.7HashMap探究&手写HashMap
date: 2020年2月23日 15:2621
author: codezhw
summary: 这是一篇关于Java JDK1.7版本HashMap的文章，了解HashMap的优势与缺点，探究底层的实现原理以及底层代码所实现的目的,简单探究了一下HashMap产生死锁的原因,以及手写一个HashMap。
categories: Java
tags: 
    - HashMap
    - 源码



---

# JDK1.7HashMap探究&手写HashMap



## HashMap源码分析

> 什么是HashMap?

简单来说，HashMap是用哈希表（直接一点可以说数组加单链表）的Map实现类。

在JDK 1.7中HashMap的底层数据结构： **链表** + **数组**



> 什么是Hash

Hash函数是把任意长度的输入（又叫做预映射pre-image）通过散列算法变换成固定长度的输出，该输出就是散列值。

这种转换是一种压缩映射，散列值的空间通常远小于输入的空间，不同的输入可能会散列成相同的输出，所以不可能从散列值来确定唯一的输入值。



**简单来说HashMap就是通过Hash函数把传入的key转化为hash值， 然会对这个值进行操作得到一个下标，这个下标就是这个传入的对象(key, value)在数组中的下标，这个元素就是链表的头节点（在数组当前下标只有一个元素的时候），后续再有元素加进来的时候，就把新加进来的元素作为链表的头结点，原头结点作为新头结点的下一元素。**



> HashMap字段

```java
/**
 * 默认容量 16
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

/**
 * 默认最大容量
 */
static final int MAXIMUM_CAPACITY = 1 << 30;

/**
 * 默认负载因子
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;

/**
 * 数组
 */
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;

/**
 * 记录容器中的大小
 */
transient int size;

//负载
int threshold;

/**
 * 可设置的负载因子
 */
final float loadFactor;

/**
 * 记录进行的put() get()的数量
 */
transient int modCount;

```



HashMap 的实例有两个参数影响其性能：初始容量 和加载因子。

容量是哈希表中桶的数量，初始容量只是哈希表在创建时的容量。

加载因子 是哈希表在其容量自动增加之前可以达到多满的一种尺度。

当哈希表中的容量超出了加载因子与当前容量的乘积时，则要对该哈希表进行 resize操作（即扩容），从而哈希表将具有大约两倍的桶数。



> HashMap中两个重要参数的默认值：
>
> * static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
>* static final float DEFAULT_LOAD_FACTOR = 0.75f;



**构造方法**

```java
public HashMap() {
    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
}
//  调用下面这个方法 设置初始化容量和负载因子

public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);

    this.loadFactor = loadFactor;
    threshold = initialCapacity;
    init();
}
```

###  put()方法

```java
public V put(K key, V value) {
    //1.初始化容器
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    if (key == null)
        //可以存入一个 key==null 的值
        return putForNullKey(value);
    //2.算出hash值 
    int hash = hash(key);
    //3.通过hash值算出这个key，对于的数组下标 
    int i = indexFor(hash, table.length);
    //4.遍历这个结点的链表 
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
	
    modCount++;
    // 5.增加节点
    addEntry(hash, key, value, i);
    return null;
}
```

> 一步步介绍put方法中的步骤：

```java

private void inflateTable(int toSize) {
    // 如果数组大小不是2的整数幂，就向上取2的整数幂（15就取 16  30 取 32）
    int capacity = roundUpToPowerOf2(toSize);

    threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
    table = new Entry[capacity];
    initHashSeedAsNeeded(capacity);
}
```

​    **int capacity = roundUpToPowerOf2(toSize);是很重要的操作，在下面会介绍**

> 取数组下标的方法

对于一个任意的数 N ，我们想要得到 范围：**0 ~M-1** 的方法（M为数组容量）：

对于任意值n : n % m 就会得到范围为 0~m-1 的数，这个值就可以作为数组的下标。

```java
//取地下标的方法
static int indexFor(int h, int length) {
    // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
    return h & (length-1);
}
```

在HashMap中则是通过  **h & (length-1)**  将 key的hash值 与 数组长度-1进行与操作。

 而这个数组的长度永远都是2的整数幂，那2的整数幂-1 的二进制的最后几位都是1

比如：16-1 = 15  （15的二进制：0000 1111）

这样的操作保证**h & (length-1)**后得到的结果在0~length-1这个范围之内。

举例：

 **h : xxxx 1010 （假设的一个hash值）**

 **l :  0000 1111（16- 1）**

 **r:  0000 1010   （结果，就是这个key对应的数组下标）**

**这样对于任意一个hash值，他在与(length-1)做与运算后的结果范围永远在： 0~15之间，这样得到的下标就会在容器的长度范围之内**

**相比于%得到数组下标，这样做的好处在于位运算的效率比较高。**

> **可以发现一个问题：在做与运算操作的时候，hash值的高位没有参与运算，那么不同的hash值，如果低位相同，通过运算得到的数组下标可能一样，这样散列性就很差。**

**再看hash()方法**

```java
final int hash(Object k) {
    int h = hashSeed;
    if (0 != h && k instanceof String) {
        return sun.misc.Hashing.stringHash32((String) k);
    }
    h ^= k.hashCode();
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

> **hash（）方法通过对 k进行右移运算和异或运算 ，使得 hash值的低位具有高位的特性，这样算出来的hash值拥有一个良好的散列性，可以解决hash冲突，得到的数组下标就更有意义。**



**HashMap 链表对象**

```java
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    int hash;

    /**
     * Creates new entry.
     */
    Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    }
```

```java
//遍历这个结点的链表，如果传入一样的key值，判断value是否相等，如果一样新值，覆盖老值，然后返回。
for (Entry<K,V> e = table[i]; e != null; e = e.next) {
    Object k;
    if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
        V oldValue = e.value;
        e.value = value;
        e.recordAccess(this);
        return oldValue;
    }
}
```

**创建新结点**

```java
void addEntry(int hash, K key, V value, int bucketIndex) 
{
   // 添加新元素前先判断数组的大小是否大于等于阀值，如果是且数组下标位置已经存在元素则对数组进行扩容，并对新的key重新根据新的数组长度计算下标
    if ((size >= threshold) && (null != table[bucketIndex])) {
        //扩容 这是导致hashmap在多线程环境下产生死锁的原因。等下分析
        resize(2 * table.length);
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }

    //table[i]==null 创建头结点, !=null 创建新节点并将其执行原头结点，然后将链表整体后移（即添加链表的方式是头部添加，不是尾部添加）。
    createEntry(hash, key, value, bucketIndex);
}
```

```java
void createEntry(int hash, K key, V value, int bucketIndex) {
    //创建一个新结点e 为其赋值为当前位置 talble[bucketIndex]的结点
    Entry<K,V> e = table[bucketIndex];
    // 创建一个新的头节点，然后next指向e 完成头部插入操作
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    // 统计元素+1
    size++;
}
```

> **在了解了HashMap头插法创建链表之后，我们来看数组的扩容（头插法的目的是为不用遍历整个链表，提高效率。）**
>
> * **正是这种提高效率的方法，导致了在多线程环境下，链表变成循环链表产生死锁的原因**

**扩容**

```java
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }
    //扩容一个新的数组，他的长度是原数组的两倍。
    Entry[] newTable = new Entry[newCapacity];
    //将旧数组内容，转移到新数组。
    //关键，在多线程环境下会产生死锁。
    transfer(newTable, initHashSeedAsNeeded(newCapacity));
    table = newTable;
    //新的容量限制 = 新的容量 * 负载因子
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
```

```java
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    //遍历原数组
    for (Entry<K,V> e : table) {
        while(null != e) {
            //创建一个结点，记录当前结点的下一个元素。
            Entry<K,V> next = e.next;
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            //计算当前结点在新数组中的位置。
            int i = indexFor(e.hash, newCapacity);
            //将当前结点的next赋值为新数组下标为i的元素
            e.next = newTable[i];
            //将当前节点赋值给新数组
            newTable[i] = e;
            //节点赋值为下一个元素
            e = next;
        }
    }
}
```

> 这里的扩容操作，我用图来说明。

![](https://i.loli.net/2020/02/24/579GMWsCgwQzJlf.png)

![](https://i.loli.net/2020/02/24/D4WGTZLlRznFX21.png)

> 来分析一下产生死锁的情况。
>
> 假设AB两个线程同时进行对容器 进行扩容操作

### 产生死锁原因分析

**假设这是初始情况**

![初始状况](https://i.loli.net/2020/02/24/Q8ALqr92liuV1Jb.png)

**A放了元素刘备之后，这时A线程突然进入等待**

**B线程进来，执行扩容操作后的结果如下**

![](https://i.loli.net/2020/02/24/mjbH7uq6Gxc9Jfg.png)

**重点!  线程 A 和线程 B 扩容的数组是私有的，可元素是共有的。**

**B执行完操作，线程调度交给A线程**

**A放完黄盖和张飞后, 发现本来张飞后面的元素null 变成了刘备,这样就形成了循环链表产生死锁,程序运行到这里cpu100%爆炸。**

![](https://i.loli.net/2020/02/24/laUyw8FVmSvzIud.png)

### get()方法

```java
public V get(Object key) {
    if (key == null)
        return getForNullKey();
    Entry<K,V> entry = getEntry(key);

    return null == entry ? null : entry.getValue();
}
```

```java
final Entry<K,V> getEntry(Object key) {
    if (size == 0) {
        return null;
    }

    int hash = (key == null) ? 0 : hash(key);
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
```

**get()方法就很简单了，得到hash值，然后遍历，得到value值，返回。**

**就不做分析了，偷个懒。**



## 手写HashMap

```java
import java.util.Arrays;

/**
 * @authoer : zhw
 * @Date: 2020/2/23
 * @Description: java_study
 * @version: 1.0
 */
public class MyHashMap<K, V> {

    private static int size = 0;
    private Node<K, V> [] table;
    private static Integer CAPACITY = 8;

    public MyHashMap ( ) {
        this.table = new Node[CAPACITY];
    }



    public Object put( K key, V value){
        int hash = key.hashCode ();
        int index = hash%8;
        for (Node<K,V> node = table[index]; node !=null ; node = node.next) {
            if (node.key.equals ( key )){
                V oldvalue = node.value;
                node.value = value;
                return oldvalue;
            }
        }
        addNode ( key, value,index );

        return null;
    }

    private void addNode ( K key, V value, int index ) {
        Node node = new Node ( key, value, table[index] );
        table[index] = node;
        size++;
    }

    public Object get(K key){
        int hash = key.hashCode ();
        int index = hash%8;
        for (Node node = table[index]; node !=null ; node = node.next) {
            if (node.key.equals ( key )){
                return node.value;
            }
        }
        return null;
    }

    public int size(){
        return size;
    }

    @Override
    public String toString () {
        return "MyHashMap{" +
                "table=" + Arrays.toString ( table ) +
                '}';
    }

    public static void main ( String[] args ) {
        MyHashMap <String, String> myHashMap = new MyHashMap <> ( );

        for (int i = 0; i < 10; i++) {
            myHashMap.put ( i+"", i+"" );
        }

        System.out.println ( myHashMap );
        System.out.println ( "haha" );
    }
}


class Node<K , V >{
    public K key;
    public V value;
    public Node<K, V> next;

    public Node ( K key, V value, Node <K, V> next ) {
        this.key = key;
        this.value = value;
        this.next = next;
    }

    @Override
    public String toString () {
        return "Node{" +
                "key=" + key +
                ", value=" + value +
                ", next=" + next +
                '}';
    }

    public K getKey () {
        return key;
    }

    public Node <K, V> getNext () {
        return next;
    }

    public V getValue () {
        return value;
    }
}
```

> 运行结果

![](https://i.loli.net/2020/02/24/qAoY56JNMQUeLRx.png)



## 最后总结一波

1.7版本的HashMap还是有很大的问题的，在下一个版本1.8中会大有改进。

最开始的时候，Java 就说明了 HashMap 不应该在高并发情况下使用。

一般的学习者会认为这是因为它没有做并发处理，所以理所应当地产生大量错读错写，所以不建议使用。

没错，没一毛钱问题。

可如果 HashMap 只是这样，那**不叫做设计缺陷, 也不叫BUG** 。

没有增加并发处理的数据结构在高并发使用时出现错读错写不能叫问题，更别说设计缺陷了。

总的来说阅读JDK1.7源码，能提高我们阅读源码的能力，可以理解java开发者们的设计思路，所以还是很有意思的。

