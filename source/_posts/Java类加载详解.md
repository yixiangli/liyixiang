---
title: Java类加载详解
toc: true
date: 2016-09-13 15:31:35
category: 
	- 技术贴
	- Java
	- JVM
tags: 
    - JRE
    - 类加载
---

# 概述
最近准备尝试实现热部署功能，发现对于类加载的知识还是很欠缺的，赶紧补补课。

## 什么是类加载
对于学习java而言，程序员编写一个个的类是如何被运行起来的是一个很常见的问题，那么jvm就是通过类加载器来完成这一任务，类加载其实指的就是java源文件编译成.class文件后被加载到内存，并提供服务的过程。

### 生命周期
类的加载分为7个阶段
加载Loading —— 验证Verification —— 准备Preparation —— 解析Resolution —— 初始化Initialization —— 使用Using —— 卸载Unloading
<!--more-->
![类加载过程](/img/classLoader.png)

这里需要注意的是，加载－验证－准备－初始化－卸载这五个阶段的顺序是确定的，而解析阶段是不固定的，这是为了支持Java的动态绑定。而验证－准备－解析统称为连接Linking过程。

#### 加载
加载阶段完成三件事
1.通过一个类的全限定名称（其实就是完整的包名加类名）来获取定义此类的二进制字节流，这一步相当于是找到这个.class
2.将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
3.在内存中生成一个代表这个类的对象，作为方法区这个类的数据访问入口

重点说下第1点，随着技术的发展，获取二进制字节流的方式多种多样
1.从zip,jar,war等格式的包中读取
2.从Applet网络小应用中获取
3.运行时计算生成，动态代理技术之基础。生成形如$Proxy的代理类
4.其他文件中，如JSP

#### 验证
验证是连接阶段Linking阶段的第一步，这一阶段目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全
验证主要包括以下几个部分
1.文件格式验证
2.元数据验证
3.字节码验证
4.符号引用验证

#### 准备
正式为类变量分配内存并设置类变量初始值的阶段，注意这里说的只是包括类变量，也就是static修饰的变量，而不包括实例变量，如
```
public static int value = 123
```
但是要记得的是，在通常情况下，在准备阶段这个过程中，value的值为0而不是123，因为这时候尚未开始执行任何Java方法，赋值动作在初始化阶段才会执行，那有的读者会问了，为什么是0？这就是我们常说的基本数据类型的初始值
基本数据类型初始化表
数据类型   ｜  初始值
int       |  0
long      |  0l
short     |  (short)0
char      |  '\u0000'
byte      |  (byte)0
boolean   |  false
float     |  0.0f
double    |  0.0d
reference |  null

说完了通常情况，接下来说说特殊情况，那就是这个例子
```
public static final int value = 123
```
这个时候准备阶段的value的值就是123了
因为如果类字段的字段属性表中存在ConstantValue，也就是常量时，那么准备阶段变量value就会被初始化为ConstantValue属性所指定的值了。

#### 解析
##### 动态绑定
一句话：运行时确定类型
在类加载的过程中，规则就是解析可能在初始化之后进行。

#### 初始化
类加载过程的最后一步，这个阶段才真正开始执行类中定义的Java程序代码。
简单的说，初始化阶段就是执行类构造器<cinit>()方法的过程

##### 什么是<cinit>()方法
由编译器自动收集类中的所有类变量的赋值动作（咱们在准备阶段提到过）和静态语句块（static{}）中的语句合并产生，且严格按照源文件的顺序。

```
public class TestStaticInit {
	static {
		i = 0;							//编译可以正常通过
		System.out.println(i);			//非法向前引用
	}
	static int i =1;

}
```

### 类加载机的分类
<!--more-->
#### Bootstrap ClassLoader（引导类加载器）
作为JVM的一部分无法在应用程序中直接引用，由C/C++实现。
负责范围：
1. <JAVA>/jre/lib目录下特定的Jar(rt.jar)
2. -Xbootclasspath参数所指定的目录 
3. 系统属性sun.boot.class.path指定的目录中特定名称的jar包
在JVM启动时将通过Bootstrap ClassLoader加载rt.jar，并初始化sun.misc.Launcher从而创建Extension ClassLoader和System ClassLoader实例，和将System ClassLoader实例设置为主线程的默认Context ClassLoader（线程上下文加载器）

#### Extension ClassLoader（扩展类加载器）
从源码上看，Extension ClassLoader是加载器sun.misc.Launcher的静态内部类，因此仅含一个实例，由 sun.misc.Launcher$ExtClassLoader 实现，负责范围：
1. <JAVA_HOME>/jre/lib/ext目录 
2. 系统属性java.ext.dirs所指定的目录 中的所有类库。

#### App/System ClassLoader（系统类加载器）
与Extension ClassLoader相同，仅含一个实例，由 sun.misc.Launcher$AppClassLoader 实现，可通过 java.lang.ClassLoader.getSystemClassLoader 获取。负责范围：
1. 系统环境变量ClassPath
2. -cp 
3. 系统属性java.class.path 所指定的目录下的类库。

#### Custom ClassLoader（用户自定义类加载器）
继承ClassLoader,加载自定义的类

#### Context ClassLoader（线程上下文加载器）
默认为System ClassLoader，验证如下：

```
public class ClassLoaderTest {
    public static void main(String[] args) {
        // TODO Auto-generated method stub
        System.out.println(Thread.currentThread().getContextClassLoader());
        System.out.println(Thread.currentThread().getContextClassLoader().getParent());
        System.out.println(Thread.currentThread().getContextClassLoader().getParent().getParent());
    }
}
```


```
sun.misc.Launcher$AppClassLoader@234f79cb
sun.misc.Launcher$ExtClassLoader@36c51089
null
```

可以看出，Context ClassLoader默认为System ClassLoader，且父加载器为ExtClassLoader，上层加载器为BootStrap ClassLoader，由于BootStrap ClassLoader是C++生成，因此显示null,由此也引出一个概念－－－双亲委派模型

#### 双亲委派模型
    双亲委派模型要求除了顶层的启动类加载器外，其他的类加载器都应当有自己的父类加载器。这里类加载器之间的父子关系一般不会以继承关系来实现，而是都使用组合关系来复用父加载器的代码
    
工作过程：
    如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传递到顶层的启动类加载器中，
   只有当父类加载器反馈自己无法完成这个请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去加载
  
好处：
    Java类随着它的类加载器一起具备了一种带有优先级的层次关系。例如类Object，它放在rt.jar中，无论哪一个类加载器要加载这个类，最终都是委派给启动类加载器进行加载，因此Object类在程序的各种类加载器环境中都是同一个类，不会出现生成多个类。
    判断两个类是否相同是通过classloader.class这种方式进行的，所以哪怕是同一个class文件如果被两个classloader加载，那么他们也是不同的类
    
上面都是书本上的描述，写的也比较清楚了，来看看代码吧
    
    
#### 类加载器中的设计模式
模版方法

```
public abstract class ClassLoader {
    //这是一个重载方法
    public Class<?> loadClass(String name) throws ClassNotFoundException {
    return loadClass(name, false);
    }
    
    //这里就是父类算法的定义
    protected synchronized Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
    {
    Class c = findLoadedClass(name);
    if (c == null) {
        try {
        if (parent != null) {
            c = parent.loadClass(name, false);
        } else {
            c = findBootstrapClass0(name);
        }
        } catch (ClassNotFoundException e) {
            c = findClass(name);
        }
    }
    if (resolve) {
        resolveClass(c);
    }
    return c;
    }
    //这里留了一个方法给子类选择性覆盖
    protected Class<?> findClass(String name) throws ClassNotFoundException {
    throw new ClassNotFoundException(name);
    }

}
```

双亲委派模型
从代码上我们可以看出，在ClassLoader中定义的算法顺序是。
1.首先看是否有已经加载好的类。
2.如果父类加载器不为空，则首先从父类类加载器加载。
3.如果父类加载器为空，则尝试从启动加载器加载。
4.如果两者都失败，才尝试从findClass方法加载。












