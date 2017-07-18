---
title: Java关键字
toc: true
date: 2016-12-19 15:02:43
category: 
	- 技术贴
	- Java
	- 基础
tags: 
    - 关键字
---


# Java关键字

Java语言规定关键字不能作为标识符。目前共有50个Java关键字，其中，"const"和"goto"这两个关键字在Java语言中并没有具体含义。
<!--more-->

- 作用域
    - public
    - private 
    - protected
    - import
    - package
- 异常
    - try
    - throw
    - throws
    - catch
    - finally
- 结构
    - extends
    - implements
    - class
    - interface
    - this
    - super
    - instanceof
    - new
    - enum
- 数据类型
    - byte
    - short
    - char
    - int
    - double
    - float
    - boolean
    - null
- 修饰符
    - abstract
    - static
    - transient
    - volatile
    - void 
    - final
    - native
    - synchronized
- 条件语言
    - for
    - do
    - while
    - switch
    - case
    - default
    - if
    - else
    - return 
    - break
    - continue
- 不常用
    - goto 
    - assert
    - const
    - strictfp
      
- - -    

## 重点关键字详解
### volatile
概念：修饰的变量jvm只保证从主内存加载到工作线程中的值是最新的
关键点：无法解决并发问题 并发问题还是需要使用concurrent包
作用：volatile变量具有可见性特性，在一些有限的情形下（如读远远大于写）使用volatile代替锁，避免锁带来的线程阻塞以及提高可伸缩性。
````
可伸缩性：
当增加计算资源时（如cpu,内存，存储容量或I/O带宽），程序的吞吐量或者处理能力能相应地增加
````
来看个代码，热乎的，完整的诠释了volatile的重要性
````
public class ConcurrentTest {
	
	//这个volatile一定要加 不加while(true) 存在空跑
	volatile boolean lock ;
    
    public boolean isLock() {
        return lock;
    }

    public void setLock(boolean lock) {
        this.lock = lock;
    }

	public static void main(String[] args) throws InterruptedException {
		// TODO Auto-generated method stub
		final Set<String> instanceSet = Collections.synchronizedSet(new HashSet<String>());
        final ConcurrentTest lock = new ConcurrentTest();
        lock.setLock(true);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < 100; i++) {
            executorService.execute(new Runnable() {
                
                public void run() {
                    while (true) {
                    	//不知为何加这一行代码 程序就可以运行了！
                    	//System.out.println(lock.isLock());
                        if (!lock.isLock()) {
                        	//System.out.println(Thread.currentThread().getName());
                            SimpleSingleTon singleton = SimpleSingleTon.getInstance();
                            instanceSet.add(singleton.toString());
                            break;
                        }
                    }
                }
            });
        }
        Thread.sleep(5000);
        lock.setLock(false);
        Thread.sleep(5000);
        System.out.println("------并发情况下我们取到的实例------");
        System.out.println(instanceSet.size());
        for (String instance : instanceSet) {
            System.out.println(instance);
        }
        executorService.shutdown();
	}

}
````
背景：
    这是一段测试普通单例模式在并发下是否还能保持一个实例的例子
如果lock不加volatile修饰，那么在部分机器上运行（如博主的MacBook）set集合中实例大小一直是0，原因就是当lock.setLock(false)后对于while(true)里死循环的线程们还是从自己的线程栈中获取lock值，尽管内存中的lock已经改变，针对这种情况，volatile就派上大用了，结果一运行可知。

### transient
概念：瞬时
关键点：一般用于序列化（serialization）与反序列化（deserialization）
作用：被修饰的变量不会被序列化，一种语言级的标记方法。


### synchronized
**互斥性**
	synchronized是对类的当前实例进行加锁，防止其他线程同时访问该类的该实例的所有synchronized块，而static synchronized恰好就是要控制类的所有实例的访问，static获取到的锁，属于类锁，而非static方法获取到的锁，是属于当前对象的锁，因此如果多线程访问不同实例的一个static方法和一个非static方法，static方法会阻塞。
	
几个场景：
1.线程A访问非static同步方法，线程B可以访问其他非static同步方法
2.线程A访问非static同步方法，线程B可以访问其他static同步方法
3.线程A访问static同步方法，线程B可以访问非其他static同步方法
4.线程A访问static同步方法，线程B不能访问其他static同步方法
    
总结一下就是：
static方法是类锁，非static是对象锁，类锁和对象锁不会产生互斥

