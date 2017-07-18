---
title: Redis内存管理
toc: true
date: 2017-05-05 14:53:15
category: 
	- 技术贴
	- 中间件
	- 缓存系列
	- Redis
tags: 
    - cache
---

# 内存管理
说起Redis，就会联想到他的兄弟Memcache，而Memcache的内存模型之前的博文中已经介绍，有兴趣的可以与此文进行对比，那么Redis的内存模型是怎样的呢？

## 机制
在Redis中，并不是所有的数据都一直存储在内存中，这是和Memcache最大的区别。那么当物理内存很紧张的时候，Redis会触发swap的操作，根据算法计算出哪些key对应的value需要swap到磁盘，然后对value进行持久化，再删除内存中的value数据（注意，Redis会缓存所有的key的信息，也不会进行swap），因此机器的内存至少要保证能存放下全部的key。

<!--more-->
## 内存模型
Redis的内存模型相对Memcache要简单得多，首先在分配一块内存之后，会将这块内存的大小存入内存块的头部。

![Redis内存模型](/img/Redis-module.jpg)

All_use可以看作是redis调用malloc后返回的指针，redis将内存块的大小size存入头部，并返回内存使用的真实长度Real_use。当需要释放内存，通过Real_use，程序可以算出All_use的值，然后统一释放。