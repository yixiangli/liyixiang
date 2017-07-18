---
title: RESTful
toc: true
date: 2017-03-10 11:17:53
category: 
    - 技术贴
    - 理论与概念
tags: 
    - REST
    - http
---

# 什么是REST
REST（Representational State Transfer）是由HTTP协议制定者Roy Fielding博士提出的一种架构原则。

## 特点

- 资源化
  互联网上所有的信息都可以看作是一种资源
- 接口统一
  对互联网上的CRUD区分通过HTTP协议的请求类型
  对返回状态有统一的规则约束
- 操作分类
  获取同一个资源对外只有一个uri
- 安全性
  Basic Auth OAuth   
  
<!--more-->
## 主要规则

### Request
GET（SELECT）：从服务器取出资源（一项或多项）
POST（CREATE）：在服务器新建一个资源
PUT（UPDATE）：在服务器更新资源（客户端提供完整资源数据）
PATCH（UPDATE）：在服务器更新资源（客户端提供需要修改的资源数据）
DELETE（DELETE）：从服务器删除资源

### 分版本控制
对于版本控制的处理，主要分为两种
1.写在uri中：http://www.example.com/1.0/album
2.加在header的Accept中，如Accept：version=1.0

第一种代表网站 CSDN
第二种代表网站 Github

个人建议第二种，原因是uri上应该都是同一资源，而版本只是对于这个资源操作的不同表现，目的还是操作本资源，因此不应当放在uri上。

### 名词代替动词
Restful和平常接口定义不同的是，不再使用add/delete/update/show等动词，而是纯名词，如果只是对资源的获取，那么就是该资源的名词定义，而如果是一种动态的转换，也使用其名词定义，代表的是提供一种服务（服务即能力）

Not Use
1. /addAlbum POST id=1&name=xxx&title=eee
2. /updateAlbum POST id=1
3. /showAlbums
4. /accounts/1/transfer/500/to/2

Can Use
1./Album POST id=1&name=xxx&title=eee
2./Album/1 PUT
3./Albums GET
4./transaction POST from=1&to=2&amount=500.00

### Response
当GET, PUT和PATCH请求成功时，要返回对应的数据，及状态码200，即SUCCESS
当POST创建数据成功时，要返回创建的数据，及状态码201，即CREATED
当DELETE删除数据成功时，不返回数据，状态码要返回204，即NO CONTENT
当GET 不到数据时，状态码要返回404，即NOT FOUND
任何时候，如果请求有问题，如校验请求数据时发现错误，要返回状态码 400，即BAD REQUEST
当API 请求需要用户认证时，如果request中的认证信息不正确，要返回状态码 401，即NOT AUTHORIZED
当API 请求需要验证用户权限时，如果当前用户无相应权限，要返回状态码 403，即FORBIDDEN

## 实例

GET /album 				# 获取album列表
GET /albums/12 			# 查看某个具体的album
POST /albums 			# 新建一个album
PUT /albums/12 			# 更新album 12
DELETE /albums/12 		# 删除ticekt 12