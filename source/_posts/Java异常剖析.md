---
title: Java异常剖析
toc: true
date: 2017-02-12 17:38:05
category: 
	- 技术贴
	- Java
	- 基础
tags: 
    - 异常
    - Exception
---

# 异常
顾名思义，非正常的，在程序的世界里，非正常执行的例子不胜枚举，所以处理异常也作为每位程序猿们日常开发必须的课题。

## 异常分类
还记得博主人生第一次面试（当时是面去哪儿）就栽到这里了。
先发一张图
![异常结构图](/img/exception.jpg)

<!--more-->

## 异常该怎么处理
近来在看阿里刚出的Java开发手册，受益匪浅，尤其关于异常部分，博主总结了几点步骤，大家可以参考下
```
（1）在编写代码的过程中，一个方法体中应当明确哪段代码是稳定的（也就是不可能抛异常）哪段是不稳定的。
（2）对于不稳定的代码，如果是Exception类型，也就是编译时异常，一般Eclipse等开发工具就会提示你必须去处理（如IOException等）
（3）如果是RuntimeException，这类异常得让程序猿同学通过一些基本的校验去避免发生，如if(null != obj)等，而不是catch(NullPointerException)，而且不要由于一句代码可能存在空指针，就把这个方法整个try-catch住，这是极其不负责任的。
（4）自己try-catch处理还是throws给调用者处理，得看实际情况
    （3.1）如果是自己开发一个完整项目，一般web层需要定义统一的用户可理解的错误描述，那么下层都可以throws给controller层，统一转换成用户可理解的描述，返回给用户。
    （3.2）如果是接口类开发，强烈建议定好标准的接口返回码（ResponseCode），统一在api层转换成接口返回码以及返回信息。
    （3.3）不要臭不要脸的抛出RuntimeException，Exception以及Throwable
    （3.4）总之，最好在应用中自定义自己的标准异常类，提供多种构造方法，将可能发生的异常进行转换，最终在该处理异常的层去捕获并返回，如业界已定义的DaoException/ServiceException.
（5）重视finally，老生常谈，涉及到流／网络的关闭，finally非你不可，如果jdk7，可以考虑try-with-resources
（6）捕获异常与抛异常,必须是完全匹配,或者捕获异常是抛异常的父类，如果你抛了一个NullPointerException，结果别人catch一个IOException，是不是很蛋疼，当然他 catch Exception也是可以的。

```

### try-with-resources

```
/**
 * @developer liyixiang
 * @date 2017年2月12日
 * @since JDK 1.8
 * @function
 * 		一个try-with-resources的例子
 */
public class TwrTest {

	public void test() throws IOException{
		try(InputStream input = new FileInputStream("file.txt");
			BufferedInputStream bufferedInput = new BufferedInputStream(input)){
			
			int data = bufferedInput.read();
			while(data != -1){
				System.out.print((char) data);
				data = bufferedInput.read();
			}
		}
	}
}
```
原理就是使用了AutoClosable接口，写一个类实现一下
```
/**
 * @developer liyixiang
 * @date 2017年2月12日
 * @since JDK 1.8
 * @function
 */
public class AutoClose implements AutoCloseable {

	public void doSomeThing(){
		System.out.println("干了一件大事");
	}
	
	@Override
	public void close() throws Exception {
		// TODO Auto-generated method stub
		System.out.println("关闭流");
	}

}
```
测试一把
```
/**
 * @developer liyixiang
 * @date 2017年2月12日
 * @since JDK 1.8
 * @function
 */
public class TryTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		try {
			testClose();
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	
	public static void testClose() throws Exception{
		try(AutoClose ac = new AutoClose()){
			ac.doSomeThing();
		}
	}

}
```
结果
```
干了一件大事
关闭流
```
还有一点就是try-with-resources比程序员自己写try catch finally来close流的优势是，如果try-catch以及close时都有异常发生，他可以将try-catch中抛出的异常给addSuppressed到Throwable中，那么Throwable.getSuppressed就能取出try-catch中发生的异常了，而之前通过try catch finally,一旦close时出异常，那么前面的异常会被丢弃。

### 一个相对标准的自定义异常

```
/**
 * 
 * @author liyixiang
 * @date 2016年3月12日
 * @since JDK 1.7
 * @Function 服务异常 封装运行时异常
 */
public class BreezeException extends RuntimeException {

	/**
	 * @param 
	 */
	private static final long serialVersionUID = 1L;

	public BreezeException() {
        super();
    }
	
	public BreezeException(String message) {
        super(message);
    }
	
	public BreezeException(String message, Throwable cause) {
        super(message, cause);
    }
	
	public BreezeException(Throwable cause) {
        super(cause);
    }
	
}
```

### finally的一个知识点
finally与return语法执行顺序的例子
先看一个场景
```
/**
 * @developer liyixiang
 * @date 2017年2月12日
 * @since JDK 1.8
 * @function
 */
public class TestFinally {

	public static void main(String[] args) {
		System.out.println(functionA());
	}
	
	
	//@SuppressWarnings("finally")
	public static int functionA(){
		try {
			int b = 5/0;
			return b;
		}catch (Exception e){
			System.out.println(e);
		}finally {
			return 3;
		}
	}
}
```
程序的结果是3，打断点发现，int b = 5/0;抛出了异常后，就执行到catch中，然后执行finally中的return 3而没有执行try后的return b;
那么如果不抛异常呢？

```
/**
 * @developer liyixiang
 * @date 2017年2月12日
 * @since JDK 1.8
 * @function
 */
public class TestFinally {

	public static void main(String[] args) {
		System.out.println(functionA());
	}
	
	
	//@SuppressWarnings("finally")
	public static int functionA(){
		try {
			int b = 0;
			return b;
		}catch (Exception e){
			System.out.println(e);
		}finally {
			return 3;
		}
	}
}
```
程序的结果当然也是3，只是打断点发现先执行了return b,而后执行了return 3;
结论，总之绝大部分情况下不要在finally中用return就对了,Eclipse也提示了警告，finally block does not complete normally。


## 日志该怎么记
如果不加栈信息,只是new自定义异常,加入自己的理解的error message,对于调用 端解决问题的帮助不会太多。如果加了栈信息,在频繁调用出错的情况下,数据序列化和传输 的性能损耗也是问题。

### 日志分类
debug : 开发过程中的调试日志，千万不要在线上启用debug，否则gg。
并且debug日志在开发过程中加入，防止在线上输出，可以有如下方法

```
1.条件控制
if(logger.isDebugEnabled()){
	logger.debug("xxx");
}

2.占位符
logger.debug("aaaa:{}",a);

```
切记不要直接拼字符串。

warn : 警告，可以看作是一种警示，为了与info/error区别，一般可以用在用户输入等非致命性错误上，方便用户投诉时排查问题。

info : 普通日志，记录一些流程化，关键性的信息，可以理解为在线上程序打了断点，但千万不要随意打点，就像人表达观点，拣重点的说，不能啰嗦。

error :错误日志，这类级别只记录系统逻辑出错，异常等重要信息，可以单独输出到一个日志文件中，方便监控使用。

### 如何正确的记录异常信息
来看一个标准的demo，以logback为例

```
  <dependencies>
    <dependency> 
	    <groupId>org.slf4j</groupId> 
	    <artifactId>slf4j-api</artifactId> 
	    <version>1.7.12</version> 
	</dependency> 
	<dependency> 
	    <groupId>org.slf4j</groupId> 
	    <artifactId>jcl-over-slf4j</artifactId> 
	    <version>1.7.12</version> 
	</dependency> 
	<dependency> 
	    <groupId>ch.qos.logback</groupId> 
	    <artifactId>logback-core</artifactId> 
	    <version>1.1.3</version> 
	</dependency> 
	<dependency> 
	    <groupId>ch.qos.logback</groupId> 
	    <artifactId>logback-classic</artifactId> 
	    <version>1.1.3</version> 
	</dependency>
  </dependencies>
```
在开发中使用log必须通过slf4j提供的统一门面接口来构造日志，这样做的好处就是有利于兼容不同日志处理方式，另外在异常记录中一定要注意，采用正确的方法。

```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * @developer liyixiang
 * @date 2017年3月6日
 * @since JDK 1.8
 * @function
 */
public class LoggerRecord {

	private static final Logger log = LoggerFactory.getLogger(LoggerRecord.class);
	
	public static void catchExceptionFunction(){
		try {  
		    Integer x = null;  
		    ++x;  
		} catch (Exception e) {  
		    //log.error(e);        //A  
		    //log.error(e, e);        //B  
		    log.error("" + e);        //C  
		    log.error(e.toString());        //D  
		    log.error(e.getMessage());        //E  
		    log.error(null, e);        //F  
		    log.error("", e);        //G  
		    log.error("{}", e);        //H  
		    log.error("{}", e.getMessage());        //I  
		    log.error("Error reading configuration file: " + e);        //J  
		    log.error("Error reading configuration file: " + e.getMessage());        //K  
		    log.error("Error reading configuration file", e);        //L  
		}
	}
	
	public static void main(String[] args) {
		LoggerRecord.catchExceptionFunction();
	}
}
```

运行一下，结果是
```
C:
14:43:20.861 [main] ERROR com.li.leader.Logger.LoggerRecord - java.lang.NullPointerException
D:
14:43:20.864 [main] ERROR com.li.leader.Logger.LoggerRecord - java.lang.NullPointerException
E:
14:43:20.864 [main] ERROR com.li.leader.Logger.LoggerRecord - null
F:
14:43:20.870 [main] ERROR com.li.leader.Logger.LoggerRecord - null
java.lang.NullPointerException: null
	at com.li.leader.Logger.LoggerRecord.catchExceptionFunction(LoggerRecord.java:26) [classes/:na]
	at com.li.leader.Logger.LoggerRecord.main(LoggerRecord.java:54) [classes/:na]
G:
14:43:20.871 [main] ERROR com.li.leader.Logger.LoggerRecord - 
java.lang.NullPointerException: null
	at com.li.leader.Logger.LoggerRecord.catchExceptionFunction(LoggerRecord.java:26) [classes/:na]
	at com.li.leader.Logger.LoggerRecord.main(LoggerRecord.java:54) [classes/:na]
H:
14:43:20.871 [main] ERROR com.li.leader.Logger.LoggerRecord - {}
java.lang.NullPointerException: null
	at com.li.leader.Logger.LoggerRecord.catchExceptionFunction(LoggerRecord.java:26) [classes/:na]
	at com.li.leader.Logger.LoggerRecord.main(LoggerRecord.java:54) [classes/:na]
I:
14:43:20.871 [main] ERROR com.li.leader.Logger.LoggerRecord - null
J:
14:43:20.873 [main] ERROR com.li.leader.Logger.LoggerRecord - Error reading configuration file: java.lang.NullPointerException
K:
14:43:20.873 [main] ERROR com.li.leader.Logger.LoggerRecord - Error reading configuration file: null
L:
14:43:20.873 [main] ERROR com.li.leader.Logger.LoggerRecord - Error reading configuration file
java.lang.NullPointerException: null
	at com.li.leader.Logger.LoggerRecord.catchExceptionFunction(LoggerRecord.java:26) [classes/:na]
	at com.li.leader.Logger.LoggerRecord.main(LoggerRecord.java:54) [classes/:na]
```

从结果上看，F/G/H/L可以完整输出异常信息，但是仔细查看会发现，F/L只是把第一个参数输出来，包括null和{}占位符都是原样输出，因此，最标准的还是使用G或者L,也就是

```
log.error("", e);        //G
log.error("Error reading configuration file", e);        //L
```

## 常见异常分析

### Java Exception
1.解决 - java.lang.OutOfMemoryError： unable to create new native thread

```
这个Exception是一次同事在做全量数据同步时，每条消息都需要创建一个线程，并向下游发消息导致，而且在本地Mac上跑的case，因此，一般出现这个问题的本质原因是我们创建了太多的线程，而能创建的线程数是有限制的，导致了异常的发生。
```
解决方案：

首先我们来看一下一个应用进程能创建的线程数的具体计算公式： 

```
(MaxProcessMemory - JVMMemory - ReservedOsMemory) / (ThreadStackSize) = Number of threads 

MaxProcessMemory 	指的是一个进程的最大内存
JVMMemory         	JVM内存
ReservedOsMemory  	保留的操作系统内存
ThreadStackSize     线程栈的大小
```

这里解释下公式：
	在java语言里， 当你创建一个线程的时候，虚拟机会在JVM内存创建一个Thread对象同时创建一个操作系统线程，而这个系统线程的内存用的不是JVMMemory，而是系统中剩下的内存(MaxProcessMemory - JVMMemory - ReservedOsMemory)。 
	
另外值得我们深思的是，何时会抛出OutOfMemoryException，其实并不是内存被耗空的时候才抛出
1.JVM98%的时间都花费在内存回收
2.每次回收的内存小于2%
   满足这两个条件将触发OutOfMemoryException，这将会留给系统一个微小的间隙以做一些Down之前的操作，比如手动打印Heap Dump或者JVM启动参数中加入-Xrunprof:heap=sites参数，但是对JVM性能影响非常大，不建议在线上服务器环境使用。

```
javac -J-agentlib:hprof=heap=dump Hello.java
```
### 堆溢出的分析与解决方案
#### 内存泄漏
一般指本不应该存活的对象由于到GC Roots还存在路径而使垃圾收集器无法回收
解决方案：使用工具查看泄漏对象到GC Roots的引用链。

#### 内存溢出
内存中的对象确实都必须存活
解决方案：1.尝试调大堆参数 2.代码上检查是否存在某些对象生命周期过长，持有状态时间过长。

### 栈溢出的分析与解决方案
栈溢出的两种场景：
1.线程请求的栈深度大于虚拟机所允许的最大深度，则StackOverflowError
2.如果虚拟机在扩展栈时无法申请到足够的内存空间，则OutOfMemoryError

单线程：
(1)－Xss减少栈空间内存容量 StackOverflowError
(2)定义大量本地变量 StackOverflowError

多线程：
java.lang.OutOfMemoryError： unable to create new native thread

内存的总大小 ＝ 堆大小 ＋ 方法区大小 ＋ 栈大小
而栈大小由每个栈的内存空间＊线程数决定，因此如果每个线程的栈空间较大，系统创建的线程数就会减少。

### 系统CPU 100%
1.top  观察cpu使用率 找到100%的pid 如 28857
2.top -H -p 28857 根据pid查看相应的线程运行状态信息
3.jstack 线程id 查看该线程运行情况

### 类加载相关问题
ClassNotFoundException一般发生在显式类加载；
NoClassDefFoundError一般发生在隐式加载；
ClassCastException一般发生在jar包冲突，比如某个jar包已经被更上层的加载器加载了，但你却要求他强制转为下层加载器加载的同名类
