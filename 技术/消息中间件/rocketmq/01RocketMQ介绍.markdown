# 一、RocketMQ介绍

RocketMQ 是阿里巴巴开源的分布式消息中间件。支持事务消息、顺序消息、批量消息、定时消息、消息回溯等。它里面有几个区别于标准消息中件间的概念，如Group、Topic、Queue等。系统组成则由Producer、Consumer、Broker、NameServer等。

**RocketMQ 特点**

- 是一个队列模型的消息中间件，具有高性能、高可靠、高实时、分布式等特点
- Producer、Consumer、队列都可以分布式
- Producer 向一些队列轮流发送消息，队列集合称为 Topic，Consumer 如果做广播消费，则一个 Consumer 实例消费这个 Topic 对应的所有队列，如果做集群消费，则多个 Consumer 实例平均消费这个 Topic 对应的队列集合
- 能够保证严格的消息顺序
- 支持拉（pull）和推（push）两种消息模式
- 高效的订阅者水平扩展能力
- 实时的消息订阅机制
- 亿级消息堆积能力
- 支持多种消息协议，如 JMS、OpenMessaging 等
- 较少的依赖

## 1.1、RocketMQ的核心概念

　　消息队列 RocketMQ 在任何一个环境都是可扩展的，生产者必须是一个集群，消息服务器必须是一个集群，消费者也同样。集群级别的高可用，是消息队列 RocketMQ 跟其他的消息服务器的主要区别，消息生产者发送一条消息到消息服务器，消息服务器会随机的选择一个消费者，只要这个消费者消费成功就认为是成功了。

**注意：**文中所提及的消息队列 RocketMQ 的服务端或者服务器包含 Name Server、Broker 等。服务端不等同于 Broker。

 ![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190821175219196-1211062486.png)

RocketMQ主要由 Producer、Broker、Consumer 三部分组成，其中Producer 负责生产消息，Consumer 负责消费消息，Broker 负责存储消息。Broker 在实际部署过程中对应一台服务器，每个 Broker 可以存储多个Topic的消息，每个Topic的消息也可以分片存储于不同的 Broker。Message Queue 用于存储消息的物理地址，每个Topic中的消息地址存储于多个 Message Queue 中。ConsumerGroup 由多个Consumer 实例构成。

图中所涉及到的概念如下所述：

1. **Name Server**: 名称服务充当路由消息的提供者

   是一个几乎无状态节点，可集群部署，节点之间无任何信息同步。在消息队列 RocketMQ 中提供命名服务，更新和发现 Broker 服务。

   为什么不使用zk呢？

    **NameServer**

   即名称服务，两个功能：

   1. 1. - 接收`broker`的请求，注册`broker`的路由信息
         - 接收`client（producer/consumer）`的请求，根据某个`topic`获取其到`broker`的路由信息
           `NameServer`没有状态，可以横向扩展。每个`broker`在启动的时候会到`NameServer`注册；`Producer`在发送消息前会根据`topic`到`NameServer`获取路由(到`broker`)信息；`Consumer`也会定时获取`topic`路由信息。

2. **Broker**：消息中转角色，负责存储消息，转发消息

   可以理解为消息队列服务器，提供了消息的接收、存储、拉取和转发服务。

   **broker**
   
   是RocketMQ的核心，它是不能挂的，

   所以需要保证`broker`的高可用。

   　　　　broker分为 Master Broker 和 Slave Broker，一个 Master Broker 可以对应多个 Slave Broker，但是一个 Slave Broker 只能对应一个 Master Broker。

   　　　　Master与Slave的对应关系通过指定相同的BrokerName，不同的BrokerId来定义，BrokerId为0表示Master，非0表示Slave。Master也可以部署多个。

   　　　　每个Broker与Name Server集群中的所有节点建立**长连接**，定时注册Topic信息到所有Name Server。Broker 启动后需要完成一次将自己注册至 Name Server 的操作；随后每隔 30s 定期向 Name Server 上报 Topic 路由信息。

3. **生产者**：与 Name Server 集群中的其中一个节点（随机）建立长链接（Keep-alive），定期从 Name Server 读取 Topic 路由信息，并向提供 Topic 服务的 Master Broker 建立长链接，且定时向 Master Broker 发送心跳。

4. **消费者**：与 Name Server 集群中的其中一个节点（随机）建立长连接，定期从 Name Server 拉取 Topic 路由信息，并向提供 Topic 服务的 Master Broker、Slave Broker 建立长连接，且定时向 Master Broker、Slave Broker 发送心跳。Consumer 既可以从 Master Broker 订阅消息，也可以从 Slave Broker 订阅消息，订阅规则由 Broker 配置决定。

另外，Broker中还存在一些非常重要的名词需要说明：

- **2.1、Topic、Queue、tags**

RocketMQ的Topic/Queue和JMS中的Topic/Queue概念有一定的差异，JMS中所有消费者都会消费一个Topic消息的副本，而Queue中消息只会被一个消费者消费；但到了**RocketMQ中Topic只代表普通的消息队列，而Queue是组成Topic的更小单元**。

这里rocketmq只有点对点的模型。但是对外显示的是topic

- **topic：**表示消息的第一级类型，比如一个电商系统的消息可以分为：交易消息、物流消息...... 一条消息必须有一个`Topic。`
- **Queue**：主题被划分为一个或多个子主题，称为“message queues”。一个`topic`下，我们可以设置多个`queue(消息队列)`。当我们发送消息时，需要要指定该消息的`topic`。RocketMQ会轮询该`topic`下的所有队列，将消息发送出去。
  **定义：Queue是Topic在一个Broker上的分片，在分片基础上再等分为若干份（可指定份数）后的其中一份，是负载均衡过程中资源分配的基本单元。**
- 因为消息是分片的，所以很容易做到扩展

集群消费模式下一个消费者只消费该Topic中部分Queue中的消息，当一个消费者开启广播模式时则会消费该Topic下所有Queue中的消息。

先看一张有关Topic和Queue的关系图：

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190822120108606-1668559091.png)

- **Tags**

　　Tags是Topic下的次级消息类型/二级类型（注：Tags也支持`TagA || TagB`这样的表达式），可以在同一个Topic下基于Tags进行消息过滤。Tags的过滤需要经过两次比对，首先会在Broker端通过Tag hashcode进行一次比对过滤，匹配成功传到consumer端后再对具体Tags进行比对，以防止Tag hashcode重复的情况。比如交易消息又可以分为：交易创建消息，交易完成消息..... 一条消息可以没有`Tag`。RocketMQ提供2级消息分类，方便大家灵活控制。标签，换句话说，为用户提供了额外的灵活性。有了标签，来自同一个业务模块的不同目的的消息可能具有相同的主题和不同的标签。标签将有助于保持您的代码干净和连贯，并且标签还可以为RocketMQ提供的查询系统提供帮助。

Queue中具体的存储单元结构如下图，最后面的8个Byte存储Tag信息。

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190822135151149-575354679.png)

见《[RocketMQ之五：RocketMQ消息存储](https://www.cnblogs.com/duanxz/p/5020398.html)》的1.3.1、ConsumeQueue章节

- **3.1、Producer 与 Producer Group**

**`　　Producer`**表示消息队列的生产者。消息队列的本质就是实现了publish-subscribe模式，生产者生产消息，消费者消费消息。所以这里的`Producer`就是用来生产和发送消息的，一般指业务系统。RocketMQ提供了发送：普通消息（同步、异步和单向（one-way）消息）、定时消息、延时消息、事务消息。见1.2 消息类型章节

**`　 Producer Group`**是一类`Producer`的集合名称，这类`Producer`通常发送一类消息，且发送逻辑一致。相同角色的生产者被分组在一起。同一生产者组的另一个生产者实例可能被broker联系，以提交或回滚事务，以防原始生产者在交易后崩溃。

　　**警告**：考虑提供的生产者在发送消息时足够强大，每个生产者组只允许一个实例，以避免对生产者实例进行不必要的初始化。

- **4.1、Consumer 与 Consumer Group**

　　　　**Consumer:**消息消费者，一般由业务后台系统异步的消费消息。

> ```
> Push Consumer：　　Consumer 的一种，应用通常向 Consumer 对象注册一个 Listener 接口，一旦收到消息，Consumer 对象立刻回调 Listener 接口方法。Pull Consumer：　　Consumer 的一种，应用通常主动调用 Consumer 的拉消息方法从 Broker 拉消息，主动权由应用控制。
> ```

`　　　　**Consumer Group**：Consumer Group`是一类`Consumer`的集合名称，这类`Consumer`通常消费一类消息，且消费逻辑一致(使用相同 Group ID 的订阅者属于同一个集群。同一个集群下的订阅者消费逻辑必须完全一致（包括 Tag 的使用），这些订阅者在逻辑上可以认为是一个消费节点)。消费者群体是一个伟大的概念，它实现了负载平衡和容错的目标，在信息消费方面，是非常容易的。

　　　　**警告**：消费者群体的消费者实例**必须**订阅完全相同的主题。

## 1.2、组件的关系

### **1.2.1、Broker，Producer和Consumer**

如果不考虑负载均衡和高可用，最简单的Broker，Producer和Consumer之间的关系如下图所示：

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190823163201971-1030010884.png)

### **1.2.2、Topic，Topic分片和Queue**

 Queue是RocketMQ中的另一个重要概念。在对该概念进行分析介绍前，我们先来看上面的这张图：

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190823163215638-1621876516.png)

从本质上来说，RocketMQ中的Queue是数据分片的产物。为了更好地理解Queue的定义，我们还需要引入一个新的概念：Topic分片。在分布式数据库和分布式缓存领域，分片概念已经有了清晰的定义。同理，对于RocketMQ，一个Topic可以分布在各个Broker上，我们可以把一个Topic分布在一个Broker上的子集定义为一个Topic分片。对应上图，TopicA有3个Topic分片，分布在Broker1,Broker2和Broker3上，TopicB有2个Topic分片，分布在Broker1和Broker2上，TopicC有2个Topic分片，分布在Broker2和Broker3上。

**将Topic分片再切分为若干等分，其中的一份就是一个Queue**。每个Topic分片等分的Queue的数量可以不同，由用户在创建Topic时指定。

#### **queue数量指定方式：**

1、代码指定：producer.setDefaultTopicQueueNums(8);

2、配置文件指定

同时设置broker服务器的配置文件broker.properties：defaultTopicQueueNums=16

3、rocket-console控制台指定

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190823163246804-938503784.png)

我们知道，数据分片的主要目的是突破单点的资源（网络带宽，CPU，内存或文件存储）限制从而实现水平扩展。RocketMQ 在进行Topic分片以后，已经达到水平扩展的目的了，为什么还需要进一步切分为Queue呢？

这里消费者是以queue为单位进行消费的，所以为了consumer的负载，我们有了queue的概念

解答这个问题还需要从负载均衡说起。以消息消费为例，借用Rocket MQ官方文档中的Consumer负载均衡示意图来说明：

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190823163259282-39536543.png)

 



 

如图所示，TOPIC_A在一个Broker上的Topic分片有5个Queue，一个Consumer Group内有2个Consumer按照集群消费的方式消费消息，按照平均分配策略进行负载均衡得到的结果是：第一个 Consumer 消费3个Queue，第二个Consumer 消费2个Queue。如果增加Consumer，每个Consumer分配到的Queue会相应减少。Rocket MQ的负载均衡策略规定：Consumer数量应该小于等于Queue数量，如果Consumer超过Queue数量，那么多余的Consumer 将不能消费消息。

在一个Consumer Group内，Queue和Consumer之间的对应关系是多对一的关系：一个Queue最多只能分配给一个Consumer，一个Cosumer可以分配得到多个Queue。这样的分配规则，每个Queue只有一个消费者，可以避免消费过程中的多线程处理和资源锁定，有效提高各Consumer消费的并行度和处理效率。

  由此，我们可以给出Queue的定义：

  Queue是Topic在一个Broker上的分片等分为指定份数后的其中一份，是负载均衡过程中资源分配的基本单元。

 

# 二、RocketMQ发布订阅大体流程

a、producer生产者连接nameserver，产生数据放入不同的topic；

b、对于RocketMQ，一个Topic可以分布在各个Broker上，我们可以把一个Topic分布在一个Broker上的子集定义为一个Topic分片；

c、将Topic分片再切分为若干等分，其中的一份就是一个Queue。每个Topic分片等分的Queue的数量可以不同，由用户在创建Topic时指定。

d、consumer消费者连接nameserver，根据broker分配的Queue来消费数据。

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190823162946324-1549109758.png)

 

# 三、消息的种类

## 3.1、按照发送的特点分：

1. 同步消息
2. 异步消息
3. 单向消息

 

　　　　**1）同步消息（可靠同步发送）**：同步发送是指消息发送方发出数据后，会阻塞直到MQ服务方发回响应消息。**应用场景**：此种方式应用场景非常广泛，例如重要通知邮件、报名短信通知、营销短信系统等。

​        ![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190822140553910-852020229.png)

​          关键代码：`SendResult sendResult = producer.send(msg);`

　　　　**2）异步消息（可靠异步发送）**：异步发送是指发送方发出数据后，不等接收方发回响应，接着发送下个数据包的通讯方式。MQ 的异步发送，需要用户实现异步发送回调接口（SendCallback），在执行消息的异步发送时，应用不需要等待服务器响应即可直接返回，通过回调接口接收服务器响应，　　　　并对服务器的响应结果进行处理。**应用场景**：异步发送一般用于链路耗时较长，对 RT 响应时间较为敏感的业务场景，例如用户视频上传后通知启动转码服务，转码完成后通知推送转码结果等。

​        ![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190822140928450-916707856.png)

​        关键代码：`producer.sendAsync(msg, new SendCallback() {//...});`

　　　　**3）单向（one-way）消息**：单向（Oneway）发送特点为只负责发送消息，不等待服务器回应且没有回调函数触发，即只发送请求不等待应答。此方式发送消息的过程耗时非常短，一般在微秒级别。**应用场景**：适用于某些耗时非常短，但对可靠性要求并不高的场景，例如日志收集。

​         ![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190822141109130-1670428212.png)

​           单向只发送，不等待返回，所以速度最快，一般在微秒级，但可能丢失

​          关键代码：```producer.sendOneway(msg);```

### 3.2、按照使用功能特点分：

1. 普通消息（订阅）--见《[RocketMQ之二：分布式开放消息系统RocketMQ的原理与实践（消息的顺序问题、重复问题、可靠消息/事务消息）](https://www.cnblogs.com/duanxz/p/6053598.html)》
2. 顺序消息--见《[RocketMQ之二：分布式开放消息系统RocketMQ的原理与实践（消息的顺序问题、重复问题、可靠消息/事务消息）](https://www.cnblogs.com/duanxz/p/6053598.html)》
3. 广播消息
4. 延时消息
5. 批量消息
6. 事务消息

 　　　**4）定时消息**

```
// 定时消息，单位毫秒（ms），在指定时间戳（当前时间之后）进行投递，例如 2016-03-07 16:21:00 投递。如果被设置成当前时间戳之前的某个时刻，消息将立刻投递给消费者。    
long timeStamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").parse("2016-03-07 16:21:00").getTime();    
msg.setStartDeliverTime(timeStamp);    
// 发送消息，只要不抛异常就是成功    
SendResult sendResult = producer.send(msg);   
```

**`　　　　5）延时消息`**

```
Message sendMsg = new Message(topic, tags, message.getBytes());
sendMsg.setDelayTimeLevel(delayLevel);
// 默认3秒超时
SendResult sendResult = rocketMQProducer.send(sendMsg);
```

**`　　　　6）事务消息`**

　　　　RocketMQ提供类似X/Open XA的分布式事务功能来确保业务发送方和MQ消息的最终一致性，其本质是通过半消息(prepare消息和commit消息)的方式把分布式事务放在MQ端来处理。

　　　　**![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190822152402506-1526265977.jpg)**　　

 

　　　　其中：

　　　　1，发送方向消息队列 RocketMQ 服务端发送消息。

　　　　2，服务端将消息持久化成功之后，向发送方 ACK 确认消息已经发送成功，此时消息为半消息。

　　　　3，发送方开始执行本地事务逻辑。

　　　　4，发送方根据本地事务执行结果向服务端提交二次确认（Commit 或是 Rollback），服务端收到 Commit 状态则将半消息标记为可投递，订阅方最终将收到该消息；服务端收到 Rollback 状态则删除半消息，订阅方将不会接受该消息。

　　　　补偿流程：

　　　　5，在断网或者是应用重启的特殊情况下，上述步骤 4 提交的二次确认最终未到达服务端，经过固定时间后服务端将对该消息发起消息回查。

　　　　6，发送方收到消息回查后，需要检查对应消息的本地事务执行的最终结果。

　　　　7，发送方根据检查得到的本地事务的最终状态再次提交二次确认，服务端仍按照步骤 4 对半消息进行操作。

　　　　**RocketMQ的半消息机制的注意事项是**

　　　　1，根据第六步可以看出他要求发送方提供业务回查接口。

　　　　2，不能保证发送方的消息幂等，在ack没有返回的情况下，可能存在重复消息

　　　　3，消费方要做幂等处理。

　　　　**更多介绍见《[RocketMQ之二：分布式开放消息系统RocketMQ的原理与实践（消息的顺序问题、重复问题、可靠消息/事务消息）](https://www.cnblogs.com/duanxz/p/6053598.html)》单独对事务消息说明。**　

# 四、消息发布和订阅

　　在RocketMQ中，producer发布消息，consumer订阅消息。消息的收发模型如下图：

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190822153014812-635263207.png)

## 4.1、producer端，***\*所有消息发布原理图\****

***\*![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190822152848138-62473910.jpg)\****

producer完全无状态，可以集群部署。

## 4.2、consumer端，consumer有两种消息的获取模式

- 一种是Push模式，即MQServer主动向消费端推送；
- 另外一种是Pull模式，即消费端在需要时，主动到MQServer拉取。

但在具体实现时，Push和Pull模式都是采用消费端主动拉取的方式。

消费端的Push模式是通过长轮询的模式来实现的，就如同下图：

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190822160712773-1182877836.png)

　　　　　　　　Push模式示意图

　　Consumer端每隔一段时间主动向broker发送拉消息请求，broker在收到Pull请求后，如果有消息就立即返回数据，Consumer端收到返回的消息后，再回调消费者设置的Listener方法。如果broker在收到Pull请求时，消息队列里没有数据，broker端会阻塞请求直到有数据传递或超时才返回。

　　当然，Consumer端是通过一个线程将阻塞队列`LinkedBlockingQueue<PullRequest>`中的`PullRequest`发送到broker拉取消息，以防止Consumer一致被阻塞。而Broker端，在接收到Consumer的`PullRequest`时，如果发现没有消息，就会把`PullRequest`扔到ConcurrentHashMap中缓存起来。

　　broker在启动时，会启动一个线程不停的从ConcurrentHashMap取出`PullRequest`检查，直到有数据返回。

## 4.3、consumer端，**consumer有两种消息消费模式**

基本概念

消息队列 RocketMQ 是基于发布/订阅模型的消息系统。消息的订阅方订阅关注的 Topic，以获取并消费消息。由于订阅方应用一般是分布式系统，以集群方式部署有多台机器。因此消息队列 RocketMQ 约定以下概念。

**集群：**使用相同 Group ID 的订阅者属于同一个集群。同一个集群下的订阅者消费逻辑必须完全一致（包括 Tag 的使用），这些订阅者在逻辑上可以认为是一个消费节点。

**集群消费：**当使用集群消费模式时，消息队列 RocketMQ 认为任意一条消息只需要被集群内的任意一个消费者处理即可。

　　一个`Consumer Group`中的`Consumer`实例平均分摊消费消息。例如某个`Topic`有 9 条消息，其中一个`Consumer Group`有 3 个实例(可能是 3 个进程,或者 3 台机器)，那么每个实例只消费其中的 3 条消息。

**广播消费：**当使用广播消费模式时，消息队列 RocketMQ 会将每条消息推送给集群内所有注册过的客户端，保证消息至少被每台机器消费一次。

　　一条消息被多个`Consumer`消费，即使这些`Consumer`属于同一个`Consumer Group`，消息也会被`Consumer Group`中的每个`Consumer`都消费一次。在广播消费中的`Consumer Group`概念可以认为在消息划分方面无意义。　

#### 场景对比

#### **集群消费模式：**

**![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190822151103561-181472291.png)**

#### 适用场景&注意事项

- 消费端集群化部署，每条消息只需要被处理一次。
- 由于消费进度在服务端维护，可靠性更高。
- 集群消费模式下，每一条消息都只会被分发到一台机器上处理。如果需要被集群下的每一台机器都处理，请使用广播模式。
- 集群消费模式下，不保证每一次失败重投的消息路由到同一台机器上，因此处理消息时不应该做任何确定性假设。

#### **广播消费模式：**

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190822151117043-256440384.png)

适用场景&注意事项

- 广播消费模式下不支持顺序消息。
- 广播消费模式下不支持重置消费位点。
- 每条消息都需要被相同逻辑的多台机器处理。
- 消费进度在客户端维护，出现重复的概率稍大于集群模式。
- 广播模式下，消息队列 RocketMQ 保证每条消息至少被每台客户端消费一次，但是并不会对消费失败的消息进行失败重投，因此业务方需要关注消费失败的情况。
- 广播模式下，客户端每一次重启都会从最新消息消费。客户端在被停止期间发送至服务端的消息将会被自动跳过，请谨慎选择。
- 广播模式下，每条消息都会被大量的客户端重复处理，因此推荐尽可能使用集群模式。
- 目前仅 Java 客户端支持广播模式。
- 广播模式下服务端不维护消费进度，所以消息队列 RocketMQ 控制台不支持消息堆积查询、消息堆积报警和订阅关系查询功能。

#### **使用集群模式模拟广播：**

如果业务需要使用广播模式，也可以创建多个 Group ID，用于订阅同一个 Topic。

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190822151126411-914702734.png)

适用场景&注意事项

- 每条消息都需要被多台机器处理，每台机器的逻辑可以相同也可以不一样。
- 消费进度在服务端维护，可靠性高于广播模式。
- 对于一个 Group ID 来说，可以部署一个消费端实例，也可以部署多个消费端实例。 当部署多个消费端实例时，实例之间又组成了集群模式（共同分担消费消息）。 假设 Group ID 1 部署了三个消费者实例 C1、C2、C3，那么这三个实例将共同分担服务器发送给 Group ID 1 的消息。 同时，实例之间订阅关系必须保持一致。

# 五、负载均衡（发送端）

**生产端负载均衡**，看下图：

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190822161117835-1771617328.png)

*producer发送消息负载均衡图*

首先分析一下RocketMQ的客户端发送消息的源码：

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190822161154277-236395467.png)

在整个应用生命周期内，生产者需要调用一次start方法来初始化，初始化主要完成的任务有：

> 1. 如果没有指定namesrv地址，将会自动寻址
> 2. 启动定时任务：更新namesrv地址、从namsrv更新topic路由信息、清理已经挂掉的broker、向所有broker发送心跳...
> 3. 启动负载均衡的服务

初始化完成后，开始发送消息，发送消息的主要代码如下：

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190822161224119-940044313.png)

代码中需要关注的两个方法tryToFindTopicPublishInfo和selectOneMessageQueue。前面说过在producer初始化时，会启动定时任务获取路由信息并更新到本地缓存，所以tryToFindTopicPublishInfo会首先从缓存中获取topic路由信息，如果没有获取到，则会自己去namesrv获取路由信息。selectOneMessageQueue方法通过轮询的方式，返回一个队列，以达到负载均衡的目的。

如果Producer发送消息失败，会自动重试，重试的策略：

1. 重试次数 < retryTimesWhenSendFailed（可配置）
2. 总的耗时（包含重试n次的耗时） < sendMsgTimeout（发送消息时传入的参数）
3. 同时满足上面两个条件后，Producer会选择另外一个队列发送消息

 

**消费端的负载均衡**，先看下图：

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190822160257362-840220550.png)

`Producer`向一些队列轮流发送消息，队列集合称为`Topic`，`Consumer`如果做广播消费，则一个`consumer`实例消费这个`Topic`对应的所有队列；如果做集群消费，则多个`Consumer`实例平均消费这个`Topic`对应的队列集合。

上图的集群模式里，每个consumer消费部分消息，这里的负载均衡是怎样的呢？

消费端会通过RebalanceService线程，20秒钟做一次基于topic下的所有队列负载：

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190823181539820-2028491031.png)

1. 遍历Consumer下的所有topic，然后根据topic订阅所有的消息
2. 获取同一topic和Consumer Group下的所有Consumer
3. 然后根据具体的分配策略来分配消费队列，分配的策略包含：平均分配、消费端配置等

如同上图所示：如果有 3 个队列，2 个 consumer，那么第一个 Consumer 消费 2 个队列，第二 consumer 消费 1 个队列。这里采用的就是平均分配策略，它类似于我们的分页，TOPIC下面的所有queue就是记录，Consumer的个数就相当于总的页数，那么每页有多少条记录，就类似于某个Consumer会消费哪些队列。

通过这样的策略来达到大体上的平均消费，这样的设计也可以很方面的水平扩展Consumer来提高消费能力。

 

# 六、总结

#### 1、异步复制和同步双写总结

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190829171753417-1928324652.png)

 

2、集群方式对比

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190829171812054-1125543560.png)

 

 3、高可用演练场景

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190829171827851-65852714.png)

 

