## zab协议

### 一种支持奔溃恢复的原子消息广播协议（两种模式）

恢复模式 

当服务启动或者在领导者崩溃后，Zab就进入了恢复模式，当领导者被选举出 来，且大多数 server 完成了和 leader 的状态同步以后，恢复模式就结束了。状 态同步保证了 leader 和 server 具有相同的系统状态。 

广播模式 

一旦 leader 已经和多数的 follower 进行了状态同步后，它就可以开始广播消息 了，即进入广播状态。这时候当一个 server 加入 ZooKeeper 服务中，它会在 恢复模式下启动，发现 leader，并和 leader 进行状态同步。待到同步结束，它也参与消息广播。ZooKeeper 服务一直维持在 Broadcast 状态，直到 leader  崩溃了或者 leader 失去了大部分的 followers 支持。 