---
title: Java并发编程
toc: true
date: 2017-01-24 10:44:49
category: 
	- 技术贴
	- Java
	- 基础
tags: 
    - 线程
    - concurrent
---

# 什么是并发

## Java中的并发
Java 5 引入java.util.concurrent包，ExecutorService诞生，并引入CountDownLathc,CyclicBarrier、Semaphore、ConcurrentHashMap和BlockingQueue
Java 7 引入Fork/Join

### 什么是AQS
AQS的全称为（AbstractQueuedSynchronizer），这个类也是在java.util.concurrent.locks下面，是JDK1.5提供的一个基于FIFO等待队列实现的一个用于实现同步器的基础框架，各种同步工具类内部都有一个如下定义的内部结构：
```
abstract static class Sync extends AbstractQueuedSynchronizer
```
<!--more-->

#### ArrayBlockingQueue
一个由数组支持的有界阻塞队列
一旦创建，无法扩容容量
试图向已满队列中放入元素会导致操作受阻塞，从空队列提取元素也会阻塞
支持生产者和消费者线程进行排序的可选公平策略，默认非公平，公平会降低吞吐量，但减少可变性和“不平衡性”
只使用一个lock来控制互斥访问


#### LinkedBlockingQueue
一个基于链表的，范围任意的blocking queue.
吞吐量一般高于基于数组的队列，但并发场景下性能差
可选的容量范围防止队列无限扩展，默认Interger.MAX_VALUE
如果生产者速度但与消费者速度，也许还没有等到队列满阻塞，系统内存就有可能已被消耗殆尽。
使用了2个lock，一个takelock和一个putlock，读写分离，并发效率更高

#### ConcurrentLinkedQueue
ArrayBlockingQueue和LinkedBlockingQueue都是使用lock来实现，都属于阻塞式队列，而ConcurrentLinkedQueue使用循环CAS来实现，是非阻塞式的"lock-free"

基于链表的无界队列，使用循环CAS来实现非阻塞

#### HashMap
HashMap线程不安全,多线程场景下进行put操作会引起死循环，导致CPU利用率接近100%

##### HashMap多线程场景下存在的问题
1.多线程put操作后，get操作导致死循环。
2.多线程put非null元素后，get操作得到null值。
3.多线程put操作，导致元素丢失。

**死循环原理**
1.HashMap的put操作rehash会导致死循环
2.如果扩容前相邻的两个Entry在扩容后还是分配到相同的table位置上，就会出现死循环的BUG

#### ConcurrentHashMap
HashMap线程不安全
HashTable使用synchronized来保证线程安全，但线程竞争激烈的情况下效率非常低下
ConcurrentHashMap采用Segment分段技术，将数据分为若干段，形成多段锁，多线程访问时就大大减少竞争，提高并发访问

ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成，Segment是一种可重入锁ReentrantLock，一个ConcurrentHashMap里包含一个Segment数组，Segment的结构和HashMap类似，是一种数组和链表结构， 一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素， 每个Segment守护者一个HashEntry数组里的元素,当对HashEntry数组的数据进行修改时，必须首先获得它对应的Segment锁。

通过按位与的哈希算法来定位segments数组的索引，必须保证segments数组的长度是2的N次方，注意concurrencyLevel的最大大小是65535，意味着segments数组的长度最大为65536，对应的二进制是16位。

##### ConcurrentHashMap的size操作
ConcurrentHashMap的做法是先尝试2次通过不锁住Segment的方式来统计各个Segment大小，如果统计的过程中，容器的count发生了变化，则再采用加锁的方式来统计所有Segment的大小。
那么ConcurrentHashMap是如何判断在统计的时候容器是否发生了变化呢？使用modCount变量，在put , remove和clean方法里操作元素前都会将变量modCount进行加1，那么在统计size前后比较modCount是否发生变化，从而得知容器的大小是否发生变化。

### CountDownLatch

javadoc
CountDownLatch: A synchronization aid that allows one or more threads to wait until a set of operations being performed in other threads completes.

//调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行  
public void await() throws InterruptedException { };   

//和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行  
public boolean await(long timeout, TimeUnit unit) throws InterruptedException { };  

//将count值减1 ，这个count值就是CountDownLatch初始化的时候传入的值
public void countDown() { };  

```
/**
 * @developer liyixiang
 * @date 2017年2月27日
 * @since JDK 1.8
 * @function
 */
public class CountRunnable implements Runnable{

	private CountDownLatch begin,end;
	private List<Integer> subList;
	
	public CountRunnable(List<Integer> subList, CountDownLatch begin,CountDownLatch end) {
		// TODO Auto-generated constructor stub
		this.subList = subList;  
        this.begin = begin;  
        this.end = end; 
	}

	@Override
	public void run() {
		// TODO Auto-generated method stub
		try {
			begin.await();
			if (subList != null) {  
                for (int i : subList) {  
                    System.out.println("线程" + Thread.currentThread().getName() + ", i = " + i);  
                }  
            } 
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally{  
            System.out.println(System.currentTimeMillis() + ",线程" + Thread.currentThread().getName() + ",开始执行！");  
            //end线程组准备
            end.countDown();  
        }
	}
}


/**
 * @developer liyixiang
 * @date 2017年2月27日
 * @since JDK 1.8
 * @function
 */
public class BatchWithCountDownLatch {

	private static final int MAX = 3;
	
	public static void list(List<Integer> list){
		if(null == list){
			list = new ArrayList<>();
		}
		
		for(int i=0;i<100;i++){
			list.add(i);
		}
	}
	
	public static void main(String[] args) {
		List<Integer> list = new ArrayList<>();
		list(list);
		
		//把list拆分
		int mod = list.size()%MAX;
		int threadCount = mod == 0 ? list.size()/MAX : list.size()/MAX+1;
		
		ExecutorService executors = Executors.newFixedThreadPool(threadCount);
		CountDownLatch begin = new CountDownLatch(1);
		CountDownLatch end = new CountDownLatch(threadCount);
		
		for(int i=0;i<threadCount;i++){
			int subsize = (i+1)*MAX;
			//这里先执行线程池run
            executors.execute(new CountRunnable(list.subList(i * MAX, subsize > list.size() ? list.size() : subsize),begin,end));  
		}
		
		System.out.println("开始 !");
		//闭锁放开，千军万马狂奔 begin的CountDownLatch await方法阻塞消除
        begin.countDown();  
        long startTime = System.currentTimeMillis();  
          
        try {  
        	//end组的线程都阻塞
            end.await();  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        } finally {  
            System.out.println("线程" + Thread.currentThread().getName() + "," + System.currentTimeMillis() + ", 所有线程已完成，开始进入下一步！");  
            System.out.println("花费时间 -> " + (System.currentTimeMillis() - startTime) + " ms");  
        }  
        System.out.println("开始进入第二步操作! ");        
        System.out.println("end! ");  
	}
}
```

其实CountDownLatch的用法很好理解，首先await方法是线程的闭锁，一执行到这里就会判断count的大小，跟进源码

```
public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
    
public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }
    
    
protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;
        }

```

会发现这里最终会根据getState()==0这个结果来判断线程是否中断，那么这个状态是怎么来的呢，看看CountDownLatch初始化时的源码吧


```
public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }
    
    
private static final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 4982264981922014374L;

        Sync(int count) {
            setState(count);
        }
        
        ….

```

果然，在CountDownLatch初始化的时候，就设置了state属性，这个state属性是Node内部类的一个volatile变量，可以从源码得知。说完了await方法，那么countdown方法就很好解释了，应该就是操纵这个闭锁的钥匙，先开车，看看源码怎么写的

```
public void countDown() {
        sync.releaseShared(1);
    }
    
protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }    

```

看到了吧，int nextc = c -1;说明countdown方法是某线程将state数减1的过程。好了，通过几段源码就对CountDownLatch熟悉了。总结一下
（1）CountDownLatch是用来同步一个或者多个执行多个任务的线程。它只能使用一次。
（2）CountDownLatch是做减计数，当count值为0时，线程突破await，开始向下执行。
（3）CountDownLatch强调的是一个线程等其他多个线程完成某件事情。


### CyclicBarrier
javadoc
CyclicBarrier : A synchronization aid that allows a set of threads to all wait for each other to reach a common barrier point.

先看一个demo

```
/**
 * @developer liyixiang
 * @date 2017年2月27日
 * @since JDK 1.8
 * @function
 */
public class CyclicBarrierTest {

    public static void main(String[] args) throws InterruptedException {  
        CyclicBarrier cyclicBarrier = new CyclicBarrier(5, new Runnable() {  
            @Override  
            public void run() {  
                System.out.println("线程组执行结束");  
            }  
        });  
        for (int i = 0; i < 5; i++) {  
            new Thread(new readNum(i,cyclicBarrier)).start();  
        }  
        //CyclicBarrier 可以重复利用，  
        // 这个是CountDownLatch做不到的  
//        for (int i = 11; i < 16; i++) {  
//            new Thread(new readNum(i,cyclicBarrier)).start();  
//        }  
    }  
    static class readNum  implements Runnable{  
        private int id;  
        private CyclicBarrier cyc;  
        public readNum(int id,CyclicBarrier cyc){  
            this.id = id;  
            this.cyc = cyc;  
        }  
        @Override  
        public void run() {  
            synchronized (this){  
                System.out.println("id:"+id);  
                try {  
                    cyc.await();  
                    System.out.println("线程组任务" + id + "结束，其他任务继续");  
                } catch (Exception e) {  
                    e.printStackTrace();  
                }  
            }  
        }  
    }  
}
```


结果如下
```
id:2
id:0
id:4
id:1
id:3
线程组执行结束
线程组任务1结束，其他任务继续
线程组任务4结束，其他任务继续
线程组任务3结束，其他任务继续
线程组任务0结束，其他任务继续
线程组任务2结束，其他任务继续
```

从结果看，关键点就在CyclicBarrier的await方法，还是这个节奏，看源码

```
public int await() throws InterruptedException, BrokenBarrierException {
        try {
            return dowait(false, 0L);
        } catch (TimeoutException toe) {
            throw new Error(toe); // cannot happen
        }
    }
    
进dowait方法看，方法较长，截取重要部分
 第一部分
 int index = --count;
            if (index == 0) {  // tripped
                boolean ranAction = false;
                try {
                    final Runnable command = barrierCommand;
                    if (command != null)
                        command.run();
                    ranAction = true;
                    nextGeneration();
                    return 0;
                } finally {
                    if (!ranAction)
                        breakBarrier();
                }
            }
            
            
第二部分
for (;;) {
                try {
                    if (!timed)
                        trip.await();
                    else if (nanos > 0L)
                        nanos = trip.awaitNanos(nanos);
                } catch (InterruptedException ie) {
                    if (g == generation && ! g.broken) {
                        breakBarrier();
                        throw ie;
                    } else {
                        // We're about to finish waiting even if we had not
                        // been interrupted, so this interrupt is deemed to
                        // "belong" to subsequent execution.
                        Thread.currentThread().interrupt();
                    }
                }

```

从源码可以看出，执行顺序就是先执行传入的实现Runnable接口的线程类run方法，再检测await的状态，通过count控制是否执行run，继续跟代码能看到，count初始值为parties，也就是构造CyclicBarrier时传入的线程数。
那么问题来了。。网上在说CountDownLatch和CyclicBarrier的异同点时，都说CountDownLatch是减计数器，CyclicBarrier是加计数器，但是貌似源代码也是- -count操作，不过咱们要这么理解，CyclicBarrier让一个线程达到屏障时被阻塞，直到最后一个线程达到屏障时，屏障才会开门，所有被屏障拦截的线程才会继续执行，这是一种积累的过程。而CountDownLatch

。不过对于CyclicBarrier来说，也总结一下
 （1）CyclicBarrier初始化时规定一个数目，然后计算调用了CyclicBarrier.await()进入等待的线程数。当线程数达到了这个数目时，所有进入等待状态的线程被唤醒并继续。 
 （2）CyclicBarrier就象它名字的意思一样，可看成是个障碍， 所有的线程必须到齐后才能一起通过这个障碍。 
 （3）CyclicBarrier初始时还可带一个Runnable的参数， 此Runnable任务在CyclicBarrier的数目达到后，所有其它线程被唤醒前被执行。
 （4）最重要一点，CyclicBarrier是可以重复使用的。


### 什么是Fork/Join
（1）多核
（2）分治&递归

1.把大任务拆分为多个小的Fork，通过这些Fock去处理任务。
2.处理任务完毕后Join起来，合并结果。


### CAS
Compare and swap,每一个CAS操作过程都包含三个运算符：一个内存地址V，一个旧的期望的值A和一个新值B，操作的时候如果这个地址上存放的值等于这个旧的期望的值A，则将地址上的值赋为新值B，否则不做任何操作。CAS的基本思路就是，如果这个地址上的值和期望的值相等，则给其赋予新值，否则不做任何事儿，但是要返回原值是多少。大致原理用代码表示为

```
class SimulatedCAS {  
    private int value;  
  
    public synchronized int getValue() {  
        return value;  
    }  
    public synchronized int compareAndSwap(int expectedValue, int newValue) {  
        int oldValue = value;  
        if (value == expectedValue)  
            value = newValue;  
        return oldValue;  
    }  
}  
```

总结一个流程，方便更清晰的理解CAS：
1、AtomicInteger里面的value原始值为3，即主内存中AtomicInteger的value为3，根据Java内存模型，线程1和线程2各自持有一份value的副本，值为3
2、线程1运行到第三行获取到当前的value为3，线程切换
3、线程2开始运行，获取到value为3，利用CAS对比内存中的值也为3，比较成功，修改内存，此时内存中的value改变比方说是4，线程切换
4、线程1恢复运行，利用CAS比较发现自己的value为3，内存中的value为4，得到一个重要的结论–>此时value正在被另外一个线程修改，所以我不能去修改它
5、线程1的compareAndSet失败，循环判断，因为value是volatile修饰的，所以它具备可见性的特性，线程2对于value的改变能被线程1看到，只要线程1发现当前获取的value是4，内存中的value也是4，说明线程2对于value的修改已经完毕并且线程1可以尝试去修改它
6、最后说一点，比如说此时线程3也准备修改value了，没关系，因为比较-交换是一个原子操作不可被打断，线程3修改了value，线程1进行compareAndSet的时候必然返回的false，这样线程1会继续循环去获取最新的value并进行compareAndSet，直至获取的value和内存中的value一致为止


#### CAS的不完美
ABA问题，从上述例子解释也就是说本身内存是3，线程1修改成4而后又修改成3，但是对于线程2来说内存值和预期值是保持一致的。那么java.util.concurrent包为了解决这个问题，提供了一个带有标记的原子引用类”AtomicStampedReference”，它可以通过控制变量值的版本来保证CAS的正确性，不过比较鸡肋，毕竟这种场景并不影响并发修改，不过如果真有这种场景，还是使用互斥信号量解决更加高效。

#### 如何保证线程安全
其实我们在语言层面是没有做任何同步的操作的，大家也可以看到Atomic源码没有任何锁加在上面，可它为什么是线程安全的呢？这就是Atomic包下这些类的奥秘：语言层面不做处理，我们将其交给硬件—CPU和内存，利用CPU的多处理能力，实现硬件层面的阻塞，再加上volatile变量的特性即可实现基于原子操作的线程安全。所以说，CAS并不是无阻塞，只是阻塞并非在语言、线程方面，而是在硬件层面，所以无疑这样的操作会更快更高效！


