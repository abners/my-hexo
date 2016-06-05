title: hashMap源码分析
date: 2016-05-31 17:21
tags:
- java

categories: 原创
comments: true
-----
前几天面试被问到了hashMap的底层实现原理，虽然之前有所了解，并且对源码大概看了一遍，回答的时候仍然不是太满意，今天写这篇文章来对HashMap的源码进行详细的分析。
<!--more-->
提到对HashMap的操作，无非就是put、get、初始化等操作。首先来分析下HashMap的put操作，废话不多讲，先看下put的源码：

```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
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
```

第一行首先判断存放键值对的Entry数组是否为空，为空的话对其进行初始化，对于inflateTable方法，我们稍后再对其进行分析，先继续往下看，接下来判断key是否为null，如果为null调用putForNullKey方法，由此我们可以看出
HashMap的key是允许为null的（HashTable的key是不允许为空的，感兴趣的同学可以去看下HashTable的实现），在继续往下分析之前先看下这个putForNullKey:

```java
/**
  * Offloaded version of put for null keys
  */
private V putForNullKey(V value) {
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    modCount++;
    addEntry(0, null, value, 0);
    return null;
}

```

从中可以HashMap是将key为null的元素放到了Entry数组的开头，接着遍历检查key是否为null，是则用新值替换掉旧值，否则调用addEntry添加到索引为0的Entry数组中。

下面我们接着看put方法，分析key不为null的处理过程：
#### 下面是获取key的hash值的过程：
```java
/**
     * Retrieve object hash code and applies a supplemental hash function to the
     * result hash, which defends against poor quality hash functions.  This is
     * critical because HashMap uses power-of-two length hash tables, that
     * otherwise encounter collisions for hashCodes that do not differ
     * in lower bits. Note: Null keys always map to hash 0, thus index 0.
     */
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
上面就是获取的key的hashCode的方法，其中用到了hashSeed，那么hashSeed是什么？为什么在此处要用到hashSeed呢？jdk的源码注释中是这么解释的：
hashSeed的是一个与当前hashhMap相关联的一个随机值，用于减少hash冲突，此算法加入了高位计算，防止低位不变，高位变化产生的hash冲突。
接下来就是根据计算出的hashCode获取当前key在Entry数组中的位置：
```java

 /**
     * Returns index for hash code h.
     */
    static int indexFor(int h, int length) {
        // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
        return h & (length-1);
    }

```
这里我们来解释下为什么要使用取模运算来计算位置，我们都知道HashMap是使用数组+链表来存储元素，因为链表的查找效率比较低，这样我们肯定希望HashMap的数组中的
元素尽量分布的均匀一些，这样就会尽量使数组的每一个位置上只有一个元素，这样我们在查找元素的时候就可以根据hashCode计算出的索引直接查找到该元素，而不用遍历
Entry链表，这样就会使查找效率大大提高。
接下来就是根据计算出的索引位置遍历数组对应位置的Entry链表，如果找到就替换掉旧值，否则将其添加到链表中。

至此我们整个put方法分析完毕。下一篇我们来分析一下HashMap的构造方法，并会讲解Hash在什么情况会进行reHash操作。

参考文章：[http://blog.csdn.net/top_code/article/details/17396961]http://blog.csdn.net/top_code/article/details/17396961




   
