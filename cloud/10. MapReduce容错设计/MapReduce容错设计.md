## MapReduce Fault-Tolerance

#### 服务器容错设计
+ `Node Manager`向`Resource Manager`发送心跳包。服务器宕机后，`Resource Manager`会通知所有受影响的`Application Master`。
+ `Node Manager`会追踪所有当前服务器上运行的任务。如果有正在进行中的任务被终止，`Node Manager`将任务标记为`idle`, 并且重启任务
+ `Application Master`向`Resource Manager`发送心跳包。一旦心跳停止，`Resource Manager`重启`Application Master`，`Application Master`同步执行中任务的信息

#### `Resource Manager`容错设计
+ 使用旧的存档点(`checkpoints`)，启动`secondary Resource Manager`

*为了避免消息传输在网络中消耗过多的资源，`container`分配请求是通过心跳包传输的的*

#### `stragglers`(落后者)

`straggler`是指集群中执行缓慢的任务。每个阶段，执行最慢的任务，决定了总体的速度。`straggler`的原因可能有很多：磁盘损坏、网络拥堵、CPU瓶颈、内存瓶颈<br>

**容错设计:**
+ 跟踪记录每个任务的进度
+ `speculative execuation`: 对`straggler`另起一份副本执行`(repliacted execuation)`。一旦其中一份副本`(replica)`执行完成，整个任务被标记为完成。

假设`job1`、`job2`、`job3`的完成进度分别为`90%`、`50%`、`10%`。`Application Master`会在另一台服务器上运行`job3`的`replica`。一旦其中一个`job3`完成，整个任务会被标记为完成。

#### 地方性(locality)

+ 云具有继承性拓扑，机架内通信一般会快于跨核心交换机的通信
+ 数据被分成`chunk`，在不同机架备份三份
+ MapReduce 会根据网络距离调度任务 : 拥有对应数据副本的节点 > 在同一机架的节点 > 任意地点