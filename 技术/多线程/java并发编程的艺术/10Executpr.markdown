## 关闭线程池

可以通过调用线程池的shutdown或shutdownNow方法来关闭线程池。它们的原理是遍历线程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程，所以无法响应中断的任务可能永远无法终止。但是它们存在一定的区别，shutdownNow首先将线程池的状态设置成STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表，而shutdown只是将线程池的状态设置成SHUTDOWN状态，然后中断所有没有正在执行任务的线
程。  



## 线程池的监控

1. taskCount：线程池需要执行的任务数量。
2. completedTaskCount：线程池在运行过程中已完成的任务数量，小于或等于taskCount。
3. largestPoolSize：线程池里曾经创建过的最大线程数量。通过这个数据可以知道线程池是否曾经满过。如该数值等于线程池的最大大小，则表示线程池曾经满过。
4. getPoolSize：线程池的线程数量。如果线程池不销毁的话，线程池里的线程不会自动销毁，所以这个大小只增不减。
5. getActiveCount：获取活动的线程数。  



## FixedThreadPool

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
return new ThreadPoolExecutor(nThreads, nThreads,
0L, TimeUnit.MILLISECONDS,
new LinkedBlockingQueue<Runnable>());
}
```

FixedThreadPool的corePoolSize和maximumPoolSize都被设置为创建FixedThreadPool时指定的参数nThreads。当线程池中的线程数大于corePoolSize时，keepAliveTime为多余的空闲线程等待新任务的最长时间，超过这个时间后多余的线程将被终止。这里把keepAliveTime设置为0L，意味着多余的空闲线程会被立即终止。 

**问题**

1）当线程池中的线程数达到corePoolSize后，新任务将在无界队列中等待，因此线程池中
的线程数不会超过corePoolSize。
2）由于1，使用无界队列时maximumPoolSize将是一个无效参数。
3）由于1和2，使用无界队列时keepAliveTime将是一个无效参数。
4）由于使用无界队列，运行中的FixedThreadPool（未执行方法shutdown()或
shutdownNow()）不会拒绝任务（不会调用RejectedExecutionHandler.rejectedExecution方法）。  



## SingleThreadExecutor  

```java
public static ExecutorService newSingleThreadExecutor() {
return new FinalizableDelegatedExecutorService
(new ThreadPoolExecutor(1, 1,
0L, TimeUnit.MILLISECONDS,
new LinkedBlockingQueue<Runnable>()));
}
```

##  CachedThreadPool  

```java
public static ExecutorService newCachedThreadPool() {
return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
60L, TimeUnit.SECONDS,
new SynchronousQueue<Runnable>());
}
```

CachedThreadPool的corePoolSize被设置为0，即corePool为空；maximumPoolSize被设置为
Integer.MAX_VALUE，即maximumPool是无界的。这里把keepAliveTime设置为60L，意味着
CachedThreadPool中的空闲线程等待新任务的最长时间为60秒，空闲线程超过60秒后将会被
终止。  