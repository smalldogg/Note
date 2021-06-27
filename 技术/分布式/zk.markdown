## ZK特性

### 有序

这里如何去理解有序呢?

这里是单个leader，单个leader那么就比多个leader来控制写入更容易实现顺序性，leader中还有事务id来保证顺序性

### 集群从不可用状态恢复快

zk中有两种状态，可用和不可用。当集群处于不可用状态时，会迅速选出一个leader，具体选出leader的过程可以看看paxos，zk在这里简化了这个协议，使用了zab协议，并且这个恢复的过程很快。

原因①：zk中有多种角色，leader observer（不参与选举的过程） follwer（尽量少，可以试想一下，是3亿人选出领导的过程快，还是3个人选出领导的过程快）

原因②： 待补充

## 使用场景

### 分布式锁

​	Redis实现分布式锁很麻烦，需要使用多个线程多控制时间，zk中使用了seesion的概念来规避问题，zk会给每一个连接分配一个session，当sesssion不存在的时候，那么zk会自动把断开连接的那个client设置的节点删除掉，

这样就避免了死锁问题。同时在不同的节点，session是会共享的。

### 分布式配置中心

​	zk的初衷就不是将zk当作数据库来使用，具体可以看一下官网在不同数据和机器情况下写入和读取速度，所以不应该将zk当作数据库来使用

## 安装

安装笔记：

准备 node01~node04

1,安装jdk，并设置javahome

*, node01:

2，下载zookeeper  zookeeper.apache.org

3,tar xf zookeeper.*.tar.gz

4,mkdir /opt/mashibing

5, mv zookeeper /opt/mashibing

6,vi /etc/profile

​    export ZOOKEEPER_HOME=/opt/mashibing/zookeeper-3.4.6

​    export PATH=$PATH:$ZOOKEEPER_HOME/bin

7,cd zookeeper/conf

8,cp zoo.sem*.cfg  zoo.cfg

9,vi zoo.cfg

   dataDir=

   server.1=node01:2888:3888

10, mkdir -p /var/mashibing/zk

11,echo 1 > /var/mashibing/zk/myid

12,cd /opt && scp -r ./mashibing/ node02:`pwd`

13:node02~node04  创建 myid

14：启动顺序 1，2，3，4

15：zkServer.sh  start-foreground

## 特性

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201217170304502.png" alt="image-20201217170304502" style="zoom:67%;" />

## zk中的两种状态

### 可用

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201218160901440.png" alt="image-20201218160901440" style="zoom:67%;" />



### 不可用

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201218160942377.png" alt="image-20201218160942377" style="zoom:50%;" />



## zk节点



<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201218161307313.png" alt="image-20201218161307313" style="zoom:67%;" />



<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201218161346090.png" alt="image-20201218161346090" style="zoom:67%;" />

## zk具体特性

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201220114021187.png" alt="image-20201220114021187" style="zoom:67%;" />

这里，3888具体是用来选主通信，2888是用来节点之间的通信

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201220114125953.png" alt="image-20201220114125953" style="zoom:67%;" />

zk中是使用了最终一致性，如果说leader挂了的话，那么整个集群是出于不可用的状态的，等到leader被重新选举出来集群才能恢复可用



## zk中两阶段提交

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201220114309380.png" alt="image-20201220114309380" style="zoom:67%;" />

当客户端发起创建节点的命令时，从节点会将命令转发给leader，然后由leader发起创建节点

首先，leader会给所有的节点发送写日志的命令，当节点完成之后，并发送ok的命令返回给leader时，这时leader会发送write的命令，让节点去创建。这里leader为每个节点都维护了一个队列，这里也是使用了过半通过的方式来决定这个节点是否创建。因为有了队列，可能会出现有的人创建的的比较慢，这里你可用在从节点使用sysnc同步数据。

## zk选主的过程

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201220114916470.png" alt="image-20201220114916470" style="zoom:67%;" />

这里当leader挂了之后，当从节点发现之后会发起新一轮的选主的过程。使用谦让制来决定而不是使用争抢制来选出leader。只要你给别的节点发出投票的命令，那么接到的节点会发起原子广播。会给所有的人发出投票。

这里首先会比较Zxid，再比较Myid，，过半通过后选出leaders



## zk中的Watch

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201222172534576.png" alt="image-20201222172534576" style="zoom:67%;" />

只有读请求才会有watch，并且是一次性的

### data watches and child watches

getData() exists() 设置 的是data watches. 

getChildren() 设置的是 child watches

## 分布式锁

![image-20201223151056126](C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201223151056126.png)