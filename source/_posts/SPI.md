---
title: SPI
toc: true
date: 2017-02-10 17:06:16
category: 
    - 技术贴
    - 理论与概念
tags: 
    - spi 类加载
---

# 一段废话
博主之前在研究dubbo时，发现dubbo中有关于spi的设计理念，一直没有太明白，最近看类加载时突然看到spi,所以就仔细研究一下

## 什么是SPI
简写：Service Provider Interface，服务提供接口
在面向对象编程中，面向接口编程应当成为一种习惯，优秀的接口定义对于系统扩展性有着很大的帮助，提及扩展性，就自然引入了spi的概念。某天，你接到一个开发任务，需要提供一个crud的服务，那么首先就要定义接口，编写实现类，经过时间的推移，现有实现类无法满足需求，那么现在有两种方案，1.将现有接口做逻辑兼容，2.派生出一个新类实现接口，编写新功能，很显然，第二种方案更好，尤其是针对客户端直接调用的web接口，还必须维护老版本的功能。那么客户端在使用新旧接口时就可以利用spi的特性。
因此spi的基本定义如下：
    为了实现在模块装配的时候能不在程序里动态指明，这就需要一种服务发现机制。Java spi就是提供这样的一个机制：为某个接口寻找服务实现的机制。有点类似IOC的思想，就是将装配的控制权移到程序之外，在模块化设计中这个机制尤其重要。

## 怎么用
<!--more-->
直接跟着我看代码，首先我定义了一个接口

```
package com.li.leader.spi;

/**
 * @developer liyixiang
 * @date 2017年2月10日
 * @since JDK 1.8
 * @function
 * 		简单的基于spi的接口
 */
public interface SimpleSpiInterface {

	public void insert();
	
	public void query();
}
```
有了接口就会有实现类
```
/**
 * @developer liyixiang
 * @date 2017年2月10日
 * @since JDK 1.8
 * @function
 */
public class SimpleSupport implements SimpleSpiInterface {

	@Override
	public void insert() {
		// TODO Auto-generated method stub
		System.out.println("插入了一条数据");
	}

	@Override
	public void query() {
		// TODO Auto-generated method stub
		System.out.println("查询出了一条数据");
	}

}
```
现在需求迭代了，旧的接口需要升级
```
/**
 * @developer liyixiang
 * @date 2017年2月10日
 * @since JDK 1.8
 * @function
 */
public class OtherSimpleSupport implements SimpleSpiInterface {

	@Override
	public void insert() {
		// TODO Auto-generated method stub
		System.out.println("又插入了一条数据");
	}

	@Override
	public void query() {
		// TODO Auto-generated method stub
		System.out.println("又查询出了一条数据");
	}

}
```
开发工作完毕，交付给外部使用，如何用呢

```
/**
 * @developer liyixiang
 * @date 2017年2月10日
 * @since JDK 1.8
 * @function
 *		spi加载器
 */
public class SPILoader {

	public static void main(String[] args) {
		//SimpleSpiInterface ssi = 
		ServiceLoader<SimpleSpiInterface> sl = ServiceLoader.load(SimpleSpiInterface.class);
		Iterator<SimpleSpiInterface>	iterator = sl.iterator();  
	    if (iterator.hasNext()) { 
	    	SimpleSpiInterface ss = iterator.next();  
	    	ss.query();  
	    } 
	}
}
```

代码都写完了，还需要进行spi的配置，spi是一种java提供的接口服务规则，约定当服务的提供者，提供了服务接口的一种实现之后，在jar包的META-INF/services/目录里同时创建一个以服务接口命名的文件。该文件里就是实现该服务接口的具体实现类。而当外部程序装配这个模块的时候，就能通过该jar包META-INF/services/里的配置文件找到具体的实现类名，并装载实例化，完成模块的注入。那么针对我这个例子就是 

```
1.新建resources/META-INF/services/com.li.leader.spi.SimpleSpiInterface
2.配置com.li.leader.spi.support.OtherSimpleSupport 或者SimpleSupport
3.运行SPILoader main函数
```

最终，就会通过打印看出结果，详细解释一下加载流程，ServiceLoader是jdk提供的一个查找提供服务的工具类，配置都必须按照规范（规范已说明）来写，打开ServiceLoader会发现，其实是利用了ContextClassLoader加载器完成的加载，这么做的好处就是将装配的过程移到程序之前，不再需要程序内部动态加载使用的类。

## dubbo中的spi
dubbo的文档写的很详细，详见http://dubbo.io/  开发文档中的扩展点加载部分。
