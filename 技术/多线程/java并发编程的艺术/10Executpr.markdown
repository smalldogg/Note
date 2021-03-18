## FixedThreadPool详解  

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
return new ThreadPoolExecutor(nThreads, nThreads,
0L, TimeUnit.MILLISECONDS,
new LinkedBlockingQueue<Runnable>());
}
```

FixedThreadPool的corePoolSize和maximumPoolSize都被设置为创建FixedThreadPool时指
定的参数nThreads。
当线程池中的线程数大于corePoolSize时，keepAliveTime为多余的空闲线程等待新任务的
最长时间，超过这个时间后多余的线程将被终止。这里把keepAliveTime设置为0L，意味着多余
的空闲线程会被立即终止。 