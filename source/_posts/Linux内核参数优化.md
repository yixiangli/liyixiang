---
title: Linux内核参数优化
toc: true
date: 2016-12-27 17:14:22
category: 
	- 技术贴
	- Linux
tags: 
    - 内核
    - linux
---

# 内核参数

/etc/sysctl.conf

## 参数详解
net.ipv4.tcp_fin_timeout
默认60s，减小fin_timeout，减少TIME_WAIT连接数量。
net.ipv4.tcp_tw_reuse = 1
表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
net.ipv4.tcp_tw_recycle = 1
表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。
net.ipv4.tcp_syncookies = 1 
表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭； 
net.ipv4.tcp_keepalive_time = 1200 
表示当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时，改为20分钟。 
net.ipv4.ip_local_port_range = 10000 65000 
表示用于向外连接的端口范围。缺省情况下很小：32768到61000，改为10000到65000。（注意：这里不要将最低值设的太低，否则可能会占用掉正常的端口！） 
net.ipv4.tcp_max_syn_backlog = 8192 
表示SYN队列的长度，默认为1024，加大队列长度为8192，可以容纳更多等待连接的网络连接数。 
net.ipv4.tcp_max_tw_buckets = 5000 
表示系统同时保持TIME_WAIT的最大数量，如果超过这个数字，TIME_WAIT将立刻被清除并打印警告信息。默 认为180000，改为5000。对于Apache、Nginx等服务器，上几行的参数可以很好地减少TIME_WAIT套接字数量，但是对于 Squid，效果却不大。此项参数可以控制TIME_WAIT的最大数量，避免Squid服务器被大量的TIME_WAIT拖死。


## 一些经验
（1）遇到过一次故障：查看/var/log/messages系统日志发现possible SYN flooding on port 80. Sending cookies （类似syn洪水攻击）后经查询断定是请求过大导致
net.ipv4.tcp_syncookies = 1  配置这个参数有一定的效果，但如果量过大，还是会gg的
（2）time-wait数量快速回收
查看服务器time-wait数量 sudo netstat -antp |grep TIME_WAIT |wc -l
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
配置这两个参数可以大大减少time_wait数量，不信你就试试


## 生效
sysctl -p