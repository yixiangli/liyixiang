---
title: Nginx配置详解
toc: true
date: 2017-01-11 10:11:07
category: 
	- 技术贴
	- web中间件
	- Nginx
tags: 
    - Nginx
---


# what is Nginx
一款高性能反向代理Web服务器

## 正向&反向代理
一个网友画的图，很生动
![代理](/img/proxy.jpg)

正向：比较好理解，访问不了google，然后买个代理账号或者搭个vpn，它可以访问，你只需要借助它来对外访问。
反向：
（1）真正服务器的替身，你访问反向代理，它其实是从真正存储或者处理内容的服务器上获取结果，然后转发回客户端，但是在客户端认为，他就是目的源。
（2）负载均衡
 可以在一个组织内使用多个代理服务器来平衡各 Web 服务器间的请求负载，这个很常见，一般互联网公司都是使用nginx作为软负载均衡，还记得之前一般用lvs，后面大部分都切换成nginx了，原因之一是因为lvs跨机房性能会急剧下降，水平扩展性相对来说要差点。
<!--more-->
## 一份较为完整的conf配置
服务器硬件配置
128G 24U

## 定义Nginx运行的用户和用户组
user  root;

## 此处要根据机器硬件情况来配置
worker_processes  24;
worker_cpu_affinity 000000000000000000000001 000000000000000000000010 000000000000000000000100 000000000000000000001000 000000000000000000010000 000000000000000000100000 000000000000000001000000 000000000000000010000000 000000000000000100000000 000000000000001000000000 000000000000010000000000 000000000000100000000000 000000000001000000000000 000000000010000000000000 000000000100000000000000 000000001000000000000000 000000010000000000000000 000000100000000000000000 000001000000000000000000 000010000000000000000000 000100000000000000000000 001000000000000000000000 010000000000000000000000 100000000000000000000000;


## 全局错误日志定义类型，[ debug | info | notice | warn | error | crit ]  其他不解释了 crit  意为记录最少错误信息（应该是减少日志写压力）  如果说实在不需要记录，error_log /dev/null crit;   将日志丢到Linux黑洞里 注意error_log off并不能关闭日志记录功能，它将日志文件写入一个文件名为off的文件中
error_log  /letv/logs/nginx.error.log;


## pid（进程标识符）：cat一下这个文件发现记录的是进程号
pid  logs/nginx.pid;


## 指定进程可以打开的最大描述符数 这个值最好和ulimit -n的值相同
worker_rlimit_nofile 102400;


## nginx 事件模块
events {

    网上摘录一段较为详细的解释
    使用epoll的I/O 模型。linux建议epoll，FreeBSD建议采用kqueue，window下不指定。
    补充说明:
        与apache相类，nginx针对不同的操作系统，有不同的事件模型
        A）标准事件模型
        Select、poll属于标准事件模型，如果当前系统不存在更有效的方法，nginx会选择select或poll
        B）高效事件模型
        Kqueue：使用于FreeBSD 4.1+, OpenBSD 2.9+, NetBSD 2.0 和 MacOS X.使用双处理器的MacOS X系统使用kqueue可能会造成内核崩溃。
        Epoll：使用于Linux内核2.6版本及以后的系统。
        dev/poll：使用于Solaris 7 11/99+，HP/UX 11.22+ (eventport)，IRIX 6.5.15+ 和 Tru64 UNIX 5.1A+。
        Eventport：使用于Solaris 10。 为了防止出现内核崩溃的问题， 有必要安装安全补丁。    

    use epoll;
    ## 每个工作进程的最大连接数量 这个最好也根据硬件资源调整
    worker_connections  102400;
}

## nginx http模块
http {
    include       mime.types;
    default_type  application/octet-stream;

    charset  utf-8;

    log_format  main  '$http_x_real_ip - [$time_local] '
                      '"$request" $status '
                      '"$http_user_agent" ' 
                      '$upstream_addr  $upstream_response_time  $upstream_status $request_time';
    
    #$remote_addr 上游的ip 如果前端还有代理层，请勿配置
    #$http_x_real_ip 客户端真实的ip 如果代理比较复杂，使用这个
    #$proxy_add_x_forwarded_for 可能会有多个ip 代理层会加上自己的ip向下透传	                      
    #$time_local 记录访问时间与时区          
    #$request    请求url与http方法
    #$status      记录请求状态码
    #$http_user_agent   客户端的相关信息
    #$upstream_addr     访问的后端服务地址
    #$upstream_response_time   访问后端服务消耗的时间
    #$upstream_status  访问后端服务后服务返回的状态码
    #$request_time  整体请求消耗的时间

    sendfile        on;
    tcp_nopush     on;
    tcp_nodelay on;
    keepalive_timeout  0;
    #跳至错误页面关闭nginx数字版本
    server_tokens off;  
    client_header_buffer_size 4k;   
    #开启gzip模块
    gzip  on;
    #设置允许压缩的页面最小字节数，页面字节数从header头中的Content-Length中进行获取。
    #默认值是0，不管页面多大都压缩。
    #建议设置成大于1k的字节数，小于1k可能会越压越大
    gzip_min_length 1k;
    #设置所能接收的最大请求体的大小
    client_max_body_size    2M;
    #设置用户请求头的超时时间
    #client_header_timeout   120;
    #设置用户请求体的超时时间
    #client_body_timeout     120;
    #指定客户端的响应超时时间
    #send_timeout            120;
    #lingering_close生效后，在关闭连接前，会检测是否有用户发送的数据到达服务器，
    #如果超过lingering_timeout时间后还没有数据可读，就直接关闭连接；否则，必须在读取完连接缓冲区上的数据并丢弃掉后才会关闭连接。
    lingering_timeout       30;
    #不允许代理端主动关闭连接
    proxy_ignore_client_abort on;

    upstream lepush_server {
       server 10.140.90.209:8089;
    }
    upstream lepush_api{
       server 127.0.0.1:9040;
       server 127.0.0.1:9041;
       server 127.0.0.1:9042;
    }

    server {
        listen       80;
        server_name  localhost;

        access_log  /letv/logs/nginx.access.log  main;

        location /v1 {
            proxy_pass   http://lepush_server;
            client_max_body_size   500m;  
        }

        location /client {
            proxy_pass   http://lepush_api;
	    proxy_redirect    off;
	    
	    # nginx代理 后台js无法访问的问题  需要添加
	    # 原因：请求js要有来源路径，需要把请求的主机传到应用容器
            proxy_set_header  Host             $host;
            proxy_set_header  X-Real-IP  $http_x_real_ip;
	    #proxy_read_timeout 1;
	    #proxy_next_upstream http_502 http_504 error invalid_header;
            

        }

        location / {
            proxy_pass   http://lepush_api;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }

}

```

###  location 正则匹配
先看官网的语法规则

location
syntax: location [=|~|~*|^~|@] /uri/ { … }
default: no
context: server

一个网上博客的错误
```
= 严格匹配。如果这个查询匹配，那么将停止搜索并立即处理此请求。
~ 为区分大小写匹配(可用正则表达式)
!~为区分大小写不匹配
~* 为不区分大小写匹配(可用正则表达式)
!~*为不区分大小写不匹配
^~ 如果把这个前缀用于一个常规字符串,那么告诉nginx 如果路径匹配那么不测试正则表达式。

。。。
location !~ xhtml {
}

location !~* png {
}

```
大家一定要注意，实践证明，location后面是无法匹配!的,reload直接报错，如果有不匹配某种规则这种需求，可以这么做
```
location  /client {
    if ($request_uri !~* /list?) {
        proxy_pass   http://common_api; 
    }
    if ($request_uri ~* /list?) {
        proxy_pass   http://lepush_api;
    }
}
```






