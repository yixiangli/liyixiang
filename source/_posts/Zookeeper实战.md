---
title: Zookeeper实战
toc: true
date: 2016-12-07 14:28:35
category: 
	- 技术贴
	- 中间件
	- 分布式系列
	- Zookeeper
tags: 
    - Zookeeper   
    - 集群
---


# 概述
Zookeeper是Hadoop开源的子项目，作为互联网极速发展的今天，很多技术场景都需要进行分布式管理，因此，Zookeeper作为一款分布式协调服务，深受广大开发者喜爱。

<!--more-->
## 常用命令
1. 启动ZK服务:       sh bin/zkServer.sh start
2. 查看ZK服务状态（角色）: 	sh bin/zkServer.sh status
3. 停止ZK服务:       sh bin/zkServer.sh stop
4. 重启ZK服务:       sh bin/zkServer.sh restart
1. 显示根目录下、文件： ls / 使用 ls 命令来查看当前 ZooKeeper 中所包含的内容
2. 显示根目录下、文件： ls2 / 查看当前节点数据并能看到更新次数等数据
3. 创建文件，并设置初始内容： create /zk "test" 创建一个新的 znode节点“ zk ”以及与它关联的字符串
4. 获取文件内容： get /zk 确认 znode 是否包含我们所创建的字符串
5. 修改文件内容： set /zk "zkbak" 对 zk 所关联的字符串进行设置
6. 删除文件： delete /zk 将刚才创建的 znode 删除
7. 退出客户端： quit
8. 帮助命令： help 

## 使用
Maven

```
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.3.1</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>2.4.2</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>2.4.2</version>
</dependency>
```

经测试，其他依赖的三个jar下载不下来，手动下载并copy进本地maven库
jms-1.1.jar jmxtools-1.2.1.jar jmxri-1.2.1.jar

### 集群搭建步骤

1.根据dataDir的配置路径，在该路径下新建文件夹myid文件， server.X 这个数字就是对应 myid中的数字。

2.配置文件zoo.cfg

```
 # The number of milliseconds of each tick
 tickTime=2000
 # The number of ticks that the initial
 # synchronization phase can take
 initLimit=10
 # The number of ticks that can pass between
 # sending a request and getting an acknowledgement
 syncLimit=5 
 # the directory where the snapshot is stored.
 # do not use /tmp for storage, /tmp here is just
 # example sakes.
 dataDir=/letv/data/zookeeper
 # the port at which the clients will connect clientPort=2181
 # the maximum number of client connections.
 # increase this if you need to handle more clients
 #maxClientCnxns=60
 #
 # Be sure to read the maintenance section of the
 # administrator guide before turning on autopurge.
 # # http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
 # # The number of snapshots to retain in dataDir 
 #autopurge.snapRetainCount=3
 # Purge task interval in hours
 # Set to "0" to disable auto purge feature
 #autopurge.purgeInterval=1
 server.1=10.185.28.73:2888:3888
 server.2=10.185.31.189:2888:3888
 server.3=10.185.31.197:2888:3888  
```

3.启动
./zkServer.sh start

4.客户端连接
./zkCli.sh -server 127.0.0.1:2181


## 恢复
配置Zookeeper的时候，会在zoo.cfg文件中指定dataDir路径，这个路径下会存放数据文件和日志文件，两个一起能够恢复Zookeeper集群数据。一般来说数据文件名为：snapshot.x，而日志文件对应为：log.x+1。

```
-rw-r--r-- 1 root root 67108880 Apr 12 17:57 log.1
-rw-r--r-- 1 root root 67108880 Apr 12 18:04 log.8c
-rw-r--r-- 1 root root 67108880 Apr 12 18:06 log.9e
-rw-r--r-- 1 root root 67108880 Apr 12 18:08 log.a9
-rw-r--r-- 1 root root      389 Apr 12 16:55 snapshot.0
-rw-r--r-- 1 root root     1067 Apr 12 18:00 snapshot.8b
-rw-r--r-- 1 root root     1079 Apr 12 18:05 snapshot.9d
-rw-r--r-- 1 root root     1079 Apr 12 18:07 snapshot.a8
```

## 日志分析
1.如何查看zookeeper事务日志文件

使用LogFormatter工具，在zookeeper安装目录下执行
```
java -classpath .:lib/slf4j-api-1.7.5.jar:zookeeper-3.5.0-alpha.jar org.apache.zookeeper.server.LogFormatter /tmp/zookeeper/version-2/log.1
```

/tmp/zookeeper/version-2/log.1 为zoo.cfg配置文件中dataDir配置的目录

## 常见问题
1.Zxid达到最大值时，Zookeeper会怎样？
zxid达到最大值后会触发集群重新选举，然后zxid会变为0。日志中会看到如下信息：

```
INFO [ProcessThread(sid:31814 cport:-1)::PrepRequestProcessor@137] - zxid lower 32 bits have rolled over, forcing re-election, and therefore new epoch start
```
    
## 使用场景
配置管理
名字服务
分布式锁
集群管理
