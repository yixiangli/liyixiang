---
title: Spring源码分析（一）
toc: true
date: 2017-04-22 17:14:23
category: 
    - 技术贴
    - 源码解读
tags: 
    - Spring

---

# Spring源码分析
源码分析不啰嗦，直接跟代码

## Bean加载

### ClassPathXmlApplicationContext
**类型：** 类
**作用：** 加载CLASSPATH下的Spring配置文件
**使用：**

```
ApplicationContext ac = new ClassPathXmlApplicationContext("classpath*:spring/applicationContext.xml");
ac.getBean(XXX.class);
```

一共两行代码，第一行对Bean进行加载，第二行就是获取Bean的实例，因此想要了解Bean加载流程，直接跟进ClassPathXmlApplicationContext
<!--more-->
**源码分析过程：**
1.构造方法：

```
public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent)
			throws BeansException {

		super(parent);
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();
		}
	}
```

三个步骤：
(1)super() 调用父类构造方法，ClassPathXmlApplicationContext的父类是AbstractXmlApplicationContext
(2)从方法名就大概能猜到，初始化一些config，深入代码发现主要功能集中在resolveRequiredPlaceholders方法，主要负责替换xml文件中的占位符，如：

```
<bean id="propertyPlaceholderConfigurer"   
        class="org.springframework.beans.factory.config.  
PropertyPlaceholderConfigurer">  
    <property name="locations">  
        <list>  
            <value>userinfo.properties</value>  
        </list>  
    </property>  
</bean>  
 
<bean name="userInfo" class="test.UserInfo">  
  <property name="username" value="${db.username}"/>  
  <property name="password" value="${db.password}"/>  
</bean> 
```

(3)做完了前期工作，接下来就是重头戏refresh()，refresh()方法才是整个Spring Bean加载的核心，直接进入源码（我先给部分注释按照下面分析过程翻译成中文）

```
public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			//准备刷新Spring上下文
			prepareRefresh();

			//获取刷新Spring上下文的Bean工厂
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				initMessageSource();

				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				onRefresh();

				// Check for listener beans and register them.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				finishRefresh();
			}

			catch (BeansException ex) {
				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}
		}
	}
```

首先分析，refresh方法中的代码被synchronized同步块包围，并且使用了对象锁startUpShutdownMonitor，而不是直接加在方法上，选中startUpShutdownMonitor发现，这个对象锁还在close()方法中使用到，见源码


```
public void close() {
		synchronized (this.startupShutdownMonitor) {
			doClose();
			// If we registered a JVM shutdown hook, we don't need it anymore now:
			// We've already explicitly closed the context.
			if (this.shutdownHook != null) {
				try {
					Runtime.getRuntime().removeShutdownHook(this.shutdownHook);
				}
				catch (IllegalStateException ex) {
					// ignore - VM is already shutting down
				}
			}
		}
	}
```

一看到shutdownHook，就想到了关闭钩子，这个概念在调研breeze热部署的时候遇到过，因此可以看出spring close操作也是基于java钩子，其次，startUpShutdownMonitor对象锁也保证了在调用refresh()方法的时候无法调用close()方法，避免了冲突。介绍完了整体，接下来个个击破。

#### prepareRefresh

```
protected void prepareRefresh() {
		this.startupDate = System.currentTimeMillis();

		synchronized (this.activeMonitor) {
			this.active = true;
		}

		if (logger.isInfoEnabled()) {
			logger.info("Refreshing " + this);
		}

		// Initialize any placeholder property sources in the context environment
		initPropertySources();

		// Validate that all properties marked as required are resolvable
		// see ConfigurablePropertyResolver#setRequiredProperties
		getEnvironment().validateRequiredProperties();
	}
```

1.startupDate:设置Spring上下文的开始时间
2.利用activeMonitor对象锁将active标识位设置成true
3.日志打印，这个this指的就是当前初始化的这个java类

在这里不得不提一个spring源码的精妙，在处理线程并发时大量使用了对象锁，而不是synchronized整体，因为使用对象锁可以减小同步的范围，只对不能并发的代码块进行加锁，提高了整体代码运行的效率。
特意强调一下initPropertySources()这个方法，

```
protected void initPropertySources() {
		// For subclasses: do nothing by default.
	}
```

跟代码会发现其实在本类AbstractApplicationContext中是个空实现，而且据了解initPropertySources()调用这一步也是在spring3.x之后的某个版本加上的，属于方法的扩展，而AbstractApplicationContext类为了做兼容，并不在本类方法中做实现，这样对于之前继承该类的类无影响，如果有些类需要实现initPropertySources()，重写便是。

#### obtainFreshBeanFactory

```
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
		refreshBeanFactory();
		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
		if (logger.isDebugEnabled()) {
			logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);
		}
		return beanFactory;
	}
```

看了这段代码，主要点在于以下两个方法
refreshBeanFactory()
getBeanFactory()

