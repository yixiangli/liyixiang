---
title: Linux命令快查表
toc: true
date: 2017-05-03 17:00:47
category: 
	- 技术贴
	- Linux
tags: 
    - 命令
    - linux
---

# 系统权限篇
进入root权限模式    
sudo su

输入密码登陆root权限 
su root

添加root权限
vi  /etc/sudoers
添加用户
useradd liyixiang  
passwd liyixiang

<!--more-->
# 系统内核篇
重新生成密钥
ssh-keygen -t rsa

查看系统日志
cat /var/log/messages
本地绑host(利用dns缓存)
vim /etc/hosts
配置hosts
    /etc/hosts 中配置ip pushxxx
    pushxxx为别名 在ssh等命令下就可直接使用别名而无需输入ip,简化命令
服务自启动配置 
在/etc/rc.local下添加各种shell启动脚本

查询DNS Server 
cat /etc/resolv.conf

清除DNS缓存
/etc/init.d/nscd restart

linux内核优化 
快速回收time_wait
vim /etc/sysctl.conf
    net.ipv4.tcp_tw_reuse = 1
    net.ipv4.tcp_tw_recycle = 1
文件生效
    sudo sysctl -p
系统控制参数配置
    显示系统资源的设置（包含全部）
    ulimit -a 
<程序数目> 程序数目上限，最多可开启的程序
    ulimit -u   
设置系统核心参数
    sysctl

查看cpu数
grep "model name" /proc/cpuinfo | cut -f2 -d:
查看 CPU 个数
sysctl -n machdep.cpu.core_count
    
查看TCP连接状态
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
查看服务time_wait的数量
sudo netstat -antp |grep TIME_WAIT |wc -l
查看estab的数量
sudo netstat -antp |grep ES |wc -l
查看最大time_wait数
cat /proc/sys/net/ipv4/tcp_max_tw_buckets
    
查看服务器外网ip
curl ifconfig.me
查看vip
ip a | grep '/32'
查看网络信息
ifconfig    
查看网卡信息
ethtool eth2
dns解析过程
dig +cmd +trace api.push.abc.com
域名相关
nslookup
交互

```
nslookup
> api.push.abc.com
```

非交互

```
nslookup api.push.abc.com
```

查看当前系统网络状态
ss -s     
查看网卡个数
netstat -i
查看防火墙
/etc/init.d/iptables status
查看计算机本地arp表
arp -a     
第一列：表示ip地址列表情况。
第二列：表示ip地址对应的mac地址。
第三列：表示ip地址类型，是动态ip还是静态ip。
/proc/sys/net/ipv4/neigh/default
    
    
查看内存
grep MemTotal /proc/meminfo | cut -f2 -d:     
查看swap
free -m

```
             total       used       free     shared    buffers     cached
    Mem:         1002        769        232          0         62        421
    -/+ buffers/cache:        286        715
    Swap:         1153          0       1153
```

解释说明：
第二部分(-/+ buffers/cache):
    (-buffers/cache) used内存数：286M (指的第一部分Mem行中的used - buffers - cached)
    (+buffers/cache) free内存数: 715M (指的第一部分Mem行中的free + buffers + cached)
    可见-buffers/cache反映的是被程序实实在在吃掉的内存，而+buffers/cache反映的是可以挪用的内存总数。
    对操作系统来讲是Mem的参数.buffers/cached 都是属于被使用,所以它认为free只有232.
    对应用程序来讲是(-/+ buffers/cach).buffers/cached 是等同可用的，因为buffer/cached是为了提高程序执行的性能，当程序使用内存时，buffer/cached会很快地被使用。所以,以应用来看看,以(-/+
    buffers/cache)的free和used为主
  
    
查看某个进程的线程运行情况
top –p pid -H
查看java进程的端口
netstat -apn     输出格式为port pid/java
进程监控
monit

查看文件的编码
    vim fileName
    :set fileencoding

# 基础指令篇
返回上一层目录
    cd - 
    
进入名称为-的目录
    cd -- -/
    
修改文件生成时间
    touch -m -d "2017-03-03 01:10"
    
上传
    rz –y  
    
下载
    sz –y xxx
    
删除
    rm –rf xxx.x  
    
复制文件夹（在删除前备份）
    cp –r  xxx.xx xxx_bak .xx 
    
查看xxx状态
    ps –ef|grep xxx 
    
移动文件
    mv xxxxx  
    
解压war包
    jar –xvf  xxx.war  
    
.gz解压
    gunzip lepush-api.20160216.log.gz
    
执行脚本开启/关闭
    xxx.sh start    
    xxx.sh stop  
    
查看当前目录下全部文件夹大小 
    du -sh *
    
vim 显示整个文件
    I 可写  u回退  esc退出编辑状态
    最后先按！再wq强制写入。切记wq     写在！前面    切记!前面不要有.
    
查看某文件
    less xxx  
    
实时监控某文件
    tail  –f  xxx  
    
连接某端口
    telnet ip 端口   
    
查看当前文件夹
    ls
    
查看当前文件夹下的所有文件（包括隐藏文件）
    ls –a 
    
查看权限（需要配置快捷）
    ll  等价于：ls –l 
    
查看操作记录
    history 
    
发送post请求
    curl -d "参数" "url"
    
删除指定字符串的行
    sed -i '/\<is not found in db\>/d' error.log
    
当前暂停与关闭
    ctrl+z  暂停
    jobs + fg 1  继续执行
    ctrl+c  关闭

关闭telnet
    ctrl + ] 进入telnet>   输入quit
    
文件copy
需要提前配置好白名单以及公钥
	scp hadoop-2.6.0.tar.gz root@10.11.144.118:/letv/soft
    
查询mongo开头的文件
    find / -name "mongo*"

# 日志分析篇
cat lepush-api-9040-2016-10-10.log | grep '^20:' | grep 'time' | grep 'INFO' | awk '{time=substr($NF,0,length($NF)-1);if(time+0>300) print}'
    
tail -f api-debug-9040-2016-10-11.log | grep 'WARN' | awk '{if($NF>100) print}'


统计nginx每秒请求数
tail -f nginx.access.log | awk '{print $4}' | awk -F: '{print $4}' | uniq -c
    
统计nginx响应大于1秒
tail -f nginx.access.log | awk '{if($NF>1) print}'
    
tail -f access.log | awk '{print $3}' | awk -F: '{print $4}' | uniq -c
    
滤重
    sort token.txt | uniq -c | sort -rn(降序) > abcd.txt
    cat abcd.txt | awk '{if($1>1) print $1 , $2}'

统计nginx每分钟请求数
    cat nginx.access.log  | awk '{print substr($3,0,18)}' | uniq -c
    
某时段每秒请求数
    cat nginx.access.log | grep '29/Jul/2016:11:58:'  | awk '{print substr($3,0,21)}' | uniq -c
    cat nginx.access.log | awk '{print substr($3,0,21)}' | uniq -c
    
统计每秒后端请求数
    cat lepush-api-9040-2016-06-25.log  | awk '{print substr($1,0,8)}' | uniq -c
    
非200请求
    cat nginx.access.log | awk '{if($(NF-1)!=200) print}'
    
非200非无状态请求
    cat nginx.access.log | awk '{if($(NF-1) ~ /^[0-9]+$/ && $(NF-1) !=200) print}' 
    
每小时请求响应大于1秒的总和
    cat nginx.access.log | awk '{if($NF>1) print substr($3,0,15)}' | uniq -c
    
,分割
    cat android_client.txt | awk 'BEGIN{FS=","}{print $4}'
    
滤重统计
    sort -u
    sort android_client.txt | awk 'BEGIN{FS=","}{print $15}' | uniq -c > sort.txt
    
大于1统计
    cat sort.txt | awk '{if($1!=1) print}' > dayu1.txt
    
一列数字做累加
    cat dayu1.txt | awk '{print $1}' | awk '{sum+=$1}END{print sum}'
    
非空
    cat dayu20160815.txt | awk '{if(length($3)!=0)print $3}'
    
统计推送总数
    cat lepush-server-2016-08-17.log | grep "16:" | grep "clientids count" | awk '{print $7}' | awk -F '=' '{sum+=$2} END {print sum}'
    cat lepush-server-2016-08-17.log | grep "16:" |     grep "error clientids count=" | awk '{print $8}' | awk -F '=' '{sum+=$2} END {print sum}'
    
获取文件第10行
    cat file.txt | awk 'NR==10'
    
匹配文件中不是0-9 a-z等字符串
    awk '{if($1!~/^[A-Za-z0-9]+$/)print}' device.txt

不包含  
    grep -v 
匹配的是完整单词    
    grep -w 
    
交集
    grep -F -f app_sort.txt sdk_sort.txt | sort | uniq
    
差集
    grep -F -v -f app_sort.txt sdk_sort.txt | sort | uniq

# 工具篇
**SecureCRT**
使用sftp
command+shift+p 
