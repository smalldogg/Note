# [RocketMQ之七：RocketMQ管理与监控]

# 前言

首先提出我们的监控诉求，出现如下情况时，希望能够及时接收到系统告警通知：

-   RocketMQ 服务宕机
-   RocketMQ 消费者下线
-   RocketMQ 消息出现长时间或者大量堆积

本文将通过修改 rocketmq-console源码的方式，增加RocketMQ 消费者下线 和RocketMQ 消息出现长时间或者大量堆积监控能力。

# 一. RocketMQ 服务宕机监控告警

这一级别的监控，本质上而言是监控Linux上启动的Rocket MQ Java进程的运行情况。细分的话，需要监控以下两个维度：

1.   Linux Java 进程的CPU 使用率，内存使用量；
2.   Java 进程本身的JVM的服务质量，GC，并发数，内存分布等

  一般的公司在运维方面会有专门的监控组件，如zabbix会做统一处理。

例如这里用简单的shell脚本+钉钉组装的最简单的监控告警方式：
监控的方式有很多，比如简单点的，我们可以写一个shell脚本，监控执行rocketmqJava进程的存活状态，如果rocketmq crash了，发送告警：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
#!/bin/bash
## monitor.sh
while true
do
        echo "开始监控rocket broker 进程..."
        PID=$(ps -ef | grep java | grep org.apache.rocketmq.broker.BrokerStartup | awk '{printf $2}');
        if [ -z $PID  ];then
                curl 'https://oapi.dingtalk.com/robot/send?access_token=xxxxxx' -H 'Content-Type: application/json'  \
                 -d '
                 {"msgtype": "text",
                         "text": {
                                 "content": "【172.xxx.xxx.xxx】rocketmq broker 进程不存在，可能宕机，请尽快排查！"
                         }
                 }'
        fi
        sleep 10
done
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

效果：

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190823114616791-1409764553.png)

 

# 2. RocketMQ的集群组件组成

## 2.1、rocketmq-console web控制台介绍

 官方提供了一个WEB项目，可以查看rocketmq数据和执行一些操作。incubator-rocketmq-externals，这个项目中有一个子模块叫“rocketmq-console”，这个便是管理控制台项目。Rocket-console做为rocketmq社区维护的产品需要从GitHub上下载，下载地址：https://github.com/apache/rocketmq-externals
![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190823152607742-2083479115.png)

各个功能的介绍：

- OPS运维：NameServer的地址的管理
- Dashboard驾驶舱：展示Broker和Topic的柱状图和折线图。
- Cluster集群：Brokder的集群部署情况及每个Broker的详情。
- Topic主题：对Topic的新增、修改、删除。对consumer管理、消息位点重置等。
- Consumer消费者：consumer的一些状态的管理。
- Producer生产者：producer的信息(ip、port、版本等)查看。
- Message消息：根据Topic、Message Key、Message ID三项对Message分组查询。一般第一项根据Topic查询比较多。因为据说根据key去查询有坑。建议id和Topic，id因为唯一最简单。
- MessageTrace消息轨迹：

 

一个完整的RocketMQ集群，一般组成关系如下图所示：

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190823114825079-172533107.png)


除了核心组成部分：Name Server 和 Broker Cluster 之外，RocketMQ还提供了 mqadmin工具，该工具的具体实现代码在RocketMQ tools模块(rocketmq-tools-xxxx.jar)中。但是mqadmin命令行在交互上不够友好，**rocketmq-console**作为一个社区项目，底层基于mqadmin 核心库，用Spring Boot+Angularjs实现了一个RocketMQ Web管理端,开发运维人员可以轻松地使用此管理端完成日常运维操作。

# 3. mqadmin–提供一套命令行工具，做RocketMQ的日常管理维护

## 3.1、mqadmin 工具在哪儿？

mqadmin本质上是一个Java命令行工具，也就是说执行mqadmin的过程也是执行Java的过程，**mqadmin**的位置和runbroker和mqnamesrv并列：

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190823135026736-478701441.png)

## 3.2、mqadmin能做什么？

执行./mqadmin，会在命令行输出其指令列表：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
[root@localhost bin]# ./mqadmin
The most commonly used mqadmin commands are:
   updateTopic          Update or create topic
   deleteTopic          Delete topic from broker and NameServer.
   updateSubGroup       Update or create subscription group
   deleteSubGroup       Delete subscription group from broker.
   updateBrokerConfig   Update broker's config
   updateTopicPerm      Update topic perm
   topicRoute           Examine topic route info
   topicStatus          Examine topic Status info
   topicClusterList     get cluster info for topic
   brokerStatus         Fetch broker runtime status data
   queryMsgById         Query Message by Id
   queryMsgByKey        Query Message by Key
   queryMsgByUniqueKey  Query Message by Unique key
   queryMsgByOffset     Query Message by offset
   printMsg             Print Message Detail
   printMsgByQueue      Print Message Detail
   sendMsgStatus        send msg to broker.
   brokerConsumeStats   Fetch broker consume stats data
   producerConnection   Query producer's socket connection and client version
   consumerConnection   Query consumer's socket connection, client version and subscription
   consumerProgress     Query consumers's progress, speed
   consumerStatus       Query consumer's internal data structure
   cloneGroupOffset     clone offset from other group.
   clusterList          List all of clusters
   topicList            Fetch all topic list from name server
   updateKvConfig       Create or update KV config.
   deleteKvConfig       Delete KV config.
   wipeWritePerm        Wipe write perm of broker in all name server
   resetOffsetByTime    Reset consumer offset by timestamp(without client restart).
   updateOrderConf      Create or update or delete order conf
   cleanExpiredCQ       Clean expired ConsumeQueue on broker.
   cleanUnusedTopic     Clean unused topic on broker.
   startMonitoring      Start Monitoring
   statsAll             Topic and Consumer tps stats
   allocateMQ           Allocate MQ
   checkMsgSendRT       check message send response time
   clusterRT            List All clusters Message Send RT
   getNamesrvConfig     Get configs of name server.
   updateNamesrvConfig  Update configs of name server.
   getBrokerConfig      Get broker config by cluster or special broker!
   queryCq              Query cq command.
   sendMessage          Send a message
   consumeMessage       Consume message
   updateAclConfig      Update acl config yaml file in broker
   deleteAccessConfig   Delete Acl Config Account in broker
   clusterAclConfigVersion List all of acl config version information in cluster
   updateGlobalWhiteAddr Update global white address for acl Config File in broker

See 'mqadmin help <command>' for more information on a specific command.
[root@localhost bin]# 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

例如：

通过命令行查询消息堆压：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
[root@localhost rocketmq-4.5.2]# sh bin/mqadmin consumerProgress -n 10.200.110.46:9876
RocketMQLog:WARN No appenders could be found for logger (io.netty.util.internal.PlatformDependent0).
RocketMQLog:WARN Please initialize the logger system properly.
#Group                            #Count  #Version                 #Type  #Model          #TPS     #Diff Total
please_rename_unique_group_name_  0       OFFLINE                                         0        0
configurationfile-encryption-dem  0       OFFLINE                                         0        0
groupnamedef                      1       V4_5_2                   PUSH   CLUSTERING      0        0
TOOLS_CONSUMER                    0       OFFLINE                                         0        0
[root@localhost rocketmq-4.5.2]# ls
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

具体每个指令的作用不是本文的重点，后续会开新的文章介绍~

# 4、使用 rocketmq-console添加MQ监控告警

我们可以利用rocketmq-console做如下的监控：

-   RocketMQ 消费者下线
-   RocketMQ 消息出现长时间或者大量堆积

## 4.1 rocketmq-console的监控告警功能

　　作为mqadmin的GUI封装，rocketmq-console基本上具备了mqadmin的功能外，也提供了一些额外的功能，如dashboard面板统计。但是，作为开源源码部分，rocketmq-console将MQ监控功能做了隐藏，我们需要手动放开。如下是使用rocket-console的监控原理：
![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190823135639352-843503278.png)
当此项功能被放开后，在Consumer菜单下，为每一个consumer-group 的operation 会增加MONITOR CONFIG 选项，如下图所示：
![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190823135810894-760033195.png)
指标名称    说明    备注
minCount    当前消费分组的机器数量最小阈值，低于此值将会告警    
minCount    当前消费分组允许的最大消息堆积量，高于辞职将会告警    

## 4.2 如何开启rocketmq-console的监控告警功能

开源的rocketmq-console将此功能隐藏了，可以通过下载源码，并修改源码的方式支持。

### 4.2.1 下载源码

从github中获取源码，rocketmq-externals
地址：https://github.com/apache/rocketmq-externals

### 4.2.2 导入项目

项目导入后，如下图所示，rocketmq-console即为控制台代码
![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190823135959371-1052622060.png)

 

### 4.2.3 放开console控制台的监控参数配置

默认的rocketmq-console将此功能注释掉了，修改文件:
~/rocketmq-console/src/resources/static/view/pages/consumer.html,将如下图所示的代码放开即可。
![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190823140018975-254158920.png)

### 4.2.4 开启定时任务监控，扫描实时数据，做阈值判断，告警提示

默认情况下，rocketmq-console只定义了定时任务入口，具体的策略没有任何处理，我们需要根据自己的需求加入自身的告警方式，比如：邮箱，钉钉，短信，微信等等。
其预留的定时任务实现类为：
org.apache.rocketmq.console.task.MonitorTask
定时任务的扫描频率可根据自身系统要求考量设置。



[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Component
public class MonitorTask {
    private Logger logger = LoggerFactory.getLogger(MonitorTask.class);

    @Resource
    private MonitorService monitorService;

    @Resource
    private ConsumerService consumerService;

//    @Scheduled(cron = "* * * * * ?")
//    定时任务的扫描频率可根据自身系统要求考量设置
    public void scanProblemConsumeGroup() {
        for (Map.Entry<String, ConsumerMonitorConfig> configEntry : monitorService.queryConsumerMonitorConfig().entrySet()) {
            GroupConsumeInfo consumeInfo = consumerService.queryGroup(configEntry.getKey());
            if (consumeInfo.getCount() < configEntry.getValue().getMinCount() || consumeInfo.getDiffTotal() > configEntry.getValue().getMaxDiffTotal()) {
                logger.info("op=look consumeInfo {}", JsonUtil.obj2String(consumeInfo));
               // notify the alert system
               //根据自身的要求加如通知方式
            }
        }
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

###  4.2.5 build修改后的代码，生成可运行的jar包，然后部署运行

![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190823140153581-468894516.png)

 

###  4.2.6 样例



经笔者改造后的console的控制台可以显示出 ‘MONITOR CONFIG’ 配置项：
![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190823140204983-63974786.png)

 


钉钉告警样例：
![img](https://img2018.cnblogs.com/blog/285763/201908/285763-20190823140215584-1265811442.png)

 

# 5. 总结


rocketmq-console 作为开发运维人员监控MQ的便捷入口，可根据自身要求改造rocketmq-console,rocketmq-console服务本身可以调用所有mqadmin的所有能力，项目本身基于angularjs +spring boot，作为java 开发人员来说拓展成本也比较低。不过前期需要对rocketmq的一些概念和各种衡量标准要有明确的认知。

