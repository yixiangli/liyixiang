---
title: Java开发之常见问题
toc: true
date: 2016-09-12 17:07:03
category: 
	- 技术贴
	- Java
tags: 
    - Java
    - Eclipse
    - Maven

---

# 概述
作为一名开发狗，难免会在coding的过程中遇到各种各样的问题，那就做个整理，记录问题与解决方案

## 常见问题
<!--more-->
### Maven
1.maven install时

```
Please ensure you are using JDK 1.4 or above and [ERROR] not a JRE (the com.sun.tools.javac.Main class is required）
```

原因：
Eclipse/MyEclipse默认是使用jre作为运行环境，而maven编译需要jdk作为运行环境 ;

解决方案：
```
进入eclipse properties设置，点击Java Build Path，选中JRE System Library,将JRE环境改为workspace default JRE,当然，eclipse默认的JRE环境各位要提前配置好。
```

2.Maven Problems

```
项目文件夹有个红叉叉，让人看着很恶心
```

原因：项目Web Module配置较低，一般出现在web project中，可点击项目右键——> properties ——>Project Facets 查看Dynamic Web Module是否为新版本，一般默认是2.3 正常版本是3.0

解决方案：
```
直接在Project Facets的Dynamic Web Module中修改并没有用，需要在window——> Show view——> Navigator，进入右侧Navigator界面，找到所在项目，进入.settings文件夹——>org.eclipse.wst.common.project.facet.core.xml文件，修改<installed facet="jst.web" version="3.0"/>即可
```

3.jdk version
```
项目文件夹有个红叉叉，看了下错误是Type Java compiler level does not match the version of the install
```

原因：eclipse与项目本身jdk版本不一致导致

解决方案：
```
1.在项目上右键properties->project Facets->修改右侧的version  保持一致
2.preferences->java->Compiler->设置右侧的Compiler compliance level
3.项目本身引用的JRE System Library
```

### Java Errors
1.xxx cannot be resolved to a type 错误解决方法

```
开发项目的时候，有可能突然Eclipse抽风，一些外部接口类import进来后报错，可能代码都没动过，突然某天出来。
```
原因：据网上描述有可能存在Jar冲突或者jdk版本问题，但是如果确认这两部分没问题的话，那就可能是由于某些特殊原因，eclipse没能自动编译源代码到build/classes（或其他classes目录），导致类型查找不到。

解决方案：
```
点击导航栏Project—->选中Clean，选择出现这种问题的项目。 
```

2.There is an error in invoking javac.  A full JDK (not just JRE) is required 

```
新建一个工程后，访问index.jsp,时报500
```

解决方案：
```
<dependency>
	<groupId>org.mortbay.jetty</groupId>
	<artifactId>jsp-2.1-glassfish</artifactId>
	<version>9.1.02.B04.p0</version>
</dependency> 
```


### Mac env
1.cd Internet Plug-Ins — No such file or directory

```
Mac下中java目录会被放到一个叫Internet Plug-Ins的目录下，看到了吧，这目录还带空格，怎么(cd)处理呢？
```

解决方案：
```
cd Internet\ Plug-Ins/ 
```

### Mac Eclipse
1.设置工作空间编码
```
Mac下使用Eclipse新建一个工作空间，默认编码是EUC_CN，可以通过选中项目后右键，Properties中修改，但是每个子项目都要修改很麻烦
```

解决方案：
```
选择偏好设置－－> General－－> WorkSpace
```

2.设置行宽度
```
程序狗都有代码格式的癖好，博主的癖好就是不喜欢把一段代码分行写，就喜欢一句话要说话，不要遮遮掩掩。
```
解决方案：
```
Formatter－－> HTML & Java 
设置line width属性  一般设置999就能保证在一行了
```

3.failed create the java virtual machine
```
点击Eclipse出现这个错误，一般是机器内存不够用，eclipse带不动了或者是jre环境没有配置好
```
解决方案：
```
（1）机器内存不够用：修改eclipse.ini的参数 找到这两处
--launcher.XXMaxPermSize
256m
-XX:MaxPermSize=256m
-Xms40m
-Xmx512m
将jvm持久代参数设置调小（eclipse也占不了那么大持久代），如128m，然后重启
（2）jre环境没有配置好
在-vmargs配置的上方加上
-vm
/Library/Java/JavaVirtualMachines/jdk1.8.0_111/Contents/Home
重启
```

4.run Java Application 报错，error=2,no such file or directory  
```
文件路径不存在或不合法
```
解决方案：
```
（1）认真检查下该路径是否有java文件
（2）mac系统切记，路径中不能有中文!
```