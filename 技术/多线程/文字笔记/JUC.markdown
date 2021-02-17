## JUC

### volatile

1. 保证线程之间的可见性
2. 禁止指令重拍

### 为什么需要volatile

1. 每个线程内部都保留着共享变量的副本，那么线程之间对于共享变量的操作都是基于自己的工作内存

### jmm

JMM描述的是一组规则，通过这组规则控制程序中各个变量在共享数据区域和私有数据区域的访问方式

jmm中分为主内存和工作内存

## 原子性&可见性&有序性

### 原子性

原子性可以通过synchronized和Lock来保证，任一时刻只能有一个线程访问该代码块

### 可见性

通过volatile保证可见性，是通过内存屏障来实现的



## synchronized

对象锁，作用粒度是对象，是可重入的

1. 同步实例方法，锁是当前实例对象 

2. 同步类方法，锁是当前类对象
3. 同步代码块，锁是括号里面的对象

原理

通过内部对象Monitor实现，依赖操作系统的Mutex lock(互斥锁)实现

synchronized关键字被编译成字节码后会被翻译成monitorenter 和 monitorexit 两条指令分别在同步块逻辑代码的起始位置与结束位置

## 对象的内存布局

### 对象头

比如 hash码，对象所属的年代，对象锁，锁状态标志，偏向 锁（线程）ID，偏向时间，数组长度（数组对象）等

### 实例数据

即创建对象时，对象中成员变量，方法等

### 对齐填充 

对象的大小必须是8字节的整数倍

## 锁升级的过程

锁的状态总共有四种，无锁状态、偏向锁、轻量级锁和重量级锁，锁的升级只能从高到低，不能反过来

![image-20210104103858004](C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20210104103858004.png)

## AQS

aqs中定义了一个state变量和一个fifo队列

### volatile state

### state的三种获取方式

getState()、setState()、compareAndSetState()

### AQS定义两种资源共享方式 

1. Exclusive-独占，只有一个线程能执行，如ReentrantLock 
2. Share-共享，多个线程可以同时执行，如Semaphore/CountDownLatch



## cas

轻量级锁，通过自旋来抢锁，为了避免内核的切换





## ReentrantLock

### state重入次数

### 非公平锁的加锁过程

```java
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

如果cas操作成功的话，那么直接获得锁，这里体现了非公平，直接会抢锁，插队

cas只能有一个线程成功，其他的会去调用acquire方法

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

```java
tryAcquire(arg)
final boolean nonfairTryAcquire(int acquires) {
    //获取当前线程
    final Thread current = Thread.currentThread();
    //获取state变量值
    int c = getState();
    if (c == 0) { //没有线程占用锁
        if (compareAndSetState(0, acquires)) {
            //占用锁成功,设置独占线程为当前线程
            setExclusiveOwnerThread(current);
            return true;
        }
    } else if (current == getExclusiveOwnerThread()) { //当前线程已经占用该锁
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        // 更新state值为新的重入次数
        setState(nextc);
        return true;
    }
    //获取锁失败
    return false;
}
```

如果这里也是失败的话，那么会入队

```java
/**
 * 将新节点和当前线程关联并且入队列
 * @param mode 独占/共享
 * @return 新节点
 */
private Node addWaiter(Node mode) {
    //初始化节点,设置关联线程和模式(独占 or 共享)
    Node node = new Node(Thread.currentThread(), mode);
    // 获取尾节点引用
    Node pred = tail;
    // 尾节点不为空,说明队列已经初始化过
    if (pred != null) {
        node.prev = pred;
        // 设置新节点为尾节点
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    // 尾节点为空,说明队列还未初始化,需要初始化head节点并入队新节点
    enq(node);
    return node;
}
```



```java
/**
 * 初始化队列并且入队新节点
 */
private Node enq(final Node node) {
    //开始自旋
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            // 如果tail为空,则新建一个head节点,并且tail指向head
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            // tail不为空,将新节点入队
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

这里我们假设有3个线程（ABC）

当线程A获取到了锁，那么BC就会调用acquire方法，如果线程A拿着锁不放，那么BC都会去排队

我们来看一下enq()

 这里体现了经典的自旋+CAS组合来实现非阻塞的原子操作。由于compareAndSetHead的实现使用了unsafe类提供的CAS操作，所以只有一个线程会创建head节点成功。假设线程B成功，之后B、C开始第二轮循环，此时tail已经不为空，两个线程都走到else里面。假设B线程compareAndSetTail成功，那么B就可以返回了，C由于入队失败还需要第三轮循环。最终所有线程都可以成功入队。 

```java
/**
 * 已经入队的线程尝试获取锁
 */
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true; //标记是否成功获取锁
    try {
        boolean interrupted = false; //标记线程是否被中断过
        for (;;) {
            final Node p = node.predecessor(); //获取前驱节点
            //如果前驱是head,即该结点已成老二，那么便有资格去尝试获取锁
            if (p == head && tryAcquire(arg)) {
                setHead(node); // 获取成功,将当前节点设置为head节点
                p.next = null; // 原head节点出队,在某个时间点被GC回收
                failed = false; //获取成功
                return interrupted; //返回是否被中断过
            }
            // 判断获取失败后是否可以挂起,若可以则挂起
            if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                // 线程若被中断,设置interrupted为true
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

 code里的注释已经很清晰的说明了acquireQueued的执行流程。假设B和C在竞争锁的过程中A一直持有锁，那么它们的tryAcquire操作都会失败 

```java
/**
 * 判断当前线程获取锁失败之后是否需要挂起.
 */
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    //前驱节点的状态
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL)
        // 前驱节点状态为signal,返回true
        return true;
    // 前驱节点状态为CANCELLED
    if (ws > 0) {
        // 从队尾向前寻找第一个状态不为CANCELLED的节点
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        // 将前驱节点的状态设置为SIGNAL
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
  
/**
 * 挂起当前线程,返回线程中断状态并重置
 */
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

 线程入队后能够挂起的前提是，它的前驱节点的状态为SIGNAL，它的含义是“Hi，前面的兄弟，如果你获取锁并且出队后，记得把我唤醒！”。所以shouldParkAfterFailedAcquire会先判断当前节点的前驱是否状态符合要求，若符合则返回true，然后调用parkAndCheckInterrupt，将自己挂起。如果不符合，再看前驱节点是否>0(CANCELLED)，若是那么向前遍历直到找到第一个符合要求的前驱，若不是则将前驱节点的状态设置为SIGNAL。 

## 释放锁的过程

```java
public void unlock() {
    sync.release(1);
}
  
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

 如果理解了加锁的过程，那么解锁看起来就容易多了。流程大致为先尝试释放锁，若释放成功，那么查看头结点的状态是否为SIGNAL，如果是则唤醒头结点的下个节点关联的线程，如果释放失败那么返回false表示解锁失败 

```java
/**
 * 释放当前线程占用的锁
 * @param releases
 * @return 是否释放成功
 */
protected final boolean tryRelease(int releases) {
    // 计算释放后state值
    int c = getState() - releases;
    // 如果不是当前线程占用锁,那么抛出异常
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        // 锁被重入次数为0,表示释放成功
        free = true;
        // 清空独占线程
        setExclusiveOwnerThread(null);
    }
    // 更新state值
    setState(c);
    return free;
}
```

 这里入参为1。tryRelease的过程为：当前释放锁的线程若不持有锁，则抛出异常。若持有锁，计算释放后的state值是否为0，若为0表示锁已经被成功释放，并且则清空独占线程，最后更新state值，返回free。 



![image-20210104103802563](C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20210104103802563.png)







参考博客

1. https://blog.csdn.net/fuyuwei2015/article/details/83719444
2. https://blog.csdn.net/fuyuwei2015







## 四种引用类型

### 强引用

​	不会被gc回收

### SoftReference

软引用，当堆内存不够时，会把软引用干掉

### WeakReference

只要发生弱引用，那么就会被回收



## ThreadLocal

```java
 static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
}
```

这里key是软引用，如果当前线程一直在跑，当发生gc的时候，key会被回收，但是value还在，那么会发生内存泄漏

### 关系

ThreadlocalMap 是 ThreadLocal中的静态内部类 ，Thread中有ThreadLocalMap的属性





## Queue(多线程集合)

CocurrentLinkedQueue   // concurrentArrayQueue



take 和 put 方法是会阻塞的，同时当put满了之后，程序会阻塞

add 方法当超过了指定的容量，那么会报错

offer方法会放满，不会报错

BlockingQueue 

1. LinkdeBQ  无界队列
2. ArrayBQ    有界队列

 	3. TransferQueue   多个人传递，并且线程会等到别人取走之后，线程才会去干别的。
 	4. SynchonusQueue  一个人传递

### DelayQueue

blockingqueue的一种

```java
static BlockingQueue<MyTask> tasks = new DelayQueue<>();

	static Random r = new Random();

	static class MyTask implements Delayed {
		String name;
		long runningTime;

		MyTask(String name, long rt) {
			this.name = name;
			this.runningTime = rt;
		}

		@Override
		public int compareTo(Delayed o) {
			if(this.getDelay(TimeUnit.MILLISECONDS) < o.getDelay(TimeUnit.MILLISECONDS))
				return -1;
			else if(this.getDelay(TimeUnit.MILLISECONDS) > o.getDelay(TimeUnit.MILLISECONDS))
				return 1;
			else
				return 0;
		}

		@Override
		public long getDelay(TimeUnit unit) {

			return unit.convert(runningTime - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
		}


		@Override
		public String toString() {
			return name + " " + runningTime;
		}
	}

```

### synchronusQueue

容量为0

## map

1. ConcurrentHashMap

2. ConcurrentSkipListMap 高并发并且可以排序

## 线程池

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20210105174603703.png" alt="image-20210105174603703" style="zoom:67%;" />

### 线程池参数

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20210105175316632.png" alt="image-20210105175316632" style="zoom:67%;" />

#### corePoolSize  

线程池中的核心线程数，当提交一个任务时，线程池创建一个新线程执行任务，直到当前线程数等corePoolSize；如果当前线程数为corePoolSize，继续提交的任务被保存到阻塞队列中，等待被执行；如果执行了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有核心线程  

#### maximumPoolSize
线程池中允许的最大线程数。如果当前阻塞队列满了，且继续提交任务，则创建新的线程执行任务，前提是当前线程数小于maximumPoolSize；  

#### keepAliveTime
线程池维护线程所允许的空闲时间。当线程池中的线程数量大于corePoolSize的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了keepAliveTime  

#### workQueue
用来保存等待被执行的任务的阻塞队列，且任务必须实现Runable接口，在JDK中提供了如下阻塞队列：  

1. ArrayBlockingQueue：基于数组结构的有界阻塞队列，按FIFO排序任务  
2. LinkedBlockingQuene：基于链表结构的阻塞队列，按FIFO排序任务，吞吐量通常要高于ArrayBlockingQuene  
3. SynchronousQuene：一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQuene  
4. priorityBlockingQuene：具有优先级的无界阻塞队列  

#### threadFactory
它是ThreadFactory类型的变量，用来创建新线程。默认使用Executors.defaultThreadFactory() 来创建线程。使用默认的ThreadFactory来创建线程时，会使新创建的线程具有相同的NORM_PRIORITY优先级并且是非守护线程，同时也设置了线程的名称  

#### handler
线程池的饱和策略，当阻塞队列满了，且没有空闲的工作线程，如果继续提交任务，必
须采取一种策略处理该任务，线程池提供了4种策略：  

1. AbortPolicy：直接抛出异常，默认策略  
2. CallerRunsPolicy：用调用者所在的线程来执行任务  
3. DiscardOldestPolicy：丢弃阻塞队列中靠最前的任务，并执行当前任务  
4. DiscardPolicy：直接丢弃任务  

当然也可以根据应用场景实现RejectedExecutionHandler接口，自定义饱和策略，如记录日志或持久化存储不能处理的任务  



### 四种线程池

1. newFixedThreadPool，创建定长的核心线程数量，队列为LinkedBlockingQueue

2. newCachedThreadPool  最大线程数为Integer.MAX_VALUE，队列为Sync队列

3. newSingleThreadExecutor 单一线程

4. newScheduledThreadPool 定时任务

   ```java
   ScheduledExecutorService service = Executors.newScheduledThreadPool(4);
   service.scheduleAtFixedRate(()->{
       try {
           TimeUnit.MILLISECONDS.sleep(new Random().nextInt(1000));
       } catch (InterruptedException e) {
           e.printStackTrace();
       }
       System.out.println(Thread.currentThread().getName());
   }, 0, 500, TimeUnit.MILLISECONDS);
   ```

   

### 线程池的5种状态

1. RUNNING  	

   状态说明：线程池处在RUNNING状态时，能够接收新任务，以及对已添加的任务进行处理  

   状态切换： 线程池的初始化状态是RUNNING。换句话说，线程池被一旦被创建，就处于RUNNING状态，并且线程池中的任务数为0  

2. SHUTDOWN  

   状态说明：线程池处在SHUTDOWN状态时，不接收新任务，但能处理已添加的任务  

   状态切换：调用线程池的shutdown()接口时，线程池由RUNNING -> SHUTDOWN 

3. STOP  

   状态说明：线程池处在STOP状态时，不接收新任务，不处理已添加的任务，并且会中断正在处理的任务

   状态切换：调用线程池的shutdownNow()接口时，线程池由(RUNNING orSHUTDOWN ) -> STOP    

4. TIDYING  

   状态说明：当所有的任务已终止，ctl记录的”任务数量”为0，线程池会变为TIDYING状态。当线程池变为TIDYING状态时，会执行钩子函数terminated()。terminated()在ThreadPoolExecutor类中是空的，若用户想在线程池变为TIDYING时，进行相应的处理；可以通过重载terminated()函数来实现  

   状态切换：当线程池在SHUTDOWN状态下，阻塞队列为空并且线程池中执行的任务也为空时，就会由 SHUTDOWN -> TIDYING。 当线程池在STOP状态下，线程池中执行的任务为空时，就会由STOP -> TIDYING  

5. TERMINATED  

   状态说明：线程池彻底终止，就变成TERMINATED状态  

   状态切换：线程池处在TIDYING状态时，执行完terminated()之后，就会由 TIDYING -
   \> TERMINATED  

   进入TERMINATED的条件如下：  

   1. 线程池不是RUNNING状态； 线程池状态不是TIDYING状态或TERMINATED状态；

   2. 如果线程池状态是SHUTDOWN并且workerQueue为空;
   3. 如果线程池状态是SHUTDOWN并且workerQueue为空  
   4. workerCount为0；  
   5. 设置TIDYING状态成功  ss