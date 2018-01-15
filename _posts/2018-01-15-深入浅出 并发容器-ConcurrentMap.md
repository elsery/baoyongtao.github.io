---
layout: post
title: "深入浅出 并发容器-ConcurrentMap"
date: 2018-01-15 
description: "们从头开始设想。要将对象存放在一起，如何设计这个容器。目前只有两条路可以走，一种是采用分格技术，每一个对象存放于一个格子中，这样通过对格子的编号就能取到或者遍历对象；另一种技术就是采用串联的方式，将各个对象串联起来，这需要各个对象至少带有下一个对象的索引（或者指针）。显然第一种就是数组的概念，第二种就是链表的概念。所有的容器的实现其实都是基于这两种方式的，不管是数组还是链表，或者二者俱有。HashMap采用的就是数组的方式。"
categories: 并发
--- 

  

**本来想比较全面和深入的谈谈ConcurrentHashMap的，发现网上有很多对HashMap和ConcurrentHashMap分析的文章，因此本小节尽可能的分析其中的细节，少一点理论的东西，多谈谈内部设计的原理和思想。
要谈ConcurrentHashMap的构造，就不得不谈HashMap的构造，因此先从HashMap开始简单介绍。**


## HashMap原理

我们从头开始设想。要将对象存放在一起，如何设计这个容器。目前只有两条路可以走，一种是采用分格技术，每一个对象存放于一个格子中，这样通过对格子的编号就能取到或者遍历对象；另一种技术就是采用串联的方式，将各个对象串联起来，这需要各个对象至少带有下一个对象的索引（或者指针）。显然第一种就是数组的概念，第二种就是链表的概念。所有的容器的实现其实都是基于这两种方式的，不管是数组还是链表，或者二者俱有。HashMap采用的就是数组的方式。

有了存取对象的容器后还需要以下两个条件才能完成Map所需要的条件。

*   能够快速定位元素：Map的需求就是能够根据一个查询条件快速得到需要的结果，所以这个过程需要的就是尽可能的快。
*   能够自动扩充容量：显然对于容器而然，不需要人工的去控制容器的容量是最好的，这样对于外部使用者来说越少知道底部细节越好，不仅使用方便，也越安全。

首先条件1，快速定位元素。快速定位元素属于算法和数据结构的范畴，通常情况下哈希（Hash）算法是一种简单可行的算法。所谓**哈希算法**，是将任意长度的二进制值映射为固定长度的较小二进制值。常见的MD2,MD4,MD5，SHA-1等都属于Hash算法的范畴。具体的算法原理和介绍可以参考相应的算法和数据结构的书籍，但是这里特别提醒一句，由于将一个较大的集合映射到一个较小的集合上，所以必然就存在多个元素映射到同一个元素上的结果，这个叫“碰撞”，后面会用到此知识，暂且不表。

条件2，如果满足了条件1，一个元素映射到了某个位置，现在一旦扩充了容量，也就意味着元素映射的位置需要变化。因为对于Hash算法来说，调整了映射的小集合，那么原来映射的路径肯定就不复存在，那么就需要对现有重新计算映射路径，也就是所谓的rehash过程。

好了有了上面的理论知识后来看HashMap是如何实现的。

在HashMap中首先由一个对象数组table是不可避免的，修饰符transient只是表示序列号的时候不被存储而已。size描述的是Map中元素的大小，threshold描述的是达到指定元素个数后需要扩容，loadFactor是扩容因子(loadFactor>0)，也就是计算threshold的。那么元素的容量就是table.length，也就是数组的大小。换句话说，如果存取的元素大小达到了整个容量(table.length)的loadFactor倍（也就是table.length*loadFactor个），那么就需要扩充容量了。在HashMap中每次扩容就是将扩大数组的一倍，使数组大小为原来的两倍。

 [![HashMap数据结构](http://www.blogjava.net/images/blogjava_net/xylz/WindowsLiveWriter/JavaConcurrency17part2ConcurrentMap2_FF15/image_thumb.png "HashMap数据结构")](http://www.blogjava.net/images/blogjava_net/xylz/WindowsLiveWriter/JavaConcurrency17part2ConcurrentMap2_FF15/image_2.png)

然后接下来看如何将一个元素映射到数组table中。显然要映射的key是一个无尽的超大集合，而table是一个较小的有限集合，那么一种方式就是将key编码后的hashCode值取模映射到table上，这样看起来不错。但是在Java中采用了一种更高效的办法。由于与(&)是比取模(%)更高效的操作，因此Java中采用hash值与数组大小-1后取与来确定数组索引的。为什么这样做是更有效的？[参考资料7](http://www.javaeye.com/topic/539465)对这一块进行非常详细的分析，这篇文章的作者非常认真，也非常仔细的分析了里面包含的思想。

### indexFor片段

```java
static int indexFor(int h, int length) {
    return h & (length-1);
}
```

前面说明，既然是大集合映射到小集合上，那么就必然存在“碰撞”，也就是不同的key映射到了相同的元素上。那么HashMap是怎么解决这个问题的？

在HashMap中采用了下面方式，解决了此问题。

1.  同一个索引的数组元素组成一个链表，查找允许时循环链表找到需要的元素。
2.  尽可能的将元素均匀的分布在数组上。

[![Map.Entry结构](http://www.blogjava.net/images/blogjava_net/xylz/WindowsLiveWriter/JavaConcurrency17part2ConcurrentMap2_FF15/image_thumb_1.png "Map.Entry结构")](http://www.blogjava.net/images/blogjava_net/xylz/WindowsLiveWriter/JavaConcurrency17part2ConcurrentMap2_FF15/image_4.png)对于问题1，HashMap采用了上图的一种数据结构。table中每一个元素是一个Map.Entry，其中Entry包含了四个数据，key,value,hash,next。key和value是存储的数据；hash是元素key的Hash后的表现形式（最终要映射到数组上），这里链表上所有元素的hash经过清单1 的indexFor后将得到相同的数组索引；next是指向下一个元素的索引，同一个链表上的元素就是通过next串联起来的。

再来看问题2 尽可能的将元素均匀的分布在数组上这个问题是怎么解决的。首先清单2 是将key的hashCode经过一系列的变换，使之更符合小数据集合的散列模型。

### hashCode的二次散列

```java
static int hash(int h) {
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

至于清单2 为什么这样散列我没有找到依据，也没有什么好的参考资料。[参考资料1](http://www.javaeye.com/topic/709945) 分析了此过程，认为是一种比较有效的方式，有兴趣的可以研究下。

第二点就是在清单1 的描述中，尽可能的与数组的长度减1的数与操作，使之分布均匀。这在[参考资料7](http://www.javaeye.com/topic/539465) 中有介绍。

第三点就是构造数组时数组的长度是2的倍数。清单3 反映了这个过程。为什么要是2的倍数？在[参考资料7](http://www.javaeye.com/topic/539465) 中分析说是使元素尽可能的分布均匀。

### HashMap 构造数组

```java
// Find a power of 2 >= initialCapacity
int capacity = 1;
while (capacity < initialCapacity)
    capacity <<= 1;

this.loadFactor = loadFactor;
threshold = (int)(capacity * loadFactor);
table = new Entry[capacity];
```

另外loadFactor的默认值0.75和capacity的默认值16是经过大量的统计分析得出的，很久以前我见过相关的数据分析，现在找不到了，有兴趣的可以查询相关资料。这里不再叙述了。

有了上述原理后再来分析HashMap的各种方法就不是什么问题的。

### HashMap的get操作

```java
public V get(Object key) {
    if (key == null)
        return getForNullKey();
    int hash = hash(key.hashCode());
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
            return e.value;
    }
    return null;
}
```

清单4 描述的是HashMap的get操作，在这个操作中首先判断key是否为空，因为为空的话总是映射到table的第0个元素上（可以看上面的清单2和清单1）。然后就需要查找table的索引。一旦找到对应的Map.Entry元素后就开始遍历此链表。由于不同的hash可能映射到同一个table[index]上，而相同的key却同时映射到相同的hash上，所以一个key和Entry对应的条件就是hash(key)==e.hash 并且key.equals(e.key)。从这里我们看到，Object.hashCode()只是为了将相同的元素映射到相同的链表上（Map.Entry)，而Object.equals()才是比较两个元素是否相同的关键！这就是为什么总是成对覆盖hashCode()和equals()的原因。

###  HashMap的put操作

```java
public V put(K key, V value) {
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key.hashCode());
    int i = indexFor(hash, table.length);
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
    addEntry(hash, key, value, i);
    return null;
}
void addEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
        if (size++ >= threshold)
            resize(2 * table.length);
}
```
清单5 描述的是HashMap的put操作。对比get操作，可以发现，put实际上是先查找，一旦找到key对应的Entry就直接修改Entry的value值，否则就增加一个元素。增加的元素是在链表的头部，也就是占据table中的元素，如果table中对应索引原来有元素的话就将整个链表添加到新增加的元素的后面。也就是说新增加的元素再次查找的话是优于在它之前添加的同一个链表上的元素。这里涉及到就是扩容，也就是一旦元素的个数达到了扩容因子规定的数量(threhold=table.length*loadFactor)，就将数组扩大一倍。

### HashMap扩容过程

```java
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
```

原文出处：[http://www.blogjava.net/xylz/archive/2010/07/20/326584.html](http://www.blogjava.net/xylz/archive/2010/07/20/326584.html "http://www.blogjava.net/xylz/archive/2010/07/20/326584.html")