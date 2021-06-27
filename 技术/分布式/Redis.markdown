## Redis

为什么需要使用redis,关系型数据库是可以解决问题的，那么为什么还需要使用redis呢？首先数据库通过创建索引是可以实现高效查询的，但是如果数据量上来的话，那么并发查询情况下，磁盘带宽会很大程度上影响速度

## 内存和磁盘的寻址速度对比

磁盘：

1，寻址：ms

2，带宽：G/M

内存：

1，寻址：ns

2，带宽：很大

秒>毫秒>微秒>纳秒 磁盘比内存在寻址上慢了10W倍

I/O buffer：成本问题

磁盘与磁道，扇区，一扇区 512Byte带来一个成本变大：索引

4K 操作系统，无论你读多少，都是最少4k从磁盘拿

所以我们使用一个折中的方案，将数据缓存在内存中

## io模型



## redis安装

1，yum install wget

2,cd ~

3,mkdir soft

4,cd soft

5,wget  http://download.redis.io/releases/redis-5.0.5.tar.gz

6,tar xf  redis...tar.gz

7,cd redis-src

8,看README.md

9, make 

 ....yum install gcc 

.... make distclean

10,make

11,cd src  ....生成了可执行程序

12, cd ..

13,make install PREFIX=/opt/mashibing/redis5

14,vi /etc/profile

...  export REDIS_HOME=/opt/mashibing/redis5

...  export PATH=$PATH:$REDIS_HOME/bin

..source /etc/profile

15,cd utils

16,./install_server.sh （可以执行一次或多次）

  a) 一个物理机中可以有多个redis实例（进程），通过port区分

  b) 可执行程序就一份在目录，但是内存中未来的多个实例需要各自的配置文件，持久化目录等资源

  c) service  redis_6379 start/stop/stauts   >  linux  /etc/init.d/**** 

  d)脚本还会帮你启动！

17,ps -fe | grep redis 



## 二进制安全

redis作为中间件采用的字节流，客户端在连接的时候，需要选择编码的方式，代码中需要协定好编码格式，这样才能保证不乱码。

## 正负向索引

redis作为中间件采用的字节流，客户端在连接的时候，需要选择编码的方式，代码中需要协定好编码格式，这样才能保证不乱码。

**String**

**字符串**

set k v

get k v 

append k

getrange k start end 

setrange k start end

strlen k

**数值**

​	incr k

decr k 

**位图**

​	redis的字符串也可以用作位图，

setbit k offset value

getbit k offset

bitcount 返回k中1的个数

bitpos 返回指定位置上信息

bitop 对两个key做与或非运算



## list

### 栈

​	我们知道栈的特点是先进后出,这里我们使用同向命令的时候就可以将 list 当作栈来使用。lpush 和 lpop同时使用的时候，这样list就具有了栈的特点

​	lpush lpop 

​	rpush rpop

### 队列

​	我们知道队列的特点是先进后出,当我们使用反向命令的时候就可以将 list 当作队列来使用

​	lpush rpop 

### 数组

​	数组的特点是支持下标随机访问，在 list中，我们使用 lindex key index 也可以支持这样的操作，这样 list又有了 数组的特点 

​	lindex key index

​	lset key index element

### 修剪list

​	ltrim key start end	

​	lrem key count element

### 阻塞队列

​	brpop list timeout 当为0时，一直阻塞

​	rpop list timeout



## hash

首先我们看一下 hash 的结构   key field value 

我们可以对 field 进行计算，那么我们可以在 点赞  收藏 详情页 这些场景下使用

### 增

​	hset key fieldvalue

### 删

​	hdel key field

### 改

​	hincrby key field increment	

### 查

​	hget key field

​	hgetall key 

​	hkeys  key

​	hvals key

​	hstrlen key field

## set

set 集合，特点元素不重复，集合之间的操作很多。。。

集合之间的操作都是以第一个集合为标准的

### 交集

​	sinter key1 key2

### 差集	

​	sdiff key1 key2  

### 并集

​	sunion key1 key2

这里还有一组命令是带strore的，是将计算出来的结果存到一个key中

### 添加

sadd key value 

scard key 返回集合的大小

sismember key member判断给定的值是否在集合中



SRANDMEMBER key count

正数：取出一个去重的结果集（不能超过已有集）

负数：取出一个带重复的结果集，一定满足你要的数量

如果：0，不返回	

spop key  弹出一个

 

## sorted_set

这里sorted_set的命令是以z开头的， 因为 set 已经使用了 s , redis作者用了 z 作为 sorted_set 作为的命令开头，简单来说，就是字母表的最后一个字符这样记。看一下这个结构的特点，首先它是会帮你排序的，那么是依靠什么来排序的，这里你需要给它的每一个元素指定一个分数，这样它就能通过分数来排序。实现的逻辑呢，如果你感兴趣，也可以看一下跳表这个数据结构，sorted_set就是通过这个结构来实现高效的增删改查的。

我在记这些命令的时候，联想到了 list的一些命令，这两个结构的命令有点相似。



### 增

zadd key score member

### 查

zcard key 查询元素个数、

zcount key min max 查询指定范围内元素个数

zrange key start end

zrank key member 查询元素是否在集合中

### 改

zincrby key increment member 增加

### 删

zpopmax key 

zpopmin key

zrem key member

zremrangebyscore key min max

zremrangebyrank key start stop

## redis进阶使用

redis 事务

[MULTI](http://redis.cn/commands/multi.html) 、 [EXEC](http://redis.cn/commands/exec.html) 、 [DISCARD](http://redis.cn/commands/discard.html) 和 [WATCH](http://redis.cn/commands/watch.html) 是 Redis 事务相关的命令

redis事务是不支持回滚操作的

## 使用 check-and-set 操作实现乐观锁

[WATCH](http://redis.cn/commands/watch.html) 命令可以为 Redis 事务提供 check-and-set （CAS）行为。

被 [WATCH](http://redis.cn/commands/watch.html) 的键会被监视，并会发觉这些键是否被改动过了。 如果有至少一个被监视的键在 [EXEC](http://redis.cn/commands/exec.html) 执行之前被修改了， 那么整个事务都会被取消， [EXEC](http://redis.cn/commands/exec.html) 返回[nil-reply](http://redis.cn/topics/protocol.html#nil-reply)来表示事务已经失败。

## redis pipline

从Redis 2.6开始`redis-cli`支持一种新的被称之为**pipe mode**的新模式用于执行大量数据插入工作。

```shell
cat data.txt | redis-cli --pipe
```

## redis作为缓存和数据库的差别

### 缓存

1. 数据不重要，并且redis中不是全量数据

2. 数据应该随着访问变化，存储的是热数据

#### 如何保证热数据

1. 业务逻辑保证  key的有效期

   ​	1. 会随着访问延长？不对！！

   ​	2.发生写，会剔除过期时间

   ​	3.倒计时，且，redis不能延长

   ​	4.定时

   ​	5.业务逻辑自己补全

2. 业务运转

   ​	内存是有限随着访问的变化，应该淘汰掉冷数据

3. 过期判定原理

   ​	具体就是Redis每秒10次做的事情：

   1. 测试随机的20个keys进行相关过期检测。
   2. 删除所有已经过期的keys。
   3. 如果有多于25%的keys过期，重复步奏1.

   这是一个平凡的概率算法，基本上的假设是，我们的样本是这个密钥控件，并且我们不断重复过期检测，直到过期的keys的百分百低于25%,这意味着，在任何给定的时刻，最多会清除1/4的过期keys。

4. redis的回收策略

   noeviction:返回错误当内存限制达到并且客户端尝试执行会让更多内存被使用的命令（大部分的写入指令，但DEL和几个例外）
   allkeys-lru: 尝试回收最少使用的键（LRU），使得新添加的数据有空间存放。
   volatile-lru: 尝试回收最少使用的键（LRU），但仅限于在过期集合的键,使得新添加的数据有空间存放。
   allkeys-random: 回收随机的键使得新添加的数据有空间存放。
   volatile-random: 回收随机的键使得新添加的数据有空间存放，但仅限于在过期集合的键。
   volatile-ttl: 回收在过期集合的键，并且优先回收存活时间（TTL）较短的键,使得新添加的数据有空间存放。

### 数据库

1. 如果作为数据库使用，那么就不应该让redis中的数据进行淘汰

   

## redis持久化方案

首先，我们需要知道只要是数据库，那么就会有快照和日志两中方式持久化，redis中分为rdb和aof两种方式

阻塞

<img src="D:\MyWork\MarkDownPicture\all\redis持久化方案.png" alt="redis持久化方案" style="zoom:67%;" />



如果是阻塞的话，那么数据的时点性就无法得到保证

### RDB

​	理解rdb的话，需要理解Linux中的父子进程的概念。

​	父进程可以让子进程看到自己的数据，但是修改的话是隔离的，不会看到。

​	redis这里采用了fork出一个子进程，然后利用了写时复制的技术来进行rdb的落盘

copy on write：内核机制写时复制

创建子进程并不发生复制

​	好处：建进程变快了

​				根据经验，不可能父子进程把所有数据都改一遍

​				玩的是指针

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201214141619064.png" alt="image-20201214141619064" style="zoom:67%;" />

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201214141638614.png" alt="image-20201214141638614" style="zoom: 200%;" />



![image-20201214141720745](C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201214141720745.png)

![image-20201214143021461](C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201214143021461.png)





### AOF

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201214145023878.png" alt="image-20201214145023878" style="zoom:150%;" />

下面是刷盘的机制

appendfsync always

appendfsync everysec

appendfsync no

redis默认的是ererysec

![image-20201214145039034](C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201214145039034.png)





## AKF

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201214163815495.png" alt="image-20201214163815495" style="zoom:67%;" />

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201214163841900.png" alt="image-20201214163841900" style="zoom:67%;" />

## 强一致性

如果需要强一致性，那么会降低可用性

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201214164420427.png" alt="image-20201214164420427" style="zoom: 67%;" />

## 弱一致性

如果使用弱一致性，会丢失数据

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201214164439582.png" alt="image-20201214164439582" style="zoom:67%;" />

## 最终一致性

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201214164513253.png" alt="image-20201214164513253" style="zoom:67%;" />

## redis主从配置

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201214172837108.png" alt="image-20201214172837108" style="zoom:67%;" />

<img src="C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201214172851299.png" alt="image-20201214172851299" style="zoom:67%;" />





## 缓存常见问题

### 击穿

​	![image-20201217165813573](C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201217165813573.png)

### 穿透

​		![image-20201217170106641](C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201217170106641.png)

### 雪崩

![image-20201217170120584](C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20201217170120584.png)

https://github.com/smalldogg/MarkDownPicture/raw/master/reids/%E9%9B%AA%E5%B4%A9.jpg