---
title: 网络基础-TCP
toc: true
date: 2016-12-27 17:25:37
category: 
	- 技术贴
	- 网络
	- TCP
tags: 
    - tcp
    - 三次握手
---


# 网络


## 三次握手 四次挥手
**为什么要三次握手？**
TCP三次握手的目的是为了解决网络中存在延迟而重复分组的问题，防止server端一直等待，浪费资源。保证数据的可靠传输。
存在网络延迟导致失效的连接请求报文段发送到server端，然后server误认为要建立长连接。

**为什么要四次挥手？**
保证双方的一个合约的完整执行,双向通信，一端想断开，但是另一端有可能还在发送数据。

### 手绘图
![TCP三次握手](/img/tcp.jpg)


一些解释：
```
这里的C端(client) S端(server) 并不要理解为客户端（移动设备）服务端（服务器），而是说连接建立发起方（主动方），连接关闭发起方（主动方）。
```
<!--more-->
各个状态的意义如下： 
```
LISTEN - 侦听来自远方TCP端口的连接请求； 
SYN-SENT -在发送连接请求后等待匹配的连接请求； 
SYN-RECEIVED - 在收到和发送一个连接请求后等待对连接请求的确认； 
ESTABLISHED- 代表一个打开的连接，数据可以传送给用户； 
FIN-WAIT-1 - 等待远程TCP的连接中断请求，或先前的连接中断请求的确认；
FIN-WAIT-2 - 从远程TCP等待连接中断请求； 
CLOSE-WAIT - 等待从本地用户发来的连接中断请求； 
CLOSING -等待远程TCP对连接中断的确认； 
LAST-ACK - 等待原来发向远程TCP的连接中断请求的确认； 
TIME-WAIT -等待足够的时间以确保远程TCP接收到连接中断请求的确认； 
CLOSED - 没有任何连接状态；
```

### 三次握手

TCP是主机对主机层的传输控制协议，提供可靠的连接服务，采用三次握手确认建立一个连接，其中位码即tcp标志位,有6种标示:

SYN(synchronous建立联机)
ACK(acknowledgement 确认)
PSH(push传送)
FIN(finish结束)
RST(reset重置)
URG(urgent紧急)

Sequence number(顺序号码)
Acknowledge number(确认号码)