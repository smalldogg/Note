# RocketMQ之六：RocketMQ消息存储

## 一、RocketMQ的消息存储基本介绍

先看一张图：

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190823175456557-1759238231.png)

1、Commit log存储消息实体。特点**顺序写，随机读**
2、Message queue存储消息的偏移量。读消息先读message queue，根据偏移量到commit log读消息本身。
3、索引队列用来存储消息的索引key
使用mmap方式减少内存拷贝，提高读取性能。具体实现：FileChannel.map(RandomAccessFile)

 

CommitLog以物理文件的方式存放，每台Broker上的CommitLog被本机器所有ConsumeQueue共享。

在CommitLog，一个消息的存储长度是不固定的，RocketMQ采用了一些机制，尽量向CommitLog中**顺序写，但是随即读**。

[ 磁盘存储的“快”——顺序写 ] 

 磁盘存储，使用得当，磁盘的速度完全可以匹配上网络的数据传输速度，目前的高性能磁盘，顺序写速度可以达到600MB/s，超过了一般网卡的传输速度。

[ 磁盘存储的“慢”——随机写 ]

磁盘的随机写的速度只有100KB/s，和顺序写的性能差了好几个数量级。

 

 [ 存储机制这样设计的好处——顺序写，随机读 ]

1.CommitLog顺序写，可以大大提高写入的效率；

2.虽然是随机读，但是利用package机制，可以批量地从磁盘读取，作为cache存到内存中，加速后续的读取速度。

3.为了保证完全的顺序写，需要ConsumeQueue这个中间结构，因为ConsumeQueue里只存储偏移量信息，所以尺寸是有限的。在实际情况中，大部分ConsumeQueue能够被全部读入内存，所以这个中间结构的操作速度很快，可以认为是内存读取的速度。

[ 如何保证CommitLog和ConsumeQueue的一致性？ ]

**CommitLog里存储了Consume Queues、Message Queue、Tag等所有信息，即使ConsumeQueue丢失，也可以通过commitLog完全恢复出来**

 

RocketMQ的Broker机器磁盘上的文件存储结构

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190828164219580-643002097.png)

 

### 1.1、RocketMQ的消息存储主要有如下概念：

**（1）CommitLog**：消息主体以及元数据的存储主体，存储Producer端写入的消息主体内容。单个文件大小默认1G ，文件名长度为20位，左边补零，剩余为起始偏移量，比如00000000000000000000代表了第一个文件，起始偏移量为0，文件大小为1G=1073741824；当第一个文件写满了，第二个文件为00000000001073741824，起始偏移量为1073741824，以此类推。消息主要是顺序写入日志文件，当文件满了，写入下一个文件；
**（2） ConsumeQueue**：消息消费的逻辑队列，作为消费消息的索引，保存了指定Topic下的队列消息在CommitLog中的起始物理偏移量offset，消息大小size和消息Tag的HashCode值。而IndexFile（索引文件）则只是为了消息查询提供了一种通过key或时间区间来查询消息的方法（ps：这种通过IndexFile来查找消息的方法不影响发送与消费消息的主流程）。从实际物理存储来说，ConsumeQueue对应每个Topic和QueuId下面的文件。单个文件大小约5.72M，每个文件由30W条数据组成，每个文件默认大小为600万个字节，当一个ConsumeQueue类型的文件写满了，则写入下一个文件；
**（3）IndexFile**：因为所有的消息都存在CommitLog中，如果要实现根据 key 查询 消息的方法，就会变得非常困难，所以为了解决这种业务需求，有了IndexFile的存在。用于为生成的索引文件提供访问服务，通过消息Key值查询消息真正的实体内容。在实际的物理存储上，文件名则是以创建时的时间戳命名的，固定的单个IndexFile文件大小约为400M，一个IndexFile可以保存 2000W个索引；
**（4）MapedFileQueue**：对连续物理存储的抽象封装类，源码中可以通过消息存储的物理偏移量位置快速定位该offset所在MappedFile(具体物理存储位置的抽象)、创建、删除MappedFile等操作；
**（5）MappedFile**：文件存储的直接内存映射业务抽象封装类，源码中通过操作该类，可以把消息字节写入PageCache缓存区（commit），或者原子性地将消息持久化的刷盘（flush）；

### 1.2、RocketMQ消息刷盘的主要过程

CommitLog写入：

MapedFileQueue 存储队列，数据定时删除，无限增长。
队列有多个文件（MapedFile）组成
当消息到达broker时，需要获取最新的MapedFile写入数据，调用MapedFileQueue的getLastMapedFile获取，此函数如果集合中一个也没有创建一个，如果最后一个写满了也创建一个新的。
 MapedFileQueue在获取getLastMapedFile时，如果需要创建新的MapedFile会计算出下一个MapedFile文件地址，通过预分配服务AllocateMapedFileService异步预创建下一个MapedFile文件，这样下次创建新文件请求就不要等待，因为创建文件特别是一个1G的文件还是有点耗时的，
后续如果是异步刷盘还需要将mapedFile中的消息序列化到commitLog物理文件

consumeQueue写入：

也采用mappedFile文件内存映射。
底层也用与commitLog相同的MapedFileQueue数据结构。
consume queue中存储单元是一个20字节定长的数据，是顺序写顺序读
（1）   commitLogOffset是指这条消息在commitLog文件实际偏移量
（2）   size就是指消息大小
（3）   消息tag的哈希值



在RocketMQ中消息刷盘主要可以分为同步刷盘和异步刷盘两种：

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190821181836976-2060579912.png)

**（1）同步刷盘**：如上图所示，只有在消息真正持久化至磁盘后，RocketMQ的Broker端才会真正地返回给Producer端一个成功的ACK响应。同步刷盘对MQ消息可靠性来说是一种不错的保障，但是性能上会有较大影响，一般适用于金融业务应用领域。RocketMQ同步刷盘的大致做法是，基于生产者消费者模型，主线程创建刷盘请求实例—GroupCommitRequest并在放入刷盘写队列后唤醒同步刷盘线程—GroupCommitService，来执行刷盘动作（其中用了CAS变量和CountDownLatch来保证线程间的同步）。这里，RocketMQ源码中用读写双缓存队列（requestsWrite/requestsRead）来实现读写分离，其带来的好处在于内部消费生成的同步刷盘请求可以不用加锁，提高并发度。
**（2）异步刷盘**：能够充分利用OS的PageCache的优势，只要消息写入PageCache即可将成功的ACK返回给Producer端。消息刷盘采用后台异步线程提交的方式进行，降低了读写延迟，提高了MQ的性能和吞吐量。异步和同步刷盘的区别在于，异步刷盘时，主线程并不会阻塞，在将刷盘线程wakeup后，就会继续执行。

### 1.3、几个主要的组件说明

#### 1.3.1、ConsumeQueue

consumeQueue是消息的**逻辑队列**，相当于字典的目录，用来指定消息在物理文件`commitLog`上的位置。其中包含了这个MessageQueue在CommitLog中的起始物理位置偏移量offset，消息实体内容的大小和Message Tag的哈希值。从实际物理存储来说，ConsumeQueue对应每个Topic和QueuId下面的文件。单个文件大小约5.72M，每个文件由30W条数据组成，每个文件默认大小为600万个字节，当一个ConsumeQueue类型的文件写满了，则写入下一个文件；

我们可以在配置中指定consumequeue与commitlog存储的目录
每个`topic`下的每个`queue`都有一个对应的`consumequeue`文件，比如：

```
${rocketmq.home}/store/consumequeue/${topicName}/${queueId}/${fileName}
```

Consume Queue文件组织，如图所示：

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190821180613661-1384751844.png)

Consume Queue文件组织示意图

1. **根据`topic`和`queueId`来组织文件**，图中TopicA有两个队列0,1，那么TopicA和QueueId=0组成一个ConsumeQueue，TopicA和QueueId=1组成另一个ConsumeQueue。
2. 按照消费端的`GroupName`来分组重试队列，如果消费端消费失败，消息将被发往重试队列中，比如图中的`%RETRY%ConsumerGroupA`。
3. 按照消费端的`GroupName`来分组死信队列，如果消费端消费失败，并重试指定次数后，仍然失败，则发往死信队列，比如图中的`%DLQ%ConsumerGroupA`。

> 死信队列（Dead Letter Queue）一般用于存放由于某种原因无法传递的消息，比如处理失败或者已经过期的消息。

Consume Queue中存储单元是一个20字节定长的二进制数据，顺序写顺序读，如下图所示：

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190821174157920-1786672949.png)

Queue单个存储单元结构

consume queue文件存储单元格式

1. CommitLog Offset是指这条消息在Commit Log文件中的实际偏移量
2. Size存储中消息的大小
3. Message Tag HashCode存储消息的Tag的哈希值：主要用于订阅时消息过滤（订阅时如果指定了Tag，会根据HashCode来快速查找到订阅的消息）

#### 1.3.2、Commit Log

CommitLog：消息主体以及元数据的存储主体，存储Producer端写入的消息主体内容。消息存放的物理文件，每台`broker`上的`commitlog`被本机所有的`queue`共享，不做任何区分。
文件的默认位置如下，仍然可通过配置文件修改：

```
${user.home} \store\${commitlog}\${fileName}
```

CommitLog的消息存储单元长度不固定，文件顺序写，随机读。消息的存储结构如下表所示，按照编号顺序以及编号对应的内容依次存储。

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190821174643136-502479113.png)

#### 1.3.3、**IndexFile**消息的索引文件

IndexFile：用于为生成的索引文件提供访问服务，通过消息Key值查询消息真正的实体内容。在实际的物理存储上，文件名则是以创建时的时间戳命名的，固定的单个IndexFile文件大小约为400M，一个IndexFile可以保存 2000W个索引；

indexFile存放的位置：`${rocketmq.home}/store/index/indexFile(年月日时分秒等组成文件名)`

#### Index生成过程

上一篇(https://www.jianshu.com/p/606d4b77d504)讲`ConsumeQueue`的时候，有一个`ReputMessageService`在分发消息的时候还会调用`CommitLogDispatcherBuildIndex`用来创建index。这个类实现就是直接调用的`IndexService.buildIndex()`

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190823180833036-1445356762.png)

如果一个消息包含key值的话，会使用IndexFile存储消息索引，文件的内容结构如图：

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190821174809854-1711162648.png)

消息索引

索引文件主要用于根据key来查询消息的，流程主要是：

1. 1. 根据查询的 key 的 hashcode%slotNum 得到具体的槽的位置(slotNum 是一个索引文件里面包含的最大槽的数目，例如图中所示 slotNum=5000000)
   2. 根据 slotValue(slot 位置对应的值)查找到索引项列表的最后一项(倒序排列,slotValue 总是指向最新的一个索引项)
   3. 遍历索引项列表返回查询时间范围内的结果集(默认一次最大返回的 32 条记录)

## 二、RocketMQ的消息存储原理

消息存储是MQ消息队列中最为复杂和最为重要的一部分，本文先从目前几种比较常用的MQ消息队列存储方式出发，为大家介绍RocketMQ选择磁盘文件存储的原因。然后，本文分别从RocketMQ的消息存储整体架构和RocketMQ文件存储模型层次结构两方面进行深入分析介绍。使得大家读完本文后对RocketMQ消息存储部分有一个大致的了解和认识。

### 2.1、MQ消息队列的一般存储方式

当前业界几款主流的MQ消息队列采用的存储方式主要有以下三种方式：
**（1）分布式KV存储：**这类MQ一般会采用诸如levelDB、RocksDB和Redis来作为消息持久化的方式，由于分布式缓存的读写能力要优于DB，所以在对消息的读写能力要求都不是比较高的情况下，采用这种方式倒也不失为一种可以替代的设计方案。消息存储于分布式KV需要解决的问题在于如何保证MQ整体的可靠性？
**（2）文件系统：**目前业界较为常用的几款产品（RocketMQ/Kafka/RabbitMQ）均采用的是消息刷盘至所部署虚拟机/物理机的文件系统来做持久化（刷盘一般可以分为异步刷盘和同步刷盘两种模式）。小编认为，消息刷盘为消息存储提供了一种高效率、高可靠性和高性能的数据持久化方式。除非部署MQ机器本身或是本地磁盘挂了，否则一般是不会出现无法持久化的故障问题。
**（3）关系型数据库DB：**Apache下开源的另外一款MQ—ActiveMQ（默认采用的KahaDB做消息存储）可选用JDBC的方式来做消息持久化，通过简单的xml配置信息即可实现JDBC消息存储。由于，普通关系型数据库（如Mysql）在单表数据量达到千万级别的情况下，其IO读写性能往往会出现瓶颈。因此，如果要选型或者自研一款性能强劲、吞吐量大、消息堆积能力突出的MQ消息队列，那么小编并不推荐采用关系型数据库作为消息持久化的方案。在可靠性方面，该种方案非常依赖DB，如果一旦DB出现故障，则MQ的消息就无法落盘存储会导致线上故障；
因此，综合上所述从存储效率来说， **文件系统>分布式KV存储>关系型数据库DB**，直接操作文件系统肯定是最快和最高效的，而关系型数据库TPS一般相比于分布式KV系统会更低一些（简略地说，关系型数据库本身也是一个需要读写文件server，这时MQ作为client与其建立连接并发送待持久化的消息数据，同时又需要依赖DB的事务等，这一系列操作都比较消耗性能），所以如果追求高效的IO读写，那么选择操作文件系统会更加合适一些。但是如果从易于实现和快速集成来看，**关系型数据库DB>分布式KV存储>文件系统**，但是性能会下降很多。
另外，从消息中间件的本身定义来考虑，应该尽量减少对于外部第三方中间件的依赖。一般来说依赖的外部系统越多，也会使得本身的设计越复杂，所以小编个人的理解是采用**文件系统**作为消息存储的方式，更贴近消息中间件本身的定义。

### 2.2、RocketMQ消息存储整体架构

消息存储实现，比较复杂，也值得大家深入了解，后面会单独成文来分析，这小节只以代码说明一下具体的流程。

```
// Set the storage time
msg.setStoreTimestamp(System.currentTimeMillis());
// Set the message body BODY CRC (consider the most appropriate setting
msg.setBodyCRC(UtilAll.crc32(msg.getBody()));
StoreStatsService storeStatsService = this.defaultMessageStore.getStoreStatsService();
synchronized (this) {
    long beginLockTimestamp = this.defaultMessageStore.getSystemClock().now();
    // Here settings are stored timestamp, in order to ensure an orderly global
    msg.setStoreTimestamp(beginLockTimestamp);
    // MapedFile：操作物理文件在内存中的映射以及将内存数据持久化到物理文件中
    MapedFile mapedFile = this.mapedFileQueue.getLastMapedFile();
    // 将Message追加到文件commitlog
    result = mapedFile.appendMessage(msg, this.appendMessageCallback);
    switch (result.getStatus()) {
    case PUT_OK:break;
    case END_OF_FILE:
         // Create a new file, re-write the message
         mapedFile = this.mapedFileQueue.getLastMapedFile();
         result = mapedFile.appendMessage(msg, this.appendMessageCallback);
     break;
     DispatchRequest dispatchRequest = new DispatchRequest(
                topic,// 1
                queueId,// 2
                result.getWroteOffset(),// 3
                result.getWroteBytes(),// 4
                tagsCode,// 5
                msg.getStoreTimestamp(),// 6
                result.getLogicsOffset(),// 7
                msg.getKeys(),// 8
                /**
                 * Transaction
                 */
                msg.getSysFlag(),// 9
                msg.getPreparedTransactionOffset());// 10
    // 1.分发消息位置到ConsumeQueue
    // 2.分发到IndexService建立索引
    this.defaultMessageStore.putDispatchRequest(dispatchRequest);
}
```

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190821163517562-457444235.png)

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190821175934839-32517554.png)

#### （1）RocketMQ消息存储结构类型及缺点

上图即为RocketMQ的消息存储整体架构，RocketMQ采用的是混合型的存储结构，即为Broker单个实例下所有的队列共用一个日志数据文件（即为CommitLog）来存储。而Kafka采用的是独立型的存储结构，每个队列一个文件。**这里小编认为，RocketMQ采用混合型存储结构的缺点在于，会存在较多的随机读操作，因此读的效率偏低。同时消费消息需要依赖ConsumeQueue，构建该逻辑消费队列需要一定开销。**

上面图中假设Consumer端默认设置的是同一个ConsumerGroup，因此Consumer端线程采用的是负载订阅的方式进行消费。从架构图中可以总结出如下几个关键点：
（1）**消息生产与消息消费相互分离**，Producer端发送消息最终写入的是CommitLog（消息存储的日志数据文件），Consumer端先从ConsumeQueue（消息逻辑队列）读取持久化消息的起始物理位置偏移量offset、大小size和消息Tag的HashCode值，随后再从CommitLog中进行读取待拉取消费消息的真正实体内容部分；
（2）**RocketMQ的CommitLog文件采用混合型存储**（所有的Topic下的消息队列共用同一个CommitLog的日志数据文件），并通过建立类似索引文件—ConsumeQueue的方式来区分不同Topic下面的不同MessageQueue的消息，同时为消费消息起到一定的缓冲作用（只有ReputMessageService异步服务线程通过doDispatch异步生成了ConsumeQueue队列的元素后，Consumer端才能进行消费）。这样，只要消息写入并刷盘至CommitLog文件后，消息就不会丢失，即使ConsumeQueue中的数据丢失，也可以通过CommitLog来恢复。
（3）**RocketMQ每次读写文件的时候真的是完全顺序读写么？**这里，发送消息时，生产者端的消息确实是**顺序写入CommitLog**；订阅消息时，消费者端也是**顺序读取ConsumeQueue**，然而根据其中的起始物理位置偏移量offset读取消息真实内容却是**随机读取CommitLog**。 在RocketMQ集群整体的吞吐量、并发量非常高的情况下，随机读取文件带来的性能开销影响还是比较大的，那么这里如何去优化和避免这个问题呢？后面的章节将会逐步来解答这个问题。
这里，同样也可以总结下RocketMQ存储架构的优缺点：
（1）**优点：**
　　　　a、ConsumeQueue消息逻辑队列较为轻量级；
　　　　b、对磁盘的访问串行化，避免磁盘竟争，不会因为队列增加导致IOWAIT增高；
（2）**缺点：**
　　　　a、对于CommitLog来说写入消息虽然是顺序写，但是读却变成了完全的随机读；
　　　　b、Consumer端订阅消费一条消息，需要先读ConsumeQueue，再读Commit Log，一定程度上增加了开销；

#### （2）RocketMQ消息存储架构深入分析

从上面的整体架构图中可见，RocketMQ的混合型存储结构针对Producer和Consumer分别采用了数据和索引部分相分离的存储结构，

Producer端：Producer发送消息至Broker端，然后Broker端使用同步或者异步的方式对消息刷盘持久化，保存至CommitLog中。只要消息被刷盘持久化至磁盘文件CommitLog中，那么Producer发送的消息就不会丢失。

Consumer端：Consumer也就肯定有机会去消费这条消息，至于消费的时间可以稍微滞后一些也没有太大的关系。退一步地讲，即使Consumer端第一次没法拉取到待消费的消息，Broker服务端也能够通过长轮询机制等待一定时间延迟后再次发起拉取消息的请求。这里，RocketMQ的具体做法是，使用Broker端的后台服务线程—ReputMessageService不停地分发请求并异步构建ConsumeQueue（逻辑消费队列）和IndexFile（索引文件）数据（ps：对于该服务线程在消息消费篇幅也有过介绍，不清楚的童鞋可以跳至消息消费篇幅再理解下）。然后，Consumer即可根据ConsumerQueue来查找待消费的消息了。其中，ConsumeQueue（逻辑消费队列）作为消费消息的索引，保存了指定Topic下的队列消息在CommitLog中的起始物理偏移量offset，消息大小size和消息Tag的HashCode值。而IndexFile（索引文件）则只是为了消息查询提供了一种通过key或时间区间来查询消息的方法（ps：这种通过IndexFile来查找消息的方法不影响发送与消费消息的主流程）。

#### （3）PageCache与Mmap内存映射

这里有必要先稍微简单地介绍下page cache的概念。系统的所有文件I/O请求，操作系统都是通过page cache机制实现的。对于操作系统来说，磁盘文件都是由一系列的数据块顺序组成，数据块的大小由操作系统本身而决定，x86的linux中一个标准页面大小是4KB。
操作系统内核在处理文件I/O请求时，首先到page cache中查找（page cache中的每一个数据块都设置了文件以及偏移量地址信息），如果未命中，则启动磁盘I/O，将磁盘文件中的数据块加载到page cache中的一个空闲块，然后再copy到用户缓冲区中。
page cache本身也会对数据文件进行预读取，对于每个文件的第一个读请求操作，系统在读入所请求页面的同时会读入紧随其后的少数几个页面。因此，想要提高page cache的命中率（尽量让访问的页在物理内存中），从硬件的角度来说肯定是物理内存越大越好。从操作系统层面来说，访问page cache时，即使只访问1k的消息，系统也会提前预读取更多的数据，在下次读取消息时, 就很可能可以命中内存。

##### Mmap内存映射技术—MappedByteBuffer

**（a）Mmap内存映射技术的特点**
Mmap内存映射和普通标准IO操作的本质区别在于它并不需要将文件中的数据先拷贝至OS的内核IO缓冲区，而是可以直接将用户进程私有地址空间中的一块区域与文件对象建立映射关系，这样程序就好像可以直接从内存中完成对文件读/写操作一样。只有当缺页中断发生时，直接将文件从磁盘拷贝至用户态的进程空间内，只进行了一次数据拷贝。对于容量较大的文件来说（文件大小一般需要限制在1.5~2G以下），采用Mmap的方式其读/写的效率和性能都非常高。

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190821181257157-1420314377.png)

**（b）JDK NIO的MappedByteBuffer简要分析**
从JDK的源码来看，MappedByteBuffer继承自ByteBuffer，其内部维护了一个逻辑地址变量—address。在建立映射关系时，MappedByteBuffer利用了JDK NIO的FileChannel类提供的map()方法把文件对象映射到虚拟内存。仔细看源码中map()方法的实现，可以发现最终其通过调用native方法map0()完成文件对象的映射工作，同时使用Util.newMappedByteBuffer()方法初始化MappedByteBuffer实例，但最终返回的是DirectByteBuffer的实例。在Java程序中使用MappedByteBuffer的get()方法来获取内存数据是最终通过DirectByteBuffer.get()方法实现（底层通过unsafe.getByte()方法，以“地址 + 偏移量”的方式获取指定映射至内存中的数据）。
**（c）使用Mmap的限制**
**a.Mmap映射的内存空间释放的问题**；由于映射的内存空间本身就不属于JVM的堆内存区（Java Heap），因此其不受JVM GC的控制，卸载这部分内存空间需要通过系统调用 unmap()方法来实现。然而unmap()方法是FileChannelImpl类里实现的私有方法，无法直接显示调用。**RocketMQ中的做法是**，通过Java反射的方式调用“sun.misc”包下的Cleaner类的clean()方法来释放映射占用的内存空间；
**b.MappedByteBuffer内存映射大小限制**；因为其占用的是虚拟内存（非JVM的堆内存），大小不受JVM的-Xmx参数限制，但其大小也受到OS虚拟内存大小的限制。一般来说，一次只能映射1.5~2G 的文件至用户态的虚拟内存空间，这也是为何RocketMQ默认设置单个CommitLog日志数据文件为1G的原因了；
**c.使用MappedByteBuffe的其他问题**；会存在内存占用率较高和文件关闭不确定性的问题；

##### OS的PageCache机制

PageCache是OS对文件的缓存，用于加速对文件的读写。一般来说，程序对文件进行顺序读写的速度几乎接近于内存的读写访问，这里的主要原因就是在于OS使用PageCache机制对读写访问操作进行了性能优化，将一部分的内存用作PageCache。
（1）**对于数据文件的读取**，如果一次读取文件时出现未命中PageCache的情况，OS从物理磁盘上访问读取文件的同时，会顺序对其他相邻块的数据文件进行预读取（ps：顺序读入紧随其后的少数几个页面）。这样，只要下次访问的文件已经被加载至PageCache时，读取操作的速度基本等于访问内存。
（2）**对于数据文件的写入**，OS会先写入至Cache内，随后通过异步的方式由pdflush内核线程将Cache内的数据刷盘至物理磁盘上。
对于文件的顺序读写操作来说，读和写的区域都在OS的PageCache内，此时读写性能接近于内存。**RocketMQ的大致做法是**，将数据文件映射到OS的虚拟内存中（通过JDK NIO的MappedByteBuffer），写消息的时候首先写入PageCache，并通过异步刷盘的方式将消息批量的做持久化（同时也支持同步刷盘）；订阅消费消息时（对CommitLog操作是随机读取），由于PageCache的局部性热点原理且整体情况下还是从旧到新的有序读，因此大部分情况下消息还是可以直接从Page Cache中读取，不会产生太多的缺页（Page Fault）中断而从磁盘读取。

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190821181432326-191404056.jpg)

PageCache机制也不是完全无缺点的，当遇到OS进行脏页回写，内存回收，内存swap等情况时，就会引起较大的消息读写延迟。
对于这些情况，RocketMQ采用了多种优化技术，比如内存预分配，文件预热，mlock系统调用等，来保证在最大可能地发挥PageCache机制优点的同时，尽可能地减少其缺点带来的消息读写延迟。





在RocketMQ中，ConsumeQueue逻辑消费队列存储的数据较少，并且是顺序读取，在page cache机制的预读取作用下，Consume Queue的读性能会比较高近乎内存，即使在有消息堆积情况下也不会影响性能。而对于CommitLog消息存储的日志数据文件来说，读取消息内容时候会产生较多的随机访问读取，严重影响性能。如果选择合适的系统IO调度算法，比如设置调度算法为“Noop”（此时块存储采用SSD的话），随机读的性能也会有所提升。

另外，RocketMQ主要通过MappedByteBuffer对文件进行读写操作。其中，利用了NIO中的FileChannel模型直接将磁盘上的物理文件直接映射到用户态的内存地址中（这种Mmap的方式减少了传统IO将磁盘文件数据在操作系统内核地址空间的缓冲区和用户应用程序地址空间的缓冲区之间来回进行拷贝的性能开销），将对文件的操作转化为直接对内存地址进行操作，从而极大地提高了文件的读写效率（**这里需要注意的是，采用MappedByteBuffer这种内存映射的方式有几个限制，其中之一是一次只能映射1.5~2G 的文件至用户态的虚拟内存，这也是为何RocketMQ默认设置单个CommitLog日志数据文件为1G的原因了**）。

### 2.3、RocketMQ文件存储模型层次结构

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190821164221288-1795439808.png)


RocketMQ文件存储模型层次结构如上图所示，根据类别和作用从概念模型上大致可以划分为5层，下面将从各个层次分别进行分析和阐述：
（1）**RocketMQ业务处理器层**：Broker端对消息进行读取和写入的业务逻辑入口，这一层主要包含了业务逻辑相关处理操作（根据解析RemotingCommand中的RequestCode来区分具体的业务操作类型，进而执行不同的业务处理流程），比如前置的检查和校验步骤、构造MessageExtBrokerInner对象、decode反序列化、构造Response返回对象等；
（2）**RocketMQ数据存储组件层**；该层主要是RocketMQ的存储核心类—DefaultMessageStore，其为RocketMQ消息数据文件的访问入口，通过该类的“putMessage()”和“getMessage()”方法完成对CommitLog消息存储的日志数据文件进行读写操作（具体的读写访问操作还是依赖下一层中CommitLog对象模型提供的方法）；另外，在该组件初始化时候，还会启动很多存储相关的后台服务线程，包括AllocateMappedFileService（MappedFile预分配服务线程）、ReputMessageService（回放存储消息服务线程）、HAService（Broker主从同步高可用服务线程）、StoreStatsService（消息存储统计服务线程）、IndexService（索引文件服务线程）等；
（3）**RocketMQ存储逻辑对象层**：该层主要包含了RocketMQ数据文件存储直接相关的三个模型类IndexFile、ConsumerQueue和CommitLog。IndexFile为索引数据文件提供访问服务，ConsumerQueue为逻辑消息队列提供访问服务，CommitLog则为消息存储的日志数据文件提供访问服务。这三个模型类也是构成了RocketMQ存储层的整体结构（对于这三个模型类的深入分析将放在后续篇幅中）；
（4）**封装的文件内存映射层**：RocketMQ主要采用JDK NIO中的MappedByteBuffer和FileChannel两种方式完成数据文件的读写。其中，采用MappedByteBuffer这种内存映射磁盘文件的方式完成对大文件的读写，在RocketMQ中将该类封装成MappedFile类。这里限制的问题在上面已经讲过；对于每类大文件（IndexFile/ConsumerQueue/CommitLog），在存储时分隔成多个固定大小的文件（**单个IndexFile文件大小约为400M、单个ConsumerQueue文件大小约5.72M、单个CommitLog文件大小为1G**），其中每个分隔文件的文件名为前面所有文件的字节大小数+1，即为文件的起始偏移量，从而实现了整个大文件的串联。这里，每一种类的单个文件均由MappedFile类提供读写操作服务（其中，MappedFile类提供了顺序写/随机读、内存数据刷盘、内存清理等和文件相关的服务）；
（5）**磁盘存储层**：主要指的是部署RocketMQ服务器所用的磁盘。这里，需要考虑不同磁盘类型（如SSD或者普通的HDD）特性以及磁盘的性能参数（如IOPS、吞吐量和访问时延等指标）对顺序写/随机读操作带来的影响（ps：小编建议在正式业务上线之前做好多轮的性能压测，具体用压测的结果来评测）；





# 三、RocketMQ存储优化技术

这一节将主要介绍RocketMQ存储层采用的几项优化技术方案在一定程度上可以减少PageCache的缺点带来的影响，主要包括内存预分配，文件预热和mlock系统调用。

## 3.1 预先分配MappedFile

在消息写入过程中（调用CommitLog的putMessage()方法），CommitLog会先从MappedFileQueue队列中获取一个 MappedFile，如果没有就新建一个。
这里，MappedFile的创建过程是将构建好的一个AllocateRequest请求（具体做法是，将下一个文件的路径、下下个文件的路径、文件大小为参数封装为AllocateRequest对象）添加至队列中，后台运行的AllocateMappedFileService服务线程（在Broker启动时，该线程就会创建并运行），会不停地run，只要请求队列里存在请求，就会去执行MappedFile映射文件的创建和预分配工作，分配的时候有两种策略，一种是使用Mmap的方式来构建MappedFile实例，另外一种是从TransientStorePool堆外内存池中获取相应的DirectByteBuffer来构建MappedFile（ps：具体采用哪种策略，也与刷盘的方式有关）。并且，在创建分配完下个MappedFile后，还会将下下个MappedFile预先创建并保存至请求队列中等待下次获取时直接返回。**RocketMQ中预分配MappedFile的设计非常巧妙，下次获取时候直接返回就可以不用等待MappedFile创建分配所产生的时间延迟。**

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190821182636899-322299478.png)

预分配MappedFile的主要过程.jpg

## 3.2 文件预热&&mlock系统调用

**（1）mlock系统调用**：其可以将进程使用的部分或者全部的地址空间锁定在物理内存中，防止其被交换到swap空间。对于RocketMQ这种的高吞吐量的分布式消息队列来说，追求的是消息读写低延迟，那么肯定希望尽可能地多使用物理内存，提高数据读写访问的操作效率。
**（2）文件预热**：预热的目的主要有两点；第一点，由于仅分配内存并进行mlock系统调用后并不会为程序完全锁定这些内存，因为其中的分页可能是写时复制的。因此，就有必要对每个内存页面中写入一个假的值。其中，RocketMQ是在创建并分配MappedFile的过程中，预先写入一些随机值至Mmap映射出的内存空间里。第二，调用Mmap进行内存映射后，OS只是建立虚拟内存地址至物理地址的映射表，而实际并没有加载任何文件至内存中。程序要访问数据时OS会检查该部分的分页是否已经在内存中，如果不在，则发出一次缺页中断。这里，可以想象下1G的CommitLog需要发生多少次缺页中断，才能使得对应的数据才能完全加载至物理内存中（ps：X86的Linux中一个标准页面大小是4KB）？**RocketMQ的做法是**，在做Mmap内存映射的同时进行madvise系统调用，目的是使OS做一次内存映射后对应的文件数据尽可能多的预加载至内存中，从而达到内存预热的效果。

