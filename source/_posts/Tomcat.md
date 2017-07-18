---
title: Tomcat
toc: true
date: 2017-03-20 17:16:11
category: 
	- 技术贴
	- web中间件
	- Tomcat
tags: 
    - Tomcat
---

# what is Tomcat

## 目录结构
### 5.x及以下
/bin：存放windows或Linux平台上启动和关闭Tomcat的脚本文件
/conf：存放Tomcat服务器的各种全局配置文件，其中最重要的是server.xml和web.xml
/doc：存放Tomcat文档
/server：包含三个子目录：classes、lib和webapps
/server/lib：存放Tomcat服务器所需的各种JAR文件
/server/webapps：存放Tomcat自带的两个WEB应用admin应用和 manager应用
/common/lib：存放Tomcat服务器以及所有web应用都可以访问的jar文件
/shared/lib：存放所有web应用都可以访问的jar文件（但是不能被Tomcat服务器访问）
/logs：存放Tomcat执行时的日志文件
/src：存放Tomcat的源代码
/webapps：Tomcat的主要Web发布目录，默认情况下把Web应用文件放于此目录
/work：存放JSP编译后产生的class文件

### 6.x
/bin，/conf，/logs，/webapps，/work：
这几个目录没有变化，主要变化的是将
/server /common /shared 三个目录合成了/lib一个目录，下面就具体说说这些目录的由来

#### Tomcat的类加载器
我们知道JDK默认提供的类加载器有启动类加载器Bootstrap ClassLoader，其子类Extension ClassLoader以及子类的子类Application ClassLoader。而CommonClassLoader,CatalinaClassLoader,SharedClassLoader和WebappClassLoader则是Tomcat自己定义的类加载器，它们分别加载/server /common /shared 以及/Webapp/WEB-INF/四个目录下的java类库，这样做的目的是为了对多个应用依赖的类库进行共享与隔离，解决应用以及Tomcat本身安全性的同时，节约资源空间。
盗用网上两张图做对比
** 5.x **
![Tomcat5.x类加载结构](/img/Tomcat5.x.jpeg)

** 6.x **
![Tomcat6.x类加载结构](/img/Tomcat6.x.jpeg)

从图上可以看出，Tomcat也是严格按照经典的双亲委派模型来实现，CommonClassLoader能加载的类都能被SharedClassLoader和CatalinaClassLoader使用，而SharedClassLoader和CatalinaClassLoader自己能加载的类则与对方相互隔离。WebappClassLoader可以使用SharedClassLoader加载的类，且WebappClassLoader各个实例相互隔离。

** 6.x的变化 ** 
对于6.x的版本，只有指定了/conf/catalina.properties配置文件的server.loader和share.loader项后才会真正建立CatalinaClassLoader和SharedClassLoader，否则这两个加载器会被CommonClassLoader实例替代，且默认不使用，因此把/server /common /shared 三个目录合成了/lib一个目录，这是Tomcat设计团队为了简化大多数部署场景所做的一项改进，当然也没有完全摈弃5.x的设计，用户可以通过配置启用5.x的加载器结构。

## 配置


### 端口详解
8080:基于HTTP协议的对外服务端口，负责建立HTTP连接
8005:shutdown进程，用于关闭tomcat，可以理解为一个tomcat的守护进程
8009:基于AJP协议的对外服务端口，负责和其他的HTTP服务器建立连接
8443:基于HTTPS协议的对外服务端口，负责建立HTTPS连接


<!--more-->
## 优化
### jdk优化
Tomcat默认可以使用的内存为128MB，对于部署大型应用来说肯定是不够的，那么对于linux系统，在文件/bin/catalina.sh的前面，增加如下设置： 
```
JAVA_OPTS=’-Xms【初始化内存大小】 -Xmx【可以使用的最大内存】 -XX:PermSize=64M -XX:MaxPermSize=128m’ 需要把几个参数值调大。

例如： JAVA_OPTS=’-Xms256m -Xmx512m’ 表示初始化内存为256MB，可以使用的最大内存为512MB。
```

#### 参数详解
-server  启用jdk 的 server 版；
-Xms    java虚拟机初始化时的最小内存；
-Xmx    java虚拟机可使用的最大内存；
-XX:PermSize    内存永久保留区域
-XX:MaxPermSize   内存最大永久保留区域 
-Xmn    jvm最小内存

### 线程优化
在tomcat配置文件server.xml中的配置中，和连接数相关的参数有：

maxThreads： Tomcat使用线程来处理接收的每个请求。这个值表示Tomcat可创建的最大的线程数。默认值150。
acceptCount： 指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理。默认值10。
minSpareThreads： Tomcat初始化时创建的线程数。默认值25。
maxSpareThreads： 一旦创建的线程超过这个值，Tomcat就会关闭不再需要的socket线程。默认值75。
enableLookups： 是否反查域名，默认值为true。为了提高处理能力，应设置为false
connnectionTimeout： 网络连接超时，默认值60000，单位：毫秒。设置为0表示永不超时，这样设置有隐患的。通常可设置为30000毫秒。
maxKeepAliveRequests： 保持请求数量，默认值100。 bufferSize： 输入流缓冲大小，默认值2048 bytes。
compression： 压缩传输，取值on/off/force，默认值off。 其中和最大连接数相关的参数为maxThreads和acceptCount。如果要加大并发连接数，应同时加大这两个参数。

```
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000" maxThreads="1000" minSpareThreads="60" maxSpareThreads="600"  acceptCount="120" 
               redirectPort="8443" URIEncoding="utf-8"/>
```