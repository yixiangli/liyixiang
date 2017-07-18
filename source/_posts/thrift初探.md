---
title: thrift初探
toc: true
date: 2017-07-08 23:42:25
category: 
	- 技术贴
	- Java
	- 框架
tags: 
    - 通信
---

# Thrift介绍
说起Thrift框架,由于之前工作中一直没约定使用内部rpc方式（都是http）进行交互，因此对Thrift也只是听说并未实现，正巧换工作，新厂对外提供服务除了传统的http接口外也提供了大量的rpc，因此终于踏上学习之路。

## 层次结构
Server 服务层 整合上述组件, 提供网络模型(单线程/多线程/事件驱动), 最终形成真正的服务.
Protocol 协议层 定义数据传输格式，可以为二进制或者XML等
Processor 处理层 这部分由定义的idl来生成, 封装了协议输入输出流, 并委托给用户实现的handler进行处理.
Transport 传输层 定义数据传输方式，可以为TCP/IP传输，内存共享或者文件共享等

<!--more-->
## 使用心得

### thrift文件
namespace java xxxx

const

struct

service

### 自动生成插件
附上插件地址，下载下来后直接在IDEA install一下即可，使用方法也特别简单，选中要生成代码的thrift文件右键，Generate java file。
[下载地址](http://www.fishfly.cn/download/thrift-gen-plugin.jar)


### 一些坑
先说0.8.0,博主本人遇到的第一个坑就是编译器bug,这个bug主要针对0.8.0版本thrift文件中的多行注释，如果这么写

```
/**这是一行注释**/

```

IDEA就会报 "Assertion `docstring.length() <= strlen(doctext)' failed" , 那么正确的方式就是文字前后空行

```
/** 这是一行注释 **/

```


