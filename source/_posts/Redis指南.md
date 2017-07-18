---
title: Redis指南
toc: true
date: 2016-12-25 17:53:23
category: 
	- 技术贴
	- 中间件
	- 缓存系列
	- Redis
tags: 
    - cache
---

# 概述
废话不多说，官方文档在此
http://doc.redisfans.com

---
## 数据结构
**使用者角度**
- string
- list
- hash
- set
- sorted set

<!--more-->
**内部实现角度**
- dict
- sds
- ziplist
- quicklist
- skiplist

数据结构的相关文档
https://redis.io/topics/data-types-intro

### String(字符串)
String作为一种很常用的缓存类型被各种缓存中间件支持，而且作为Redis，不仅支持String,还支持多种数据结构的缓存类型，这也是Redis比memcached强大的地方之一。

#### 常用操作
String 存入字符类型
        set name yixiangli  设置name = yixiangli 存储
        get name                获取设置好的name的值
        setnx name yixiangli 设置name键值为yixiangli 如果存在,则返回0 不存在返回1
        mset name yixiangli  age 23 salary 100000 设置多个键值对 (全成功,全失败)
        msetnx name xiaoyu age 23 hoby sleep 如果设置多个键值对中有存在则返回失败
        mget name age salary 获取多个键的值
        getset name xiaoyu 获取name的值,并设置新的值为xiaoyu
        setrange name 4 zhu 将键name 4个字符后面的进行替换 结果为xiaozhu
        getrange name 4 6 获取键name的值 结果为zhu
        append name .com 给键name追加.com,结果为xiaozhu.com 
        incr age 设置每个值自增 返回结果为24
        incrby age 6 给age加上6 如果是负数则减
        decr 与incr相反
        decrby 与decrby相反
        strlen 返回键对应的值得字符长度

###  hash(Hash类型)
Hash 方便存对象,键值对 
        hset user:001 name yixiangli   设置哈希表名user:001 key:name value:yixiangli
        hsetnx user name yixiangli    设置哈希表名字中的name,若已存在,则设置不成功
        hget user:001 name 获取hash表名为user001的键值对
        hmset user:003 name yixiangli age 23 批量设置
        hmget user:003 name age 批量获取user:003的值
        hincrby user:003 age 3     给hash表的age值加上3
        hexists user:003 name 判断hash表中式否存在name的键
        hlen user:003 返回hash表的所有的字段的数目
        hkeys user:003 返回hash表的所有字段
        hvals user:003 返回hash表中所有的值
        hgetall user:003 返回所有的字段和值
        hdel user:003 name 对hash的name的值和键删除

###  List(双向链表)
栈:先进后出 队列:先进先出
    lpush 从头压入
        lpush list1 "world"
        lpush list1 "hello"
        lrange list1 0 -1 把链表中的数据从头到尾全部取出
        结果： hello world
    rpush 从尾部压入
        rpush list2 "world" 
        rpush list2 "hello" 
        lrange list2 0 -1
        结果： world hello
    linsert 插入数据
        rpush list3 yixiangli
        rpush list3 xiaoyu
        lrange list3 0 -1
        linsert list3 before xiaoyu love
        lrange list3 0 -1
        linsert list3 after yixiangli love
        lrange list3 0 -1
    lset 替换
        rpush list5 yixiangli
        rpush list5 xiaoyu
        lset list5 0 "demo"
    lrem 删除list表中的数据
        rpush list6 yixiang1
        rpush list6 yixiang2
        rpush list6 yixiang3
        rpush list6 yixiang4
        rpush list6 yixiang5
        lrem list6 1 yixiang1
        删除list6 中值为yixiang1的值
     ltrim 保留范围
        lpush list7 yixiang1
        lpush list7 yixiang2
        lpush list7 yixiang3
        lpush list7 yixiang4
        lpush list7 yixiang5
        ltrim list7 1 2 (1 2 为保留的范围)
        lpush list7 yixiang2
        lpush list7 yixiang3    
     lpop 从链表的头部弹出一个元素
        lpush list8 yixiang1
        lpush list8 yixiang2
        lpush list8 yixiang3
        lpop list8 
        lpush list8 yixiang2
        lpush list8 yixiang3
     rpop 从链表的尾部弹出一个元素
        lpush list8 yixiang1
        lpush list8 yixiang2
        lpush list8 yixiang3
        rpop list8 
        lpush list8 yixiang1
        lpush list8 yixiang2
     rpoplpush 从一个链表弹出,在从头部压入到另一个链表
        List demo1 
        Demo1A
        Demo1B
        Demo1C
        List demo2
        Demo2A
        Demo2B
        Demo2C
        命令：rpoplpush demo1 demo2       
        结果:
        List demo1 
        Demo1A
        Demo1B
        List demo2
        Demo1C
        Demo2A
        Demo2B
        Demo2C
     lindex 返回一个list小标的索引值
        List11
        one
        two
        命令： lindex list11 1(list小标)
        two
        lindex list11 0
        one
     llen 返回这个链表的元素的长度


###  Set(集合)   
   sadd 向集合中插入一条数据
      sadd myset1 yixiang
      srem 删除集合中的一个元素
   srem myset1 yixiang
      smembers 查看集合中的元素
      smembers myset1
      spop 从集合随机弹出一个元素,返回键值
      sdiff 两个集合的差集 返回两个集合不一样的,根据第一个集合为标准
   setdemo1
      one
      two
   setdemo2
      one 
      three    
   sdiff setdemo1 setdemo2
      two(与setdemo2不一样)
   sdiff setdemo2 setdeo1
      three(与setdemo1 不一样)
   sdiffstroe 将两个差集存储到另外一个集合 
   sdiffstore setdemo1 setdemo2 setdemo3
   将setdemo1 setdemo2 的差集放到 setdemo3中
   sinter 将两个集合的交集
   sinterstore 将两个集合的交集存储到另外一个集合中
   sunion 将两个集合并集
   sunionstore 将两个集合并集并存储到另外一个集合中
   smove 将以个集合中的元素移动到另外一个集合中
   eg smove myset1 mysetA two mysetB 集合中的two元素移动到mysetB中
   scard 查看集合中元素的个数
   scard myset1查看myset12元素的个数
   sismember 判断是否是集合中的元素
   sismember myset13 yixiang 判断yixiang是否在myset13中的元素
   srandmember myset14 随机取出myset1 中的元素

###  Sorted set(有序集合)
   zadd 添加到有序集合中区
   zadd myzsent 1 yixiang1
   zadd myzsent 2 yixiang2
   zadd myzsent 3 yixiang3
        zadd myzsent 4 yixiang4
        zrange myzsent 0 -1 withscores 
        zrem 删除有序集合中的元素
        zrem myzsent yixiang1 删除myzsent集合中的yixiang1   
        zincrby myzsent yixiang1 3将myzsent luown1的序号更改为4
        如果没有,就创建他   
        zrank 找到myzsent 对应值得索引
        zrevrank 反过来去索引
        zrangebyscore 返回集合中指定的元素
        zrangebyscore mysetdeom 2 5 withscores
        返回mysetdemo中2-5中的元素
        zcount 返回指定空间的数量
        zcount myset 2  4 返回2 4中的元素个数
        zcard 返回集合中所有元素的个数
        zremrangbyrank 删除集合中指定区间的元素,并将索引进行排序
        zremrangbyscore 删除集合中指定元素,按循序进行排序

### 常用操作命令
   key-values
    1 keys * 匹配键所有的键. 模糊匹配 keys my* 取出所有以my开头的键
    2 exists 判断是否键 exists name判断是否有name这个键是否存在
    3 del 删除键 del name 删除name的键
    4 expire 设置过期时间 expire key time 
    5 ttl key 查看键的过期时间
    6 select database 选择数据库
    7 move key dababase1 讲key移动dao database1中的数据库中
    8 persist 取消键的过期时间 
    9 randomkey 随机返回一个键的值
    10 rename 重命名一个键
    11 type key 判断key的数据类型

   Server
    1 ping ping我们的主机能否链接 链接是否存活
    2 echo 命令 echo demo直接输出
    3 select 选择数据库 select 0-16个数据库
    4 quit exit 退出链接
    5 dbsize 返回数据库的键的个数
    6 info 返回服务器相关信息
    7 config get 返回服务配置信息
    8 flush db 清空数据库
    9 flushall 删除所有数据库中所有的键

## 高级应用
1 在配置文件里面设置 requirepass password
2 进入后 auth 密码 进行授权 方法二: 登入或在后面加上 –a 加上密码
3 主从复制:
        One: 一个master服务器可以拥有多个slave
        Two: 一个salve可以有多个master 并且还可以与其他的salve相连接
    配置salve
        打开salveof注释 并添加主机的ip以及端口
        主机加了密码的时候还需要配置masterauth 密码
4 redis 的事务处理
    输入:multi 打开一个上下文
        set age 10
        set age 144   
    上面的全部放入队列最后执行
        exec 
    最后age为144
    回滚
        discard    
    watch 监视键的命令
        
5 Redis的持久化
    方式一:  snapshotting (快照)将内存的数据写入到文件中 save 500 32 500秒内有32个键发生变化则发起快照到文件中
    方式二: append only file 将没次写修改的命令保存到文件中
    配置:打开append only
        Appendfsync yes
        Appendfsync always 每次都写入
        Appendfsync everysec    每个一秒写入
        Appendfsync no    不写入
6 发布和订阅消息
    订阅:
    　　 subscribe tv1 tv2 订阅了两个频道
    发布:
        publish tv1 yixiangli
    注:publish tv1的信息 订阅的信息都可以收到
7 虚拟内存
    方式一:暂时把不使用的数据放到硬盘里面
    方式二:可以把数据分割到其他的slave数据服务器中
    　
    　
## 常见问题
1. redis.clients.jedis.exceptions.JedisConnectionException: Could not get a resource from the pool
原因：连接不上Redis


### 内部结构实现
#### dict
dict是一个用于维护key和value映射关系的数据结构，与很多语言中的Map或dictionary类似。在Redis中，dict是一个基于哈希表的算法，而Redis的dict实现最显著的一个特点，就在于它的重哈希。它采用了一种称为增量式重哈希（incremental rehashing）的方法

##### 增量式重哈希
在需要扩展内存时避免一次性对所有key进行重哈希，而是将重哈希操作分散到对于dict的各个增删改查的操作中去。这种方法能做到每次只对一小部分key进行重哈希，而每次重哈希之间不影响dict的操作。dict之所以这样设计，是为了避免重哈希期间单个请求的响应时间剧烈增加，导致Redis无法快速响应。

###### incremental rehashing的实现
实现的基本思路是：dict的数据结构里包含两个哈希表。在重哈希期间，数据从第一个哈希表向第二个哈希表迁移，具体可以参见Redis源码dict.h

#### sds
全称是Simple Dynamic String，特点如下：
1.可动态扩展内存。sds表示的字符串其内容可以修改，也可以追加。在很多语言中字符串会分为mutable和immutable两种，显然sds属于mutable类型的。
2.二进制安全（Binary Safe）。sds能存储任意二进制数据，而不仅仅是可打印字符。

##### sds与string的关系
当它存储的值是个字符的时候，内部结构为sds，当它存储的值是个数字，并且string类型本身支持incr、decr等操作，内部结构