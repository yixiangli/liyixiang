---
title: Java集合详解
toc: true
date: 2016-12-25 17:47:34
category: 
	- 技术贴
	- Java
	- 基础
tags: 
    - 集合
    - Map 
    - List 
    - Set
---

# 集合 Or 容器
java.util.*

## 结构
![集合结构图](/img/collection.jpg)

## 泛型
Java1.5引入了泛型，泛型允许我们为集合提供一个可以容纳的对象类型
解决了两个问题：
（1）对某集合定义泛型后，如果添加非泛型类型的任何元素，会在编译时报错，避免在运行期出现ClassCastException
（2）不需要使用显式转换和instanceOf操作符，代码更整洁


## 集合框架类


## Map

### HashMap多线程并发可能存在的问题
#### get死循环
多线程同时进行put操作后，都同时rehash操作，导致Entry List出现循环引用，get会出现死循环，导致CPU飙升，多线程都被Hang住

#### get数据丢失
看createEntry方法

```
void createEntry(int hash, K key, V value, int bucketIndex) {
        Entry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new Entry<>(hash, key, value, e);
        size++;
    }
```

该方法中请注意new Entry（）构造方法，如果在多线程场景下两个线程同时获取到了e实例，那么在赋值给table时有一个成功有一个丢失。

#### put非null元素后get出来的却是null
源码

### Enumeration和Iterator接口的区别
Enumeration的速度是Iterator的两倍，也使用更少的内存。Enumeration是非常基础的，也满足了基础的需要。但是，与Enumeration相比，Iterator更加安全，因为当一个集合正在被遍历的时候，它会阻止其它线程去修改集合。迭代器取代了Java集合框架中的Enumeration。迭代器允许调用者从集合中移除元素，而Enumeration不能做到

## 源码

### Objects
equals AND deepEquals
(1) deepEquals用于判定两个指定数组彼此是否深层相等，此方法适用于任意深度的嵌套数组。
(2) equals用于判定两个数组是否相等，如果两个数组以相同顺序包含相同元素，则返回true，否则返回false。
<!--more-->

## CopyOnWrite容器
### 基本概念
简称COW,是一种用于程序设计中的优化策略,即写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。


### 使用场景
CopyOnWrite并发容器用于读多写少的并发场景。


### 注意事项
1.减少扩容开销，预估容器大小。
2.尽量使用批量添加，减少容器复制次数。


### 缺点
1.内存占用问题
因为CopyOnWrite的写时复制机制，所以在进行写操作的时候，内存里会同时驻扎两个对象的内存，旧的对象和新写入的对象（注意:在复制的时候只是复制容器里的引用，只是在写的时候会创建新对象添加到新容器里，而旧容器的对象还在使用，所以有两份对象内存）。如果这些对象占用的内存比较大，比如说200M左右，那么再写入100M数据进去，内存就会占用300M，那么这个时候很有可能造成频繁的Yong GC和Full GC。
针对内存占用问题，可以通过压缩容器中的元素的方法来减少大对象的内存消耗，比如，如果元素全是10进制的数字，可以考虑把它压缩成36进制或64进制，或者不使用CopyOnWrite容器，而使用其他的并发容器，如ConcurrentHashMap。

2.数据一致性问题
数据一致性问题。CopyOnWrite容器只能保证数据的最终一致性，不能保证数据的实时一致性。所以如果你希望写入的的数据，马上能读到，请不要使用CopyOnWrite容器。


## IdentityHashMap


### 与HashMap的区别
- 等值比较
Identity使用 "==" 进行比较，普通map使用"equals"
- hashCode
identity使用System.identityHashCode,普通map使用重写的hashCode查找位置

因此IdentityHashMap可以允许key重复或者为null,这就引出了它的使用场景
### 使用场景
IdentityHashMap有其特殊用途，比如序列化或者深度复制。或者记录对象代理。
举个例子，jvm中的所有对象都是独一无二的，哪怕两个对象是同一个class的对象，而且两个对象的数据完全相同，对于jvm来说，他们也是完全不同的，如果要用一个map来记录这样jvm中的对象，你就需要用IdentityHashMap，而不能使用其他Map实现。
