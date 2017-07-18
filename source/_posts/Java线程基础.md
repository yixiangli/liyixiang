---
title: Java线程基础
toc: true
date: 2017-02-13 11:06:23
category: 
	- 技术贴
	- Java
	- 基础
tags: 
    - 线程
    - run
---

# 什么是线程

## 线程状态
new, runnable, blocked, waiting, time waiting, 或 terminated

![线程状态转换图](/img/thread_state.jpg)

**Blocked**
一般只有synchronized这种monitor锁才会出现这种状态

**Waiting**
无超时的等待，无法自动恢复，必须notify或者Interrupt信号
ReentrantLock.lock()的无参方法调用也会Waiting

**Time_waiting**
有超时的等待，sleep wait join等指定超时值。

### interrupt&interrupted&isInterrupted
interrupt是一个用于设置线程中断标志的方法，阅读源码发现实际上调用了一个interrupt0的native方法来中断当前线程，这里注意只是将这个线程设置成中断状态（这个状态和java线程基本状态无关）,运行这个例子就很清楚了
<!--more-->
```
/**
 * @developer liyixiang
 * @date 2017年2月13日
 * @since JDK 1.8
 * @function
 */
public class InterruptThread extends Thread{

	private boolean stop = false;
	
	public static void main(String[] args) throws Exception {
		InterruptThread thread = new InterruptThread();
        System.out.println( "Starting thread..." );
        thread.start();
        Thread.sleep( 3000 );
        System.out.println( "Interrupting thread..." );
        thread.interrupt();
        Thread.sleep( 3000 );
        System.out.println("Stopping application..." );
        //System.exit(0);
	}
	
	@Override
	public void run() {
		// TODO Auto-generated method stub
		while(!stop){
			System.out.println( "Thread is running..." );
			System.out.println(Thread.currentThread().getState());
            long time = System.currentTimeMillis();
            while((System.currentTimeMillis()-time < 1000)) {
            }
            System.out.println("Thread exiting under request..." );
		}
	}
}
```

那么有了这个标志，如何获取或者做相关操作呢？这就用到了interrupted方法和isInterrupted方法。
interrupted是一个静态方法，作用于当前线程，调用native的isInterrupted方法，参数传true,意为返回线程的状态位后，要清掉原来的状态位（恢复成原来情况（非中断））。
isInterrupted是一个普通方法，作用于调用该方法的线程对象所对应的线程，也是调用native的isInterrupted方法，参数传false,只是返回线程的状态位。

中断是通过调用Thread.interrupt()方法来做的. 这个方法通过修改了被调用线程的中断状态来告知那个线程, 说它被中断了. 对于非阻塞中的线程, 只是改变了中断状态, 即Thread.isInterrupted()将返回true; 对于可取消的阻塞状态中的线程, 比如等待在这些函数上的线程, Thread.sleep(), Object.wait(), Thread.join(), 这个线程收到中断信号后, 会抛出InterruptedException, 同时会把中断状态置回为false.

### ThreadLocal
作用：解决多线程场景下，相同变量的访问冲突问题，保证线程安全使用。

适用场景：
	ThreadLocal不是用来解决对象共享访问问题的，ThreadLocal的应用场合，我觉得最适合的是按线程多实例（每个线程对应一个实例）的对象的访问，并且这个对象很多地方都要用到。

原理：
    当使用ThreadLocal维护变量时，ThreadLocal为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。ThreadLocal是如何做到为每一个线程维护变量的副本的呢？其实实现的思路很简单：在ThreadLocal类中有一个ThreadLocalMap，用于存储每一个线程的变量副本，Map中元素的键为ThreadLocal对象，而值对应线程的变量副本。
详见get源码   
   
    
```
public T get() {
        Thread t = Thread.currentThread();
        //通过当前线程获取到ThreadLocalMap
        //而ThreadLocalMap维护的是一个Entry对象
        //key:ThreadLocal实例 value副本变量
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
``` 

 
常用方法：
    jdk5.0之后支持泛型，主要方法为void set(T value)、T get()以及T initialValue()。一定要注意initialValue(),本身是一个protected方法，所以就是为子类覆盖所用，如果不实现，默认返回null，且该方法只会执行一次，执行时机在线程第1次调用get()或set(Object)时才执行
 
使用：
    通过匿名内部类的方式定义ThreadLocal的子类。

```
     private static ThreadLocal<Date> startDate= new ThreadLocal<Date>() {
		protected Date initialValue() {
			// TODO Auto-generated method stub
			return new Date();
		}
	};
```
 
## 一个JAVA进程究竟能创建多少线程
这是一个常见的问题，很多面试或者工作中使用线程池都会遇到这个问题，正巧博主看了一篇文章，写的很不错，就吸收借鉴一下，做个总结

### 影响线程创建的因素
** JVM：**
Xmx
MaxPermSize
MaxDirectMemorySize
ReservedCodeCacheSize

java中创建线程的方式一般是new Thread().start()
而真正创建线程并不是new Thread()，而是start时调用native方法start0,而start0方法中会检验当前内存情况，因此如：Xmx MaxPermSize等JVM参数都会影响剩余的内存。

Xss
这个参数本身就是控制线程堆栈的内存大小，因此也会有影响。

** Kernel：**
max_user_processes
创建线程会判断进程数有多少，而在linux中，进程与线程使用的数据结构相同，所以进程数可以理解为轻量级线程数，ulimit -u命令可以查到，所以如果当前用户起的线程数超过了这个限制，那肯定是不会创建线程成功的

max_map_count
在底层分配malloc时会判断，如果进程被分配的内存段超过sysctl_max_map_count就会失败，而这个值在linux下对应/proc/sys/vm/max_map_count，默认值是65530，可以通过修改上面的文件来改变这个阈值。

max_threads
同上分析，/proc/sys/kernel/threads-max控制这个参数

pid_max
同上分析，/proc/sys/kernel/pid_max控制这个参数

### 线程中的异常处理
在多线程开发中，一个常见的问题就是在线程run中存在异常，实际上是抛不到主线程上，或者说主线程catch不到，如何处理呢？

** 方案一： **
在不使用线程池或者使用的是线程池的execute方法时，Thread类提供了一个接口UncaughtExceptionHandler，可以捕获线程中抛出的异常,给个demo

```
class CatchExc implements UncaughtExceptionHandler {

	@Override
	public void uncaughtException(Thread t, Throwable e) {
		// TODO Auto-generated method stub
		System.out.println("This is:" + t.getName() + ",Message:"
		        + e.getMessage());
		    //e.printStackTrace();
	}	
}

设置UncaughtExceptionHandler
Thread t = new Thread(new ThreadOne());
t.setUncaughtExceptionHandler(new CatchExc());
t.start();
```

** 方案二 **
ExecutorService submit方法
返回一个Future对象，调用该对象的get()方法的时候再次抛出线程中的异常

### Executor

![类结构图](/img/executor.png)

从图上看好像少了Executors类，深入源码发现Executors类主要用于提供线程池相关的操作，是一个无状态的类，可以理解为一个工具类，所有方法都是static，且构造方法私有，属于无实例的类。
ThreadPoolExecutor提供了多种参数配置，Executors对其进行了包装，形成了newSingleThreadExecutor，newFixedThreadPool，newCachedThreadPool等多种线程池，通过源码一读便知。


#### ThreadPoolExecutor

```
public class ThreadPoolExecutor extends AbstractExecutorService {
    .....
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,RejectedExecutionHandler handler);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
        BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler);
    ...
}
```

ThreadPoolExecutor 的内部工作原理，整个思路总结起来就是 5 句话：
(1) 如果当前池大小 poolSize 小于 corePoolSize ，则创建新线程执行任务。
(2) 如果当前池大小 poolSize 大于 corePoolSize ，且等待队列未满，则进入等待队列
(3) 如果当前池大小 poolSize 大于 corePoolSize 且小于 maximumPoolSize ，且等待队列已满，则创建新线程执行任务。
(4) 如果当前池大小 poolSize 大于 corePoolSize 且大于 maximumPoolSize ，且等待队列已满，则调用拒绝策略来处理该任务。
(5) 线程池里的每个线程执行完任务后不会立刻退出，而是会去检查下等待队列里是否还有线程任务需要执行，如果在 keepAliveTime 里等不到新的任务了，那么线程就会退出。

**其他参数解释：**
keepAliveTime：表示线程没有任务执行时最多保持多久时间会终止。默认情况下，只有当线程池中的线程数大于corePoolSize时，keepAliveTime才会起作用，直到线程池中的线程数不大于corePoolSize，即当线程池中的线程数大于corePoolSize时，如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize。但是如果调用了allowCoreThreadTimeOut(boolean)方法，在线程池中的线程数不大于corePoolSize时，keepAliveTime参数也会起作用，直到线程池中的线程数为0；

workQueue：一个阻塞队列，用来存储等待执行的任务，排队有三种通用策略：

**直接提交**工作队列的默认选项是SynchronousQueue，这个队列比较特殊，它不会保存提交的任务，而是将直接新建一个线程来执行新来的任务，最大达到maximumPoolSizes

**无界队列**使用无界队列（例如，不具有预定义容量的LinkedBlockingQueue）将导致在所有 corePoolSize 线程都忙时新任务在队列中等待。这样，创建的线程就不会超过 corePoolSize。（因此，maximumPoolSize 的值也就无效了。）当每个任务完全独立于其他任务，即任务执行互不影响时，适合于使用无界队列；例如，在 Web 页服务器中。这种排队可用于处理瞬态突发请求，当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

**有界队列**当使用有限的 maximumPoolSizes 时，有界队列（如ArrayBlockingQueue）有助于防止资源耗尽，但是可能较难调整和控制。队列大小和最大池大小可能需要相互折衷：使用大型队列和小型池可以最大限度地降低 CPU 使用率、操作系统资源和上下文切换开销，但是可能导致人工降低吞吐量。如果任务频繁阻塞（例如，如果它们是 I/O 边界），则系统可能为超过您许可的更多线程安排时间。使用小型队列通常要求较大的池大小，CPU 使用率较高，但是可能遇到不可接受的调度开销，这样也会降低吞吐量。


handler：表示当拒绝处理任务时的策略，有以下四种取值：

```
ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。 
ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。 
ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务
```

#### 线程池复用原理
之前一直认为线程池的优点就是统一管理线程，但是不太明白线程池相对直接new Thread优点到底在哪？只是控制线程总数吗？
首先我们要理解，线程池最大的优点是能够复用线程，减少线程创建，销毁，恢复等状态切换的开销，提高程序的性能。那么从Java实现的角度怎么理解这段代码呢？

```
public class Main {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ExecutorService pool = Executors.newFixedThreadPool(5);
		pool.execute(new ThreadA());
	}

}

class ThreadA implements Runnable {

	@Override
	public void run() {
		// TODO Auto-generated method stub
		
	}
	
}
```

这里就有疑问了，每次线程池execute不都传入一个新的Thread吗？但是仔细观察发现，其实并没有调用start()方法。对于Thread来说，要执行run方法就需要调用start()开启线程，那么简单思考就能明白，线程池帮我们做了start()方法，其实就是控制了对线程的操作，线程池在初始化的时候就根据参数创建了n个线程于池子中，每当客户端调用execute时，实际上是复用现在池子中空闲的线程，调用new Thread（）这个传入对象的run方法来执行。
这里一定要知道一个概念，在java的世界里，世间万物皆对象，new Thread（）只是在jvm中创建一个Thread对象，并不是真正的系统内核线程，当start时才会在系统内存中创建线程去执行，因此也要说明一点，Thread对象和真正线程所用的内存空间是不同的。

以下是jdk的代码

```
 /** 
         * Main run loop 
         */ 
        public void run() { 
            try { 
                Runnable task = firstTask; 
                firstTask = null; 
                while (task != null || (task = getTask()) != null) { 
                    runTask(task);//这里最终会调用task.run() 
                    task = null; 
                } 
            } finally { 
                workerDone(this); 
            } 
        } 
    } 
```

总结一下：
线程重用的核心是，它把Thread.start()给屏蔽起来了（一定不要重复调用），然后它自己有一个Runnable.run()，循环在跑，跑的过程中不断检查我们是否有新加入的子Runnable对象，有就调一下我们的run()，其实就一个大run()把其它小run()#1,run()#2,...给串联起来了，基本原理就这么简单。 

#### execute & submit 区别
1.接收的参数不同
2.submit有返回值，而execute没有,通过Future来实现
3.submit方便Exception处理

#### 配置线程池的大小
参考：
如果是CPU密集型任务，就需要尽量压榨CPU，参考值可以设为 NCPU+1
如果是IO密集型任务，参考值可以设置为2*NCPU