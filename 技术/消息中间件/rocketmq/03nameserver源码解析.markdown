# RocketMQ之三：nameserver源码解析

## NameServer

namesrv主要的工作是维持心跳，不保证数据的一致性，因为namesrv是无状态的

数据一致性由broker来维持

### 对比zk





# NameServer代码结构

如下图：

![img](https://upload-images.jianshu.io/upload_images/10845135-1841f82c178d0292.png)

代码结构

如上图所示，nameServer设计比较轻量级的，其中几个主要类的功能为：

NamesrvStartup:从命名就可以看出这为NameServer的启动类。

RouteInfoManager:从类名可以看出为路由信息的管理类，就是存放Broker的状态信息及Topic于Broker的关联关系。如下截图：

![img](https://upload-images.jianshu.io/upload_images/10845135-8a14f671640ee54d.png)

DefaultRequestProcessor:类名可看出为处理请求的类,里面封装了对broker发来的各种请求的响应。

NamesrvController:NameServer控制类，管控NameServer的启动、初始化、停止等生命周期。

先看下NamesrvStartup的代码，简要的抽取几点：

final NamesrvConfig namesrvConfig = new NamesrvConfig();

final NettyServerConfig nettyServerConfig = new NettyServerConfig();

nettyServerConfig.setListenPort(9876);

。。。。。。。。。。。。。

final NamesrvController controller = new NamesrvController(namesrvConfig, nettyServerConfig);

controller.getConfiguration().registerConfig(properties);

boolean initResult = controller.initialize();

可以看出NameServer启动时候，初始化了nameServer和netty的配置（可以看出RocketMQ各部分之间的通信使用的netty）,并初始化了NameserController。

看下initialize（）方法

![img](https://upload-images.jianshu.io/upload_images/10845135-5b13309569399436.png)

可以看出初始化了nettyServer，初始化了线程池，注册了请求处理类。

 

![img](https://upload-images.jianshu.io/upload_images/10845135-8ff5ef75c06b7650.png)

我们先看下broker的注册，在DefaultRequestProcessor类中

 

![img](https://upload-images.jianshu.io/upload_images/10845135-318e813ad6dbe08f.png)

监听到register请求后，调用注册请求方法

![img](https://upload-images.jianshu.io/upload_images/10845135-240832617019fcc2.png)

上图可以看出为了避免线程安全问题，使用了lock锁。注册过程就是维护RouteInfoManager（管理消息路由）中的数据结构。

注册broker成功后，就是通过心跳来检测broker的生命周期，维护broker状态信息。

从boolean initResult = controller.initialize();方法中可以看到这一块代码：

![img](https://upload-images.jianshu.io/upload_images/10845135-947b7687949dc9bf.png)

![img](https://upload-images.jianshu.io/upload_images/10845135-87ebb3c114420298.png)

![img](https://upload-images.jianshu.io/upload_images/10845135-9d64a2ce57f44db4.png)

![img](https://upload-images.jianshu.io/upload_images/10845135-48ec32c477fc42d7.png)

图一为初始化调度线程池（单线程），图二初始化后启动调度线程，图三可以看出，调度线程定时的扫描brokerLiveTable里面broker的状态信息，发现最近的更新时间与当前时间相差大于BROKER_CHANNEL_EXPIRED_TIME =1000 *60 *2，即两分钟的话，就删除当前broker，从图四可以看出，删除broker就是删除RouteInfoManager中维护的几个hashmap中关于broker的信息

