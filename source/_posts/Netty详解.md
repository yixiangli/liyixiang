---
title: Netty详解
toc: true
date: 2017-05-08 16:30:18
category: 
	- 技术贴
	- Java
	- 框架
tags: 
    - netty
---

# Netty

## 基础组件
**Channel:** 在NIO模型中负责非阻塞I/O操作的网络I/O操作类，但并非原生NIO的Channel，自己重新实现，包含了JDK的SocketChannel与ServerSocketChannel,为使用者提供了统一的视图。

**Unsafe:**Channel内部辅助类，不对外暴露，实际的I/O读写操作，包括将Channel注册到多路复用器上，并只提供给Channel使用。

**ChannelPipeline:**Channel的过滤器，作为ChannelHandler的容器，负责ChannelHandler的管理和事件拦截与调度。线程安全。

**ChannelHandler:**对I/O事件进行拦截或处理。线程不安全，需要自己保证线程安全。

**Future:**用于获取异步操作的结果

**Promise:**对Future进行扩展，为其增加I/O写操作功能

<!--more-->
## 事件机制
在Netty中，ChannelPipeline被分为上行和下行，先来看一张图，从 Netty 官网上拷贝的一个图示:

![Netty事件处理](/img/netty_bound.jpg)

## Reactor模型
**单线程模型：**
一个线程处理连接，接收数据，编解码等全流程任务

**多线程模型：**
一个专门的Accept线程负责接收tcp连接，转发给后端NIO线程去处理编解码任务

**主从多线程模型：**
将Accept线程池化，一组线程负责接收tcp连接，后端同样是一组NIO线程去处理编解码任务

Netty的线程模型是可配置的，根据不同场景配置不同的线程池数来选择最合适的Reactor模型，另外采用局部无锁化设计，I/O线程内部进行串行操作，避免多线程竞争。

### 推荐的线程数量公式
公式一：线程数 ＝ （线程总时间／瓶颈资源时间）＊瓶颈资源线程的并行数
公式二：QPS = 1000/线程总时间＊线程数

**解释几个名词：**
线程总时间:处理一次请求线程需要消耗的总时间
瓶颈资源时间:逻辑中最耗时的处理时间（木桶效应）
瓶颈资源线程的并行数:程序中开启的如数据库／MQ等中间件的最大连接数（如果该组件是瓶颈，那么最大连接数就是瓶颈资源并行数）

### selector空轮询
这个bug臭名昭著，实际上是linux epoll的bug，在Nio中具体触发是Selector本次轮询结果为空，也没有wakeup操作或是新的消息需要处理，则说明是个空轮询，就会触发bug,导致I/O线程一直处于100%

Netty修复策略：
（1）对Selector的select操作周期进行统计
（2）每完成一次空的select操作进行一次计数
（3）在某个周期（如100ms）内如果连续发生N次空轮询，说明触发NIO epoll()死循环bug
