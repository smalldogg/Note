# [RocketMQ之二：分布式开放消息系统RocketMQ的原理与实践（消息的顺序问题、重复问题、可靠消息/事务消息）

分布式消息系统作为实现分布式系统可扩展、可伸缩性的关键组件，需要具有高吞吐量、高可用等特点。而谈到消息系统的设计，就回避不了三个问题：

> 1. 消息的顺序问题
> 2. 消息的重复问题
> 3. 消息的可靠性

RocketMQ作为阿里开源的一款高性能、高吞吐量的消息中间件，它是怎样来解决这两个问题的？RocketMQ 有哪些关键特性？其实现原理是怎样的？

## 关键特性以及其实现原理

## 消息顺序（Message Order）

在使用`DefaultMQPushConsumer`时，您需要决定使用排序的还是并行的消息。

- 排序的（Orderly）
  消费消息的有序意味着，消息的使用顺序与生产者为每个消息队列发送的顺序相同。如果您正在处理全局顺序是强制性的场景，请确保您使用的主题只有一个消息队列。
  **警告**：如果指定消费有序，则消息消耗的最大并发性是消费者组订阅的消息队列的数量。
- 并行的（Concurrently）
  并行地使用消息时，消费消息的最大并行度只受每个客户端指定的线程池大小限制。
  **警告**：此模式不再保证消息顺序。

### 一、顺序消息

消息有序指的是一类消息消费时，能按照发送的顺序来消费。例如：一个订单产生了 3 条消息，分别是订单创建、订单付款、订单完成。消费时，要按照这个顺序消费才有意义。但同时订单之间又是可以并行消费的。

假如生产者产生了2条消息：M1、M2，要保证这两条消息的顺序，应该怎样做？你脑中想到的可能是这样：

![img](https://images2017.cnblogs.com/blog/285763/201712/285763-20171208114323437-1412195033.png)

你可能会采用这种方式保证消息顺序


M1发送到S1后，M2发送到S2，如果要保证M1先于M2被消费，那么需要M1到达消费端后，通知S2，然后S2再将M2发送到消费端。

这个模型存在的问题是，如果M1和M2分别发送到两台Server上，就不能保证M1先达到，也就不能保证M1被先消费，那么就需要在MQ Server集群维护消息的顺序。那么如何解决？一种简单的方式就是将M1、M2发送到同一个Server上：

![img](https://images2017.cnblogs.com/blog/285763/201712/285763-20171208114339749-1756555858.png)

保证消息顺序，你改进后的方法


这样可以保证M1先于M2到达MQServer（客户端等待M1成功后再发送M2），根据先达到先被消费的原则，M1会先于M2被消费，这样就保证了消息的顺序。

这个模型，理论上可以保证消息的顺序，但在实际运用中你应该会遇到下面的问题：

![img](https://images2017.cnblogs.com/blog/285763/201712/285763-20171208114354734-2068603988.png)

网络延迟问题

只要将消息从一台服务器发往另一台服务器，就会存在网络延迟问题。如上图所示，如果发送M1耗时大于发送M2的耗时，那么M2就先被消费，仍然不能保证消息的顺序。即使M1和M2同时到达消费端，由于不清楚消费端1和消费端2的负载情况，仍然有可能出现M2先于M1被消费。如何解决这个问题？将M1和M2发往同一个消费者即可，且发送M1后，需要消费端响应成功后才能发送M2。

但又会引入另外一个问题，如果发送M1后，消费端1没有响应，那是继续发送M2呢，还是重新发送M1？一般为了保证消息一定被消费，肯定会选择重发M1到另外一个消费端2，就如下图所示。

![img](https://images2017.cnblogs.com/blog/285763/201712/285763-20171208114413218-1682599385.png)

保证消息顺序的正确姿势

这样的模型就严格保证消息的顺序，细心的你仍然会发现问题，消费端1没有响应Server时有两种情况，一种是M1确实没有到达，另外一种情况是消费端1已经响应，但是Server端没有收到。如果是第二种情况，重发M1，就会造成M1被重复消费。也就是我们后面要说的第二个问题，消息重复问题。

回过头来看消息顺序问题，严格的顺序消息非常容易理解，而且处理问题也比较容易，要实现严格的顺序消息，简单且可行的办法就是：

> 保证`生产者 - MQServer - 消费者`是一对一对一的关系

但是这样设计，并行度就成为了消息系统的瓶颈（吞吐量不够），也会导致更多的异常处理，比如：只要消费端出现问题，就会导致整个处理流程阻塞，我们不得不花费更多的精力来解决阻塞的问题。

但我们的最终目标是要集群的高容错性和高吞吐量。这似乎是一对不可调和的矛盾，那么阿里是如何解决的？

> 世界上解决一个计算机问题最简单的方法：“恰好”不需要解决它！—— [沈询](http://i.youku.com/u/UMTcwMTg3NDc1Mg==?from=113-2-1-2)

有些问题，看起来很重要，但实际上我们可以通过合理的设计或者将问题分解来规避。如果硬要把时间花在解决它们身上，实际上是浪费的，效率低下的。从这个角度来看消息的顺序问题，我们可以得出两个结论：

> 1、不关注乱序的应用实际大量存在
> 2、队列无序并不意味着消息无序

最后我们从源码角度分析RocketMQ怎么实现发送顺序消息。

一般消息是通过轮询所有队列来发送的（负载均衡策略），顺序消息可以根据业务，比如说订单号相同的消息发送到同一个队列。下面的示例中，OrderId相同的消息，会发送到同一个队列：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// RocketMQ默认提供了两种MessageQueueSelector实现：随机/Hash
SendResult sendResult = producer.send(msg, new MessageQueueSelector() {
    @Override
    public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
        Integer id = (Integer) arg;
        int index = id % mqs.size();
        return mqs.get(index);
    }
}, orderId);
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

在获取到路由信息以后，会根据`MessageQueueSelector`实现的算法来选择一个队列，同一个OrderId获取到的队列是同一个队列。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private SendResult send()  {
    // 获取topic路由信息
    TopicPublishInfo topicPublishInfo = this.tryToFindTopicPublishInfo(msg.getTopic());
    if (topicPublishInfo != null && topicPublishInfo.ok()) {
        MessageQueue mq = null;
        // 根据我们的算法，选择一个发送队列
        // 这里的arg = orderId
        mq = selector.select(topicPublishInfo.getMessageQueueList(), msg, arg);
        if (mq != null) {
            return this.sendKernelImpl(msg, mq, communicationMode, sendCallback, timeout);
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### 二、消息重复

上面在解决消息顺序问题时，引入了一个新的问题，就是消息重复。那么RocketMQ是怎样解决消息重复的问题呢？还是“恰好”不解决。

造成消息的重复的根本原因是：网络不可达。只要通过网络交换数据，就无法避免这个问题。所以解决这个问题的办法就是不解决，转而绕过这个问题。那么问题就变成了：如果消费端收到两条一样的消息，应该怎样处理？

> 1、消费端处理消息的业务逻辑保持幂等性
> 2、保证每条消息都有唯一编号且保证消息处理成功与去重表的日志同时出现

第1条很好理解，只要保持幂等性，不管来多少条重复消息，最后处理的结果都一样。第2条原理就是利用一张日志表来记录已经处理成功的消息的ID，如果新到的消息ID已经在日志表中，那么就不再处理这条消息。

我们可以看到第1条的解决方式，很明显应该在消费端实现，不属于消息系统要实现的功能。第2条可以消息系统实现，也可以业务端实现。正常情况下出现重复消息的概率不一定大，且由消息系统实现的话，肯定会对消息系统的吞吐量和高可用有影响，所以最好还是由业务端自己处理消息重复的问题，这也是RocketMQ不解决消息重复的问题的原因。

**RocketMQ不保证消息不重复，如果你的业务需要保证严格的不重复消息，需要你自己在业务端去重。**

### 三、事务消息

RocketMQ除了支持普通消息，顺序消息，另外还支持事务消息。首先讨论一下什么是事务消息以及支持事务消息的必要性。我们以一个转帐的场景为例来说明这个问题：Bob向Smith转账100块。

在单机环境下，执行事务的情况，大概是下面这个样子：

![img](https://images2017.cnblogs.com/blog/285763/201712/285763-20171208114621218-1969957853.png)

单机环境下转账事务示意图

当用户增长到一定程度，Bob和Smith的账户及余额信息已经不在同一台服务器上了，那么上面的流程就变成了这样：

![img](https://images2017.cnblogs.com/blog/285763/201712/285763-20171208114633406-57080271.png)

集群环境下转账事务示意图

这时候你会发现，同样是一个转账的业务，在集群环境下，耗时居然成倍的增长，这显然是不能够接受的。那我们如何来规避这个问题？

> **大事务 = 小事务 + 异步**

将大事务拆分成多个小事务异步执行。这样基本上能够将跨机事务的执行效率优化到与单机一致。转账的事务就可以分解成如下两个小事务：

![img](https://images2017.cnblogs.com/blog/285763/201712/285763-20171208114647406-948314576.png)

小事务+异步消息


图中执行本地事务（Bob账户扣款）和发送异步消息应该保持同时成功或者失败中，也就是扣款成功了，发送消息一定要成功，如果扣款失败了，就不能再发送消息。那问题是：我们是先扣款还是先发送消息呢？

首先我们看下，先发送消息，大致的示意图如下：

![img](https://images2017.cnblogs.com/blog/285763/201712/285763-20171208114701671-860706988.png)

事务消息：先发送消息

存在的问题是：如果消息发送成功，但是扣款失败，消费端就会消费此消息，进而向Smith账户加钱。

先发消息不行，那我们就先扣款呗，大致的示意图如下：

![img](https://images2017.cnblogs.com/blog/285763/201712/285763-20171208114716999-1681728483.png)

事务消息-先扣款

存在的问题跟上面类似：如果扣款成功，发送消息失败，就会出现Bob扣钱了，但是Smith账户未加钱。

可能大家会有很多的方法来解决这个问题，比如：直接将发消息放到Bob扣款的事务中去，如果发送失败，抛出异常，事务回滚。这样的处理方式也符合“恰好”不需要解决的原则。RocketMQ支持事务消息，下面我们来看看RocketMQ是怎样来实现的。

![img](https://images2017.cnblogs.com/blog/285763/201712/285763-20171208114731890-1856906407.png)

RocketMQ实现发送事务消息

RocketMQ第一阶段发送`Prepared消息`时，会拿到消息的地址，第二阶段执行本地事物，第三阶段通过第一阶段拿到的地址去访问消息，并修改状态。细心的你可能又发现问题了，如果确认消息发送失败了怎么办？RocketMQ会定期扫描消息集群中的事物消息，这时候发现了`Prepared消息`，它会向消息发送者确认，Bob的钱到底是减了还是没减呢？如果减了是回滚还是继续发送确认消息呢？RocketMQ会根据发送端设置的策略来决定是回滚还是继续发送确认消息。这样就保证了消息发送与本地事务同时成功或同时失败。

那我们来看下RocketMQ源码，是不是这样来处理事务消息的。客户端发送事务消息的部分（完整代码请查看：`rocketmq-example`工程下的`com.alibaba.rocketmq.example.transaction.TransactionProducer`）：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// 未决事务，MQ服务器回查客户端
// 也就是上文所说的，当RocketMQ发现`Prepared消息`时，会根据这个Listener实现的策略来决断事务
TransactionCheckListener transactionCheckListener = new TransactionCheckListenerImpl();
// 构造事务消息的生产者
TransactionMQProducer producer = new TransactionMQProducer("groupName");
// 设置事务决断处理类
producer.setTransactionCheckListener(transactionCheckListener);
// 本地事务的处理逻辑，相当于示例中检查Bob账户并扣钱的逻辑
TransactionExecuterImpl tranExecuter = new TransactionExecuterImpl();
producer.start()
// 构造MSG，省略构造参数
Message msg = new Message(......);
// 发送消息
SendResult sendResult = producer.sendMessageInTransaction(msg, tranExecuter, null);
producer.shutdown();
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

接着查看`sendMessageInTransaction`方法的源码，总共分为3个阶段：发送`Prepared消息`、执行本地事务、发送确认消息。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public TransactionSendResult sendMessageInTransaction(.....)  {
    // 逻辑代码，非实际代码
    // 1.发送消息
    sendResult = this.send(msg);
    // sendResult.getSendStatus() == SEND_OK
    // 2.如果消息发送成功，处理与消息关联的本地事务单元
    LocalTransactionState localTransactionState = tranExecuter.executeLocalTransactionBranch(msg, arg);
    // 3.结束事务
    this.endTransaction(sendResult, localTransactionState, localException);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

`endTransaction`方法会将请求发往`broker(mq server)`去更新事物消息的最终状态：

1. 根据`sendResult`找到`Prepared消息`
2. 根据`localTransaction`更新消息的最终状态

如果`endTransaction`方法执行失败，导致数据没有发送到`broker`，`broker`会有回查线程定时（默认1分钟）扫描每个存储事务状态的表格文件，如果是已经提交或者回滚的消息直接跳过，如果是`prepared状态`则会向`Producer`发起`CheckTransaction`请求，`Producer`会调用`DefaultMQProducerImpl.checkTransactionState()`方法来处理`broker`的定时回调请求，而`checkTransactionState`会调用我们的事务设置的决断方法，最后调用`endTransactionOneway`让`broker`来更新消息的最终状态。

再回到转账的例子，如果Bob的账户的余额已经减少，且消息已经发送成功，Smith端开始消费这条消息，这个时候就会出现消费失败和消费超时两个问题？解决超时问题的思路就是一直重试，直到消费端消费消息成功，整个过程中有可能会出现消息重复的问题，按照前面的思路解决即可。

![img](https://images2017.cnblogs.com/blog/285763/201712/285763-20171208114749093-1332007218.png)

消费事务消息

这样基本上可以解决超时问题，但是如果消费失败怎么办？阿里提供给我们的解决方法是：**人工解决**。大家可以考虑一下，按照事务的流程，因为某种原因Smith加款失败，需要回滚整个流程。如果消息系统要实现这个回滚流程的话，系统复杂度将大大提升，且很容易出现Bug，估计出现Bug的概率会比消费失败的概率大很多。我们需要衡量是否值得花这么大的代价来解决这样一个出现概率非常小的问题，这也是大家在解决疑难问题时需要多多思考的地方。

例子：https://www.jianshu.com/p/cc5c10221aa1

 

### 四、RocketMQ最佳实践

##### 一、Producer最佳实践

1. 一个应用尽可能用一个 Topic，消息子类型用 tags 来标识，tags 可以由应用自由设置。只有发送消息设置了tags，消费方在订阅消息时，才可以利用 tags 在 broker 做消息过滤。
2. 每个消息在业务层面的唯一标识码，要设置到 keys 字段，方便将来定位消息丢失问题。由于是哈希索引，请务必保证 key 尽可能唯一，这样可以避免潜在的哈希冲突。
3. 消息发送成功或者失败，要打印消息日志，**务必要打印 sendresult 和 key 字段**。
4. 对于消息不可丢失应用，务必要有消息重发机制。例如：消息发送失败，存储到数据库，能有定时程序尝试重发或者人工触发重发。
5. 某些应用如果不关注消息是否发送成功，请直接使用`sendOneWay`方法发送消息。

##### 二、Consumer最佳实践

1. 尽量使用批量方式消费方式，可以很大程度上提高消费吞吐量。
2. 优化每条消息消费过程
3. Consumer 数量要小于等于queue的总数量，由于Topic下的queue会被相对均匀的分配给Consumer，如果 Consumer 超过queue的数量，那多余的 Consumer 将没有queue可以消费消息。
4. 消费过程要做到幂等（即消费端去重），RocketMQ为了保证性能并不支持严格的消息去重。
5. 尽量使用批量方式消费，RocketMQ消费端采用pull方式拉取消息，通过consumeMessageBatchMaxSize参数可以增加单次拉取的消息数量，可以很大程度上提高消费吞吐量。另外，提高消费并行度也可以通过增加Consumer处理线程的方式，对应参数consumeThreadMin和consumeThreadMax。
6. 消息发送成功或者失败，要打印消息日志。

##### 三、其他配置

线上应该关闭`autoCreateTopicEnable`，即在配置文件中将其设置为`false`。

RocketMQ在发送消息时，会首先获取路由信息。如果是新的消息，由于MQServer上面还没有创建对应的`Topic`，这个时候，如果上面的配置打开的话，会返回默认TOPIC的（RocketMQ会在每台`broker`上面创建名为`TBW102`的TOPIC）路由信息，然后`Producer`会选择一台`Broker`发送消息，选中的`broker`在存储消息时，发现消息的`topic`还没有创建，就会自动创建`topic`。后果就是：以后所有该TOPIC的消息，都将发送到这台`broker`上，达不到负载均衡的目的。

所以基于目前RocketMQ的设计，建议关闭自动创建TOPIC的功能，然后根据消息量的大小，手动创建TOPIC。

#### RocketMQ设计相关

RocketMQ的设计假定：

> 每台PC机器都可能宕机不可服务
> 任意集群都有可能处理能力不足
> 最坏的情况一定会发生
> 内网环境需要低延迟来提供最佳用户体验

RocketMQ的关键设计：

> 分布式集群化
> 强数据安全
> 海量数据堆积
> 毫秒级投递延迟（推拉模式）

这是RocketMQ在设计时的假定前提以及需要到达的效果。我想这些假定适用于所有的系统设计。随着我们系统的服务的增多，每位开发者都要注意自己的程序是否存在单点故障，如果挂了应该怎么恢复、能不能很好的水平扩展、对外的接口是否足够高效、自己管理的数据是否足够安全...... 多多规范自己的设计，才能开发出高效健壮的程序。



