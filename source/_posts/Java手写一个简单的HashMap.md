---
title: Java手写一个简单的HashMap
date: 2021-09-11 11:56:50
tags:
  - HashMap
  - Java
  - 手写
categories:
  - 源码剖析和实现
---



#### 1.背景介绍

HashMap在Java中是常用的数据结构之一。HashMap 是一个散列表，它存储的内容是键值对(key-value)映射，并具有很快的访问速度。在JDK1.7中，HashMap是基于“数组+链表”实现的，而在JDK1.8以后，HashMap在底层实现中加入了红黑树用于提升查找速率。

![](/img/hashmap.png)
	<center><font face="宋体" size=3 color="black">图源网络，侵删</font></center>
<br>

在JDK1.8中，当链表的长度大于阈值8时，这时这个链表将会转化成红黑树以提升查找效率。为什么阈值是8呢？请读者不妨思考一下这个问题，在文章末尾笔者将给出原因。（提示：想一想在红黑树和链表中查找一个元素的复杂度）

好了，关于HashMap就简单介绍到这里，接下来我们关注于自己实现一个HashMap—MyHashMap。



#### 2.目标

在本次实现中，我们的目标如下：

* 实现put(k, v)，该方法返回V类型的元素，这里返回为空即可。
* 实现get(k)，该方法返回这个建对应的值v。
* 实现remove(k)，该方法将这个键对应的键值对删除，并返回对应的值v，如果不存在对应的键，返回空。
* 实现size()，该方法返回HashMap中的键值对数目。

明确我们的目标后，就可以关注于具体实现了。



#### 3.手写HashMap

##### 3.1定义MyMap接口

这个接口定义了我们需要实现的具体行为。

```java
public interface MyMap <K, V>{
    V get(K k);

    V put(K k, V v);

    int size();

    V remove(K k);

    boolean isEmpty();
}
```

接下来我们要根据这个接口的定义完成我们的MyHashMap类去实现接口中定义的行为。



##### 3.2定义链表节点

因为HashMap中存在着链表，所以我们也需要实现一个链表。我们以内部类的形式定义这样一个节点Entry，Entry类保存了"K-V"数据，next字段表明它可能会是一个链表节点。

参考形式如下：

```java
public class MyHashMap <K,V> implements MyMap<K, V> {
    //定义内部类Entry作为链表节点
     class Entry<K, V>{
         K k;
         V v;
         Entry<K,V> next;

         public Entry(K k, V v){
             this.k = k;
             this.v = v;
         }
     }
}

```



##### 3.2定义成员变量

这里参照HashMap设置一个默认的容量capacity和默认的加载因子loadFactor，table就是底层数组，另外，考虑到size方法的实现，这里肯定还需要一个成员变量size用于表示HashMap的大小。

```java
//定义成员变量
final static int DEFAULT_CAPACITY = 16;
final static float DEFAULT_LOAD_FACTOR = 0.75f;

private int capacity;
private float loadFactor;
private int size = 0;

Entry<K,V>[] table;
```



##### 3.3实现构造方法

```java
public MyHashMap(){
   this(DEFAULT_CAPACITY, DEFAULT_LOAD_FACTOR);
}

public MyHashMap(int capacity, float loadFactor){
   this.capacity = upperMinPowerOf2(capacity);     //获取为2的幂次方的容量大小
    this.loadFactor = loadFactor;                  //加载因子，用于扩容,本次实现中尚未用到该字段
    this.table = new Entry[capacity];
}
```

这里的`upperMinPowerOf2`的作用是获取大于capacity的最小的2次幂。在HashMap中，开发者采用了更精妙的位运算的方式完成了这个功能，效率比这种方式要更高。capacity要求为2次幂是为了方便HashMap在数组扩容时能够更好地对已存在的元素进行重新哈希。

```java
private static int upperMinPowerOf2(int n){
    int power = 1;
    while(power <= n){
        power *= 2;
    }
    return power;
}
```



##### 3.4实现put方法

put方法将传入的键值对封装成一个Entry<K, V>对象进行存入，存入数组中的index我们通过对键哈希取模得到，这样能减少哈希冲突，也就能减少链表的数目，能提高HashMap的查找效率。

假如数组中没有元素，那么直接将该Entry放入对应位置即可。

假如数组中已经存在元素，我们需要遍历这个链表，检查是否存在有相等的key（这里用equals方法会更好），如果存在相等key，那么用新值替换旧值，然后返回；如果不存在，那么就使用头插法插入链表。

记得对size做正确的操作，保持记录的元素个数正确。

```java
@Override
public V put(K k, V v) {
    // 通过hashcode散列获取索引
    int index = k.hashCode() % table.length;
    Entry<K,V> current = table[index];
    //判断是否已经存在元素
    if (current!=null){
        // 遍历链表是否有相等key, 有则替换且返回旧值
        while (current!=null){
            if(current.k == k){
                V oldValue = current.v;
                current.v = v;
                return oldValue;
            }
            current = current.next;
        }
        // 没有则使用头插法
        table[index] = new Entry<K, V>(k,v, table[index]);
        size++;
        return null;
    }
    //不存在元素直接放入即可
    table[index] = new Entry<K, V>(k,v,null);
    size++;
    return null;
}
```



##### 3.5实现get方法

get方法通过键k获取对应的值v，根据put中存放的索引位置，get方法中也是一样的计算方法。

遍历链表，如果检查存在键为k的键值对Entry，那么就返回对应的值v，没有则返回空。

```java
@Override
public V get(K k) {
     int index = k.hashCode() % table.length;
     Entry<K, V> current = table[index];
     //遍历链表
    while (current!=null){
        if(current.k == k)
            return current.v;
        current = current.next;
    }
    //不存在则返回空
    return null;
}
```



##### 3.6实现remove方法

remove方法通过对应的键k删去在HashMap中对应的键值对Entry，同样是遍历链表，不同的是我们需要一个前置节点pre保存当前节点的上一个节点信息，这样才能正确地删除节点。删除成功则将size-1。

如果不存在这样一个节点，返回空。

```java
@Override
public V remove(K k) {
    int index = k.hashCode() % table.length;
    V result = null;
    Entry<K, V> current = table[index];
    //遍历链表
    Entry<K, V> pre = null;

    while(current!=null){
        if(current.k == k){
            result = current.v;
            size--;
            if (pre!=null){
                pre.next = current.next;
            }else {
                table[index] = current.next;
            }

            return result;
        }
        //向下遍历
        pre = current;
        current = current.next;
    }

    return null;
}
```



##### 3.7实现size方法与isEmpty方法

比较简单的实现。

```java
@Override
public int size() {
    return size;
}

@Override
public boolean isEmpty() {
	return size==0;
}
```



#### 4.测试MyHashMap

测试代码如下：

```java
public class hashMap_test {
    public static void main(String[] args){
        MyHashMap<Integer, Integer> hashMap = new MyHashMap<>();
        hashMap.put(1,101);
        hashMap.put(2,202);
        hashMap.put(3,303);
        hashMap.put(1,111);

        int[] keys = new int[]{1,2,3};
        System.out.println("hashMap size:"+hashMap.size());
        for(int i=0;i< keys.length;i++){
            System.out.println(keys[i]+": "+hashMap.get(keys[i]));
        }

        hashMap.remove(1);
        hashMap.remove(3);
        System.out.println("hashMap size:"+hashMap.size());
        System.out.println(1+": "+hashMap.get(1));
        System.out.println(3+": "+hashMap.get(3));
        System.out.println(2+": "+hashMap.get(2));  //当然这里其实应该报错的
    }
}
```



#### 5.总结

本文简单地实现了一个HashMap，实现了Java中HashMap的put、get、remove、size、isEmpty等方法。但其实还有一些工作可以做，比如对HashMap进行扩容，当HashMap中元素过多的时候，我们需要将HashMap扩容以提高其查找速率，其实也就是减少HashMap中链表的数目，还有就是对于空值null的支持，其实HashMap是允许key为null的，当然，这一点不算太难。

对于我自己来说，原本我以为实现HashMap会是一件很困难的事情，所以我迟迟没有自己动手写一个，但是写了以后发现也就是这样，所以我们大家一定要多动手实践，也不要害怕困难，很多时候，其实是我们自己吓住了自己，导致没有去完成本可以完成的事情。



关于文章开头的那个问题：**为什么当链表中的元素超过8个的时候需要将链表转换成红黑树？**

答案如下：

> 在链表中查找时，根据next引用依次比较各个节点的key，长度为n的链表节点平均比较次数为n/2
>
> 在红黑树中查找时，由于红黑树的特性，节点数为n的红黑树平均比较次数为log(n)

链表长度超过8时树化（TREEIFY），正是因为n=8，就是log(n) < n/2的阈值。而n<6时，log(n) > n/2，红黑树解除树化（UNTREEIFY）。

如果文章内容存在不当之处，请各位读者能不吝赐教，笔者欢迎至极。

<br>
<br>

#### 参考资料：

[手写一个简单的HashMap - 周周zzz - 博客园 (cnblogs.com)](https://www.cnblogs.com/2511zzZ/p/12770864.html)

[纯手写实现HashMap - Java丨Mr.Chen - 博客园 (cnblogs.com)](https://www.cnblogs.com/chenfei-java/p/10674341.html)

[JavaGuide (gitee.io)](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/HashMap(JDK1.8)源码+底层数据结构分析?id=hashmap-源码分析)