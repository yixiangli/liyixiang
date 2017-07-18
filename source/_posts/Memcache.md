---
title: Memcache
toc: true
date: 2017-04-11 14:44:31
category: 
	- 技术贴
	- 中间件
	- 缓存系列
	- Memcache
tags: 
    - cache
---

# Memcache
一种分布式缓存，不啰嗦介绍，官方网站为http://memcached.org/

## 一些概念
Memcache : 项目名称
memcached : 主文件程序名

## 结构
HashMap，基于内存的分布式key-value，但是注意，Memcache本身完全不具备分布式功能，首先看一张图，一个网友画的，挺不错，直接贴过来了。
![Memcache模型](/img/memcache.jpg)

<!--more-->

从图上得知，实现分布式，完全依赖于客户端程序调度，引出Memcache读写流程

### 读写流程
1、外部应用输入需要写缓存的数据
2、API将Key输入路由算法模块，路由算法根据Key和MemCache集群服务器列表得到一台服务器编号
3、由服务器编号得到MemCache及其的ip地址和端口号
4、API调用通信模块和指定编号的服务器通信，将数据写入该服务器，完成一次分布式缓存的写操作

读缓存和写缓存一样，只要使用相同的路由算法和服务器列表，只要应用程序查询的是相同的Key，MemCache客户端总是访问相同的客户端去读取数据，只要服务器中还缓存着该数据，就能保证缓存命中。

### 路由算法
刚才介绍了Memcache的读写流程，提到了客户端根据Key值，通过路由算法找到相应的服务，因此Memcache路由算法重中之重。对于路由算法相信大家并不陌生，在工作中遇到比如分表，大表拆小表，都是需要相应的路由算法，而场景不同，路由算法也不尽相同，当然最常见的肯定还是余数Hash

#### 余数Hash
根据对象的hashCode，做求余计算，余数相同的划分为同一区域。由于HashCode随机性比较强，所以使用余数Hash路由算法就可以保证缓存数据在整个MemCache服务器集群中有比较均衡的分布。乍一看这个方案还是很完美的，但是实际上忽略了一个重要的问题，那就是伸缩性。我提一个问题，随着业务的扩张Memcache服务集群不够用了，需要扩容，那么对于缓存数据来说就是一场灾难。文字描述的不够详细，不妨举个例子，原来有HashCode为0~19的20个数据，那么：

![余数Hash伸缩例子](/img/hash.jpg)

如果扩容到20+的台数，只有前三个HashCode对应的Key是命中的，也就是15%，代表着绝大多数的数据都进行了迁移，如果这是个为防止数据库压力过大而设置的缓存层，那么就面临着大量报警吧。由这个case可知，缓存数据命中率随着服务扩容而大大下降，对于一套分布式缓存服务是令人难以接受的，那么有没有缓存数据命中率随着服务扩容而上升的算法呢，还真有，往下看。

#### 一致性Hash
数据结构和算法不分家，先介绍下一致性Hash算法采用的数据结构

![Hash环结构](/img/hash-ring.png)

加一个节点

![Hash环结构加节点](/img/hash-ring-addNode.png)

具体算法过程：先构造一个长度为2^32的整数环（这个环被称作一致性Hash环）根据节点名称的Hash值（其分布范围为[0,2^32 - 1]）将缓存服务器阶段设置在这个Hash环上。然后根据需要缓存的数据的Key值计算得到Hash值（其分布范围也同样为[0,2^32 - 1]），然后在Hash环上顺时针查找举例这个KEY的hash值最近的缓存服务器节点，完成KEY到服务器的Hash映射查找。

优化策略：将每台物理服务器虚拟为一组虚拟缓存服务器，将虚拟服务器的Hash值放置在Hash环上，key在换上先找到虚拟服务器节点，再得到物理服务器的信息。

一台物理服务器设置多少个虚拟服务器节点合适呢？经验值：150。

### 内存结构模型
先看一张图：
![Memcache内存模式](/img/memcache-struct.jpg)

#### 结构模型
从上图看，主要Memcache内存空间主要涉及slab_class、slab、page、chunk四个概念，并且互为包含的关系，下面具体介绍。

** slab ** 
MemCache将内存空间分为一组slab

** page ** 
每个slab分成若干个page,每个page默认是1M,所以如果slab有100M，那么就会有100个page

** chunk ** 
每个page分成若干组chunk,chunk是真正存放数据的地方，同一个slab里面的chunk的大小是固定的,也决定了slab_class

** slab_class ** 
相同大小chunk的slab被组织在一起，称为slab_class

#### 内存分配原理
介绍完了结构模型，接下来就是探究一下内存分配的原理，总结几个规则
1.一个value存放的位置由它本身的大小决定
2.value总是会被存放到与chunk大小最接近的一个slab中
3.申请内存是以page为单位的，所以在放入第一个数据的时候，无论大小为多少，都会有1M大小的page被分配给该slab。
4.slab内存满后会通过LRU算法进行回收（前提是必须开启-M）,且MemCache的LRU算法不是针对全局的，只是针对slab的
5.MemCache存放的value大小是有限制的，最大1M,原因就是page为1M

#### 特性与限制
这一段完全摘自网络，大家随意看看就好
1.MemCache中可以保存的item数据量是没有限制的，只要内存足够
2.MemCache单进程在32位机中最大使用内存为2G，64位机则没有限制
3.Key最大为250个字节，超过该长度无法存储
4.单个item最大数据是1MB，超过1MB的数据不予存储
5.MemCache服务端是不安全的，比如已知某个MemCache节点，可以直接telnet过去，并通过flush_all让已经存在的键值对立即失效
6.不能够遍历MemCache中所有的item，因为这个操作的速度相对缓慢且会阻塞其他的操作
7.MemCache的高性能源自于两阶段哈希结构：第一阶段在客户端，通过Hash算法根据Key值算出一个节点；第二阶段在服务端，通过一个内部的Hash算法，查找真正的item并返回给客户端。从实现的角度看，MemCache是一个非阻塞的、基于事件的服务器程序
8.MemCache设置添加某一个Key值的时候，传入expiry为0表示这个Key值永久有效，这个Key值也会在30天之后失效，见memcache.c的源代码：

```
#define REALTIME_MAXDELTA 60*60*24*30
static rel_time_t realtime(const time_t exptime) {
       if (exptime == 0) return 0;
       if (exptime > REALTIME_MAXDELTA) {                       
              if (exptime <= process_started)                         
                      return (rel_time_t)1;                                 
              return (rel_time_t)(exptime - process_started);  
       } else {                                                                  
              return (rel_time_t)(exptime + current_time);     
       }
}
```
这个失效的时间是memcache源码里面写的，开发者没有办法改变MemCache的Key值失效时间为30天这个限制

### Memcache指令
telnet  	连接Memcache
get			返回Key对应的Value值
add 		添加一个Key值，没有则添加成功并提示STORED，有则失败并提示NOT_STORED
set 	 	无条件地设置一个Key值，没有就增加，有就覆盖，操作成功提示STORED
replace 	按照相应的Key值替换数据，如果Key值不存在则会操作失败 
stats		返回MemCache通用统计信息（下面有详细解读）
stats items	返回各个slab中item的数目和最老的item的年龄（最后一次访问距离现在的秒数）
stats slabs	返回MemCache运行期间创建的每个slab的信息（下面有详细解读）
version		返回当前MemCache版本号
flush_all	清空所有键值，但不会删除items，所以此时MemCache依旧占用内存
quit		关闭连接


### 参数设置
-f
相邻slab内的chunk基本以1.25为比例进行增长，MemCache启动时可以用-f指定这个比例
-M
如果MemCache启动没有追加-M（禁止LRU，这种情况下slab内存不够会报Out Of Memory错误)