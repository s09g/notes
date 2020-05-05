## Gossip Protocol

**Gossip协议**，又称**流行病协议**。<br>
假设我们只有一条消息和一个`sender`*此协议同样适用于多`sender`多消息的情景*。

1. `sender`定期(默认5s)向随机挑选的`b`个`receiver`发送消息(`gossip message`)。
   + `gossip message`将通过`UDP`传输
   + `b`称为`gossip fan-out`，默认为2
   + 同一个`receiver`可以被反复选中

![](https://github.com/s09g/notes/raw/master/cloud/12.Gossip%20Protocol/assets/1.png)

1. 一旦`receiver`收到消息，`receiver`被称为**gossip感染**
   + 被感染的`receiver`会开始跟`sender`一样的行为，每5s随机挑选`b`个`receiver`发送`gossip`
   + 某些`receiver`可能被不同的`sender`同时挑中，反手接收同样的`gossip` *但是带来的网络开销并不大*
![](https://github.com/s09g/notes/raw/master/cloud/12.Gossip%20Protocol/assets/2.png)

#### Push 模型 vs. Pull 模型

上面描述的协议被称为**Push-based Gossip**:
+ 一旦收到组播消息，节点会开始主动感染周围，向周围发送`gossip`
+ 同一条消息会被反复推送. *`receiver`可以发送任意一组消息，或最新收到的消息，或最高优先级的消息. 这些都是`Pull Gossip`的变种*

相对而言，**Pull-based Gossip**:
+ 随机选取几点，周期性向周围轮训(`poll`)最新的组播消息

混合模式 **Hrbrid Push-Pull**:
+ 轮训周围的同时，向周围感染


