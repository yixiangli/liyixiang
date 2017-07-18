---
title: Zookeeper理论
toc: true
date: 2017-04-25 16:08:35
category: 
	- 技术贴
	- 中间件
	- 分布式系列
	- Zookeeper
tags: 
    - Zookeeper   
    - 选举
---

# Zookeeper
分布式协调服务，具体就不介绍了，这里不是百度百科。一句话
**zookeeper=文件系统+通知机制。**

先来一张图
![Zookeeper工作流程图](/img/Zookeeper.jpg)
<!--more-->

1.Server对于client透明，client可以连接任意某个Server
2.其中一个Server是Leader,其他Server对其进行数据同步
先大概了解了Zookeeper的整体结构，接下来分析工作原理

## 工作原理
刚才提到了其中一个Server是Leader,那么说明Server之间也是有划分的.接下来分别介绍

**三角色：**
leader ：领导者
follower ： 跟随者
observer ： 观察者
**四状态：**
LOOKING：当前Server不知道leader是谁，正在搜寻。
LEADING：当前Server即为选举出来的leader。
FOLLOWING：leader已经选举出来，当前Server与之同步。
OBSERVING：observer的行为在大多数情况下与follower完全一致，但是他们不参加选举和投票，而仅仅接受(observing)选举和投票的结果。


## 数据模型
![Zookeeper数据模型](/img/zk-struct.jpg)

根据图片来依次介绍，首先看局部，节点ZNode的特性，Zookeeper有四种类型的ZNode 

1、PERSISTENT-持久化目录节点 
客户端与Zookeeper断开连接后，该节点依旧存在 
2、PERSISTENT_SEQUENTIAL-持久化顺序编号目录节点 
客户端与Zookeeper断开连接后，该节点依旧存在，只是Zookeeper给该节点名称进行顺序编号
3、EPHEMERAL-临时目录节点 
客户端与Zookeeper断开连接后，该节点被删除 
4、EPHEMERAL_SEQUENTIAL-临时顺序编号目录节点 
客户端与Zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行顺序编号

咱们结合命令，查看某个节点的属性

```
[zk: 127.0.0.1:2181(CONNECTED) 3] get /zk
null
cZxid = 0x300000033
ctime = Wed Jul 13 20:15:12 CST 2016
mZxid = 0x300000033
mtime = Wed Jul 13 20:15:12 CST 2016
pZxid = 0x30069c4a3
cversion = 8
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 6
```

**参数详解：**
第一行null是该节点下存储的数据，无论是何种ZNode,每一个ZNode默认能够存储1MB的数据
czxid：创建节点的事务的zxid
mzxid：对ZNode最近修改的zxid
ctime：以距离时间原点(epoch)的毫秒数表示的ZNode创建时间
mtime：以距离时间原点(epoch)的毫秒数表示的ZNode最近修改时间
version：ZNode数据的修改次数
cversion：ZNode子节点修改次数
aversion：ZNode的ACL修改次数
ephemeralOwner：如果ZNode是临时节点，则指示节点所有者的会话ID；如果不是临时节点，则为零。
dataLength：ZNode数据长度。
numChildren：ZNode子节点个数。

说完了局部，再来看整体，Zookeeper数据模型特点如下：
1）每个子目录项如图中说明，都被称作为ZNode，这个ZNode是被它所在的路径唯一标识，如Server1这个ZNode的标识为/path1/child1/1。
2）ZNode可以有子节点目录，并且每个ZNode可以存储数据，注意EPHEMERAL（临时的）类型的目录节点不能有子节点目录。
3）ZNode是有版本的（version），每个ZNode中存储的数据可以有多个版本，也就是一个访问路径中可以存储多份数据，version号自动增加。
4）ZNode可以是临时节点（EPHEMERAL），可以是持久节点（PERSISTENT）。如果创建的是临时节点，一旦创建这个EPHEMERALZNode的客户端与服务器失去联系，这个ZNode也将自动删除，Zookeeper的客户端和服务器通信采用长连接方式，每个客户端和服务器通过心跳来保持连接，这个连接状态称为session，如果ZNode是临时节点，这个session失效，ZNode也就删除了。
5）ZNode的目录名可以自动编号，如path1已经存在，再创建的话，将会自动命名为path2,上文提到的_SEQUENTIAL
6）ZNode可以被监控，包括这个目录节点中存储的数据的修改，子节点目录的变化等，一旦变化可以通知设置监控的客户端，这个是Zookeeper的核心特性，Zookeeper的很多功能都是基于这个特性实现的。
7）ZXID：每次对Zookeeper的状态的改变都会产生一个zxid（ZooKeeper Transaction Id），zxid是全局有序的，如果zxid1小于zxid2，则zxid1在zxid2之前发生。

关于这个ZXID多说几句，在我的《Zookeeper实战》一文中讲解了如何分析ZK日志log.x,我随便format一个日志供讲解

```
ZooKeeper Transactional Log File with dbid 0 txnlog format version 2
9/4/16 3:05:31 PM CST session 0x255c3f13ec4e870 cxid 0x3 zxid 0x30040523f closeSession null
9/4/16 3:05:32 PM CST session 0x255c3f13ec4e871 cxid 0x0 zxid 0x300405240 createSession 30000

9/4/16 3:05:32 PM CST session 0x255c3f13ec4e871 cxid 0x3 zxid 0x300405241 closeSession null
9/4/16 3:05:33 PM CST session 0x255c3f13ec4e872 cxid 0x0 zxid 0x300405242 createSession 30000

9/4/16 3:05:33 PM CST session 0x255c3f13ec4e872 cxid 0x3 zxid 0x300405243 closeSession null
9/4/16 3:05:34 PM CST session 0x355c3f13ea8e665 cxid 0x0 zxid 0x300405244 createSession 30000

9/4/16 3:05:34 PM CST session 0x355c3f13ea8e665 cxid 0x3 zxid 0x300405245 closeSession null
9/4/16 3:05:35 PM CST session 0x355c3f13ea8e666 cxid 0x0 zxid 0x300405246 createSession 30000
```

截取了一小段，通过日志描述会发现有这几个参数
1.session
	上文提到，session是每个客户端和服务器保持长连接状态的标识，从日志发现，session是自增的
2.cxid
	创建节点的事务的zxid
3.zxid
	事务id,为了保证事务的顺序一致性，Zookeeper采用了递增的事务id号（zxid）来标识事务。所有的提议（proposal）都在被提出的时候加上了zxid。实现中zxid是一个64位的数字，它高32位是epoch用来标识leader关系是否改变，每次一个leader被选出来，它都会有一个新的epoch，标识当前属于那个leader的统治时期。低32位用于递增计数。说到Zookeeper事务,就不得不说ZAB协议

### 事务提交－ZAB协议
Zookeeper的核心是原子广播，这个机制保证了各个Server之间的同步。那实现这个机制的协议叫做Zab协议。Zab协议中Server有两个模式：Broadcast广播模式、Recovery恢复模式。广播模式很好理解，在Zookeeper集群正常工作时，各个Server通过发送广播信号来同步数据，使各个Server保持最新状态。而恢复模式就要提到容灾了。一般对于Zookeeper来说，进入恢复模式可能出现在服务启动或者领导者崩溃，此时剩下的Follow(角色)会通过选举机制来投票，决定新的领导者，当领导者被选举出来，且大多数Server完成了和leader的状态同步以后，恢复模式就结束了。状态同步保证了leader和Server具有相同的系统状态。

## 选举机制
主要有三种方式：
LeaderElection
AuthFastLeaderElection
FastLeaderElection

默认的算法是FastLeaderElection，先来解释下LeaderElection，一开始看其他人的介绍有点蒙蔽，慢慢琢磨终于理解了，流程不是很复杂，分如下几步：
1.每一个Server在新加入集群后会广播询问一次集群中的其他Server,它们将把票投给谁？
2.当有Server询问自己的时候，自己第一次一定会推荐自己，并把自己的myid和zxid(最近一次处理事务的id)发给询问者
3.收到所有Server回复以后，就计算出zxid最大的那个Server，并将这个Server相关信息设置成下一次要投票的Server
4.统计这个过程中获得票数最多的的Server为获胜者，如果获胜者的票数超过半数，则这个Server被选为leader。否则，继续这个过程，直到leader被选举出来。

FastLeaderElection比LeaderElection区别就在于多了个epoch，epoch在上文有提到，是zxid的高32位，代表当前leader的标识，如果新leader上台，则epoch加1.

### 选举后同步
选完Leader以后，zk就进入状态同步过程。
1.Leader等待server连接；
2.Follower连接leader，将最大的zxid发送给Leader；
3.Leader根据follower的zxid确定同步点；
4.完成同步后通知follower 已经成为uptodate状态；
5.Follower收到uptodate消息后，又可以重新接受client的请求进行服务了。

### Paxos算法
Paxos算法解决的问题是一个分布式系统如何就某个值（决议）达成一致