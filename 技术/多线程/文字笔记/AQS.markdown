## 队列同步器

内部维护了一个volatile int state（代表共享资源）变量表示同步状态，通过内置的FIFO队列

完成资源获取线程的排队工作（多线程争用资源被阻塞时会进入此队列）。

主要通过getState()、setState(int newState)和compareAndSetState(int expect, int update)对state来进行线程安全操作。

### AQS定义两种资源共享方式

1）Exclusive（独占式，只有一个线程能执行，如ReentrantLock）。

2）Share（共享式，多个线程可同时执行，如Semaphore、CountDownLatch）。

# 二 队列同步器方法

### **1.2 Node源码说明**

**int waitStatus（等待状态）：**

1）waitStatus为int，默认值为0，所以初始状态值为0；

2）CANCELLED，值为1，由于在同步队列中等待的线程等待超时或者被中断，需要从同步队列中取消等待，

节点进入该状态将不会变化；

3）SIGNAL，值为-1，后继节点的线程处于等待状态，而前驱节点的线程如果释放了同步状态或者被取消，

将会通知后继节点，使后继节点的线程得以运行；

4）CONDITION，值为-2，节点在等待队列中，节点线程等待在Condition上，当其他线程对Condition调用signal()方法后，该节点将会从等待队列中转移到同步队列中，加入到对同步状态的获取中

5）PROPAGATE，值为-3，表示下次共享式同步状态获取将会无条件地被传播下去；