---
title: Linux命令详解
toc: true
date: 2017-01-19 10:59:55
category: 
	- 技术贴
	- Linux
tags: 
    - 命令
    - linux
---


# 命令


## 链接
链接是操作系统中的一个重要功能，最常见的就是windows桌面上那一排排快捷方式，其实并不是真正的源文件，而是从源文件目录中找到程序入口点（也就是可执行文件），设置的一个链接，这样可以方便用户在桌面上就可以轻松得操作各种应用程序，不用再到每个文件夹下面去找执行文件，Linux也同理

### 硬链接&软链接
在Linux中链接分为硬链接(hard link)与软链接(symbolic link)
为了分清区别，博主特别找了台服务器尝试下

<!--more-->
源文件
```
-rw-r--r--  2 root root  1411 Jul 19  2016 postfile20.txt
```

硬链接
```
-rw-r--r--  2 root root  1411 Jul 19  2016 lnying.txt
```

软链接
```
lrwxrwxrwx  1 root root    14 Jan 19 10:57 ruan.txt -> postfile20.txt
```

软链接：
1.软链接，以路径的形式存在，类似于Windows操作系统中的快捷方式，从结果看 软链接的大小是很小的，就是一个路径左右的存储大小。
2.软链接可以 跨文件系统 ，硬链接不可以
3.软链接可以对一个不存在的文件名进行链接
4.软链接可以对目录进行链接

硬链接:
1.硬链接，以文件副本的形式存在。但不占用实际空间，从结果看，硬链接大小和源文件一模一样，
2.不允许给目录创建硬链接
3.硬链接只有在同一个文件系统中才能创建

共同特点：
同步性，无论硬链接还是软链接，只要改动一处（源文件｜硬链接出的文件｜软链接出的文件），其他文件都同步修改。

### 格式
```
ln [参数] [源地址] [目标地址] 
```
硬链接：ln 
软链接：ln -s


## 寻址
在操作系统中，每一个文件都有一个固定的地址去存放，外部访问时就需要通过这个地址去访问，那么如何快速知晓某个文件的位置，就需要用到一些寻址命令

### 常见的几个命令
which  查看可执行文件的位置。
whereis 查看文件的位置。 
locate   配合数据库查看文件位置。
find   实际搜寻硬盘查询文件名称。

#### which
一般是用来查找系统命令，如java。很多java程序猿一般都需要通过一些jdk自带的命令来检测jvm等运行情况，但是往往有可能不知道命令的具体位置，这个时候只需要
```
which jstat or  which java
```
一定要注意，这里的java  jstat一定是系统可执行文件。

接下来 我们找cd命令
```
which cd

/usr/bin/which: no cd in (/letv/apps/jdk/bin:/letv/apps/jdk/bin:/letv/apps/jdk/bin:/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin)

```
居然没有找到，再一看提示发现，实际上which命令只会在PATH目录下寻找，而cd是bash 内建的命令。


#### whereis
一般是用来查找定位可执行文件(whereis -b xxx)、源代码文件(whereis -s xxx)、帮助文件(whereis -m xxx)在文件系统中的位置

```
whereis nginx   |   whereis -b nginx

nginx: /usr/local/nginx
```
找到了nginx可执行文件，由于没有源代码和帮助文件，因此-s -m 没有结果。拿find为例
```
[root@WebServer-9788C4 aaa]# whereis find
find: /bin/find /usr/bin/find /usr/share/man/man1/find.1.gz /usr/share/man/man1p/find.1p.gz
[root@WebServer-9788C4 aaa]# 
[root@WebServer-9788C4 aaa]# 
[root@WebServer-9788C4 aaa]# 
[root@WebServer-9788C4 aaa]# whereis -s find
find:
[root@WebServer-9788C4 aaa]# whereis -m find
find: /usr/share/man/man1/find.1.gz /usr/share/man/man1p/find.1p.gz
[root@WebServer-9788C4 aaa]# 
[root@WebServer-9788C4 aaa]# 
[root@WebServer-9788C4 aaa]# whereis -b find
find: /bin/find /usr/bin/find
```
find命令存在可执行文件以及帮助文件，没有源代码文件。博主之前一直不明白whereis的用法，平时尝试使用whereis找文件总是找不到，其实如果要找普通的文本文件肯定是要使用find命令的，接下来会详细说明，至于whereis与find的区别，除了查找的文件类型区别之外，还有就是速度，whereis的速度是远远小于find的，原因就在于linux系统会将系统内的所有文件都记录在一个数据库文件中，whereis直接从数据库中查找，而find是遍历硬盘来查找。当然，这个数据库也不是实时更新，默认一周更新一次，所以会存在脏读的情况。

## 文件
在工作中有的时候我们会遇到一种情况，/abc目录磁盘空间大于90%,当然如何看磁盘空间，可以使用
```
df -h
```
这个时候就会显示
```
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/VGSYS-lv_root
                       20G  3.7G   15G  20% /
tmpfs                  63G     0   63G   0% /dev/shm
/dev/sda1             190M   33M  148M  19% /boot
/dev/mapper/VGSYS-lv_abc
                      513G  448G   65G  85% /abc
/dev/mapper/VGSYS-lv_var
                       20G  725M   18G   4% /var
```
发现/abc目录超过80%了，就要看下是什么文件占用空间，就需要使用

```
du -sh *
```

通过这个命令可以一步一步找到占用空间最大的目录与文件
```
3.1G    apps
4.0K    bak
148K    data
140G    logs
141M    redis
12K     shell
267M    soft
```

此时一看logs占用了140g，说明是日志占用了大量的磁盘空间，那么我们下意识的会把之前的日志清理下，这个时候问题就来了，比如我们发现nginx.access.log特别大想清理，就rm -rf一下，但是会出现du -sh *有变化， df -h没有任何减少的情况，这是由于nginx应用程序还在运行，rm -rf这个指令下达后，linux真正内核还会判断当前文件是否被应用程序打开，如果有打开，肯定不会立即执行清理操作，这也是内核为了保护应用程序所做的必要措施，试想我rm -rf一下应用就直接挂了，也未免太不明智了，那么怎么看当前文件是否被应用程序打开呢
```
lsof
```

lsof显示系统打开的文件，但是因为lsof需要访问核心内存和各种文件，所以必须以root用户的身份运行它才能够充分地发挥其功能。
这个时候我们想删除一下nginx.access.log文件，我们就执行命令

```
lsof nginx.access.log
```

但是发现
```
COMMAND   PID USER   FD   TYPE DEVICE    SIZE/OFF       NODE NAME
nginx   19051 root    5w   REG  253,2 18658278646 2148951603 nginx.access.log
nginx   32154 root    5w   REG  253,2 18658278877 2148951603 nginx.access.log
nginx   32155 root    5w   REG  253,2 18658278877 2148951603 nginx.access.log
nginx   32156 root    5w   REG  253,2 18658278877 2148951603 nginx.access.log
nginx   32157 root    5w   REG  253,2 18658278877 2148951603 nginx.access.log
nginx   32158 root    4w   REG  253,2 18658278877 2148951603 nginx.access.log
nginx   32159 root    3w   REG  253,2 18658278877 2148951603 nginx.access.log
nginx   32160 root    5w   REG  253,2 18658278877 2148951603 nginx.access.log
nginx   32161 root    4w   REG  253,2 18658279213 2148951603 nginx.access.log
nginx   32162 root    3w   REG  253,2 18658279213 2148951603 nginx.access.log
nginx   32163 root    5w   REG  253,2 18658279524 2148951603 nginx.access.log
nginx   32164 root    5w   REG  253,2 18658279756 2148951603 nginx.access.log
nginx   32165 root    5w   REG  253,2 18658280271 2148951603 nginx.access.log
nginx   32166 root    5w   REG  253,2 18658280471 2148951603 nginx.access.log
nginx   32167 root    5w   REG  253,2 18658280704 2148951603 nginx.access.log
nginx   32168 root    5w   REG  253,2 18658280991 2148951603 nginx.access.log
nginx   32169 root    5w   REG  253,2 18658280991 2148951603 nginx.access.log
nginx   32170 root    5w   REG  253,2 18658281598 2148951603 nginx.access.log
nginx   32171 root    4w   REG  253,2 18658281598 2148951603 nginx.access.log
nginx   32172 root    5w   REG  253,2 18658282086 2148951603 nginx.access.log
nginx   32173 root    5w   REG  253,2 18658282764 2148951603 nginx.access.log
nginx   32174 root    3w   REG  253,2 18658282764 2148951603 nginx.access.log
nginx   32175 root    5w   REG  253,2 18658282764 2148951603 nginx.access.log
nginx   32176 root    5w   REG  253,2 18658282764 2148951603 nginx.access.log
nginx   32177 root    5w   REG  253,2 18658282764 2148951603 nginx.access.log
```

大量nginx进程还在访问这个文件，所以就需要执行rm -rf后重启一下nginx，df -h过一会儿就会显示正常了。

## 系统
### TOP
VIRT：virtual memory usage 

    1、进程“需要的”虚拟内存大小，包括进程使用的库、代码、数据等 
    2、假如进程申请100m的内存，但实际只使用了10m，那么它会增长100m，而不是实际的使用量 

RES：resident memory usage 常驻内存 

    1、进程当前使用的内存大小，但不包括swap out 
    2、包含其他进程的共享 
    3、如果申请100m的内存，实际使用10m，它只增长10m，与VIRT相反 
    4、关于库占用内存的情况，它只统计加载的库文件所占内存大小 

SHR：shared memory 

    1、除了自身进程的共享内存，也包括其他进程的共享内存 
    2、虽然进程只使用了几个共享库的函数，但它包含了整个共享库的大小 
    3、计算某个进程所占的物理内存大小公式：RES – SHR 
    4、swap out后，它将会降下来 

DATA 

    1、数据占用的内存。如果top没有显示，按f键可以显示出来。 
    2、真正的该程序要求的数据空间，是真正在运行中要使用的。
