---
title: MongoDB-常用命令
toc: true
date: 2017-04-25 11:51:24
category: 
	- 技术贴
	- 数据库
	- MongoDB
tags: 
    - CRUD
    - 文档型
---

# 常用命令
学习命令，按照我们平时使用的场景与步骤来讲解，循序渐进

## 系统命令
### 连接
mongo --port=27017 --host=127.0.0.1 -uadmin -pmima --authenticationDatabase admin

**参数详解：**
--port ： 端口
--host ： 地址
-u ： 用户
-p ： 密码
--authenticationDatabase ： 验证权限的数据库
<!--more-->

### 监控工具
mongostat --port=27017 --host=127.0.0.1 -uadmin -pmima --authenticationDatabase admin

**参数详解：**
--port ： 端口
--host ： 地址
-u ： 用户
-p ： 密码
--authenticationDatabase ： 验证权限的数据库

**运行结果：**
![mongodb运行状况](/img/mongostat.jpg)

**参数详解：**
```
insert ： 一秒内的插入数
query ： 一秒内的查询数
update ： 一秒内的更新数
delete ： 一秒内的删除数
```

如果是salve，数值前有一个*，如图。代表是replicate操作，复制细节请看博文-MongoDB-主从复制和副本集
![salve运行状况](/img/mongostat_slave.jpg)

```
getmore ：查询时游标(cursor)的getmore操作
command ： 一秒内执行的命令数
```

这个有必要解释一下，如果是批量insert,也只是一条命令。另外，这行有两个数值，意义为local|replicated，通过这个比较，可以发现复制的相关问题。

```
% dirty ： 这个是Wried Tiger引擎所特有的参数，数值是缓存中无效数据所占的百分比
% used ：这个也是Wried Tiger引擎所特有的参数，数值是正在使用的缓存百分比
```

通过这两个数值可以看出当前mongodb中热数据缓存命中，如果dirty比较大，说明缓存有必要清理一下，used百分比使用大，说明数据访问量较大。

```
flushes ： 一秒内flush的次数

```

一般都是0，或者1，通过计算两个1之间的间隔时间，可以大致了解多长时间flush一次。这有点像gc,flush开销很大，如果很频繁，说明就有原因。

```
vsize ：最后一次调用mongostat时，进程中虚拟内存使用的大小 
res ： 最后一次调用mongostat时，进程中常驻存储器内存大小
```

和top一样，mapped, vsize一般不会有大的变动，res会慢慢的上升，如果res经常突然下降，去查查是否有别的程序狂吃内存。

```
qr|qw ：q的意义是queue lengths for clients waiting，队列中等待的客户端请求数，r代表read,w代表write
ar|aw ：a的意义是active clients，活跃的客户端请求，DB还未处理
```

因此如果这两个数值很大，那说明DB被堵住了，DB的处理速度不及请求速度，这个时候先看是否有大查询或者慢日志，如果一切正常，那么确实是负载很大，就需要扩容了。

```
netIn ： 网络带宽入口
netOut ：  网络带宽出口
```

通过这两个参数可以观察网络带宽压力。

```
conn ： 开启的连接数
```

MongoDB为每一个连接创建一个线程，线程的创建和释放也是有开销的。因此尽量不要让这个数值很大。

```
set ：数据库名
repl ： 服务器当前状态
```
服务器状态主要有以下几种
M   - master
SEC - secondary
REC - recovering
UNK - unknown
SLV - slave
    
``` 
time ：当前时间
```

### 系统相关
**查看集群信息：**
db.serverStatus() 

**主从切换：**
db.getMongo().setSlaveOk();

**数据备份：**
mongodump --port=9033 --host=10.11.144.91 -uadmin -pMzU3ZmU4YmFhZDg --authenticationDatabase admin -d lepush -c clients -o a.dat

**数据导出：**
mongoexport --port=9033 --host=10.11.144.91 -uadmin -pMzU3ZmU4YmFhZDg --authenticationDatabase admin -d lepush -c clients -o 

**慢日志查看：**
日志路径一般在mongo安装目录etc下cnf文件中，在cnf文件中配置

```
systemLog:
   destination: file
   path: "/data1/WZ_mongo/logs/mongo.log"
   logAppend: true
```

### 集合操作

db.createCollection("xxx");

### 数据操作
#### 插入
db.collection.insert({"":""})

#### 现有表中新增字段
db.apps.update({}, {$set:{state:0}}, false, true);

##### 查询
**普通查询:**
db.collection.find()

**格式化查询:**
db.collection.find().pretty();

##### 更新
db.default_url_node.update({"_id":ObjectId("573d5e206445fe80d1b768e5")},{$set:{"url":"cn.api.abc.com"}})

##### 条件
**小于:**
db.clients.count({"updateTime":{$lt:20160430100000}})
    
**匹配:**
$in 模糊匹配
// $in:[1,2] 只要 在 a.x 的 array 中 包含 一个  1,2 ... 就真

$all 完全匹配
//$all:[1,2] 必须 包含 1,2 才真

db.regionTest.find({ "citylevel" : { "$in" : [2, 3, 4, 5] }, "provinceid" : { "$nin" : [44] }, "districtid" : { "$nin" : [445, 556, 555] } }).explain()


### 索引操作
#### 给devId+appId建立唯一约束
db.xxx.ensureIndex({'devId':1,'appId':1},{'unique':true});
#### 给model state edition创建复合索引
db.xxx.ensureIndex({model:1,state:1,edition:1},{background: 1})
#### 给devId token 创建单独索引
db.xxx.ensureIndex({devId:1})db.clients_ios.ensureIndex({token:1})
#### 查看某个表上的所有索引
db.xxx.getIndexes();
#### 查询是否走索引
xxx.explain()

### 复杂查询

##### 去重统计
db.clients.aggregate({$group:{_id:"$F", deviceid: {$sum:1}}})

##### 查询每个app的个数    
aggregate([{ $group : {"_id" : "$appid" , "count" : {$sum : 1}}}])    

##### mapReduce
a.datmapReducedb.clients.mapReduce(function() { emit(this.appid,1);},function(key, values) {return Array.sum(values)},{query:{sessionid:40},out:"total"}).find()
db.clients.group({"key":{"uid":true},"initial":{"pCount":[ ]},"reduce":function(key,value){value.pCount.push(key.appid);},"finalize":function(value){value.count = value.pCount.length;},"condition":{$and:[{uid:{$ne:""}},{$and:[{appid:{$in:["0","26"]}},{appid:{$in:["34","40"]}}]}]}})


## 常见问题

### 终端操作
mongo终端插入数字，默认为double

### 版本
mongo3版本不支持dropDups:true

### 其他
group()函数  最多支持20000唯一key

### count锁死
mongo的count()在数据量很大的情况下会锁数据库 update等操作无法进行
    count()的原理是会把查询数据load到内存中，这个时候内存不足会flash掉之前内存里的热点数据
    
### 数据清洗
清洗某一张表中数据，实际上内存不会释放，除非重启mongo节点，重新拉取数据


