title: hashMap源码分析
date: 2016-05-31 17:21
tags:
- java

categories: 原创
comments: true
-----
前几天面试被问到了hashMap的底层实现原理，虽然之前有所了解，并且对源码大概看了一遍，回答的时候仍然不是太满意，今天写这篇文章来对HashMap的源码进行详细的分析。

提到对HashMap的操作，无非就是put、get、初始化等操作。首先来分析下HashMap的put操作，废话不多讲，先看源码：

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

从中可以HashMap是将key为null的元素放到了Entry数组的开头，
   
