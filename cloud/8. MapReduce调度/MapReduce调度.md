## MapReduce Scheduling

#### Scheduler

1. 并行`Map`任务
    + `splitting and sharding data`
    + `Map`任务相互独立
2. 将数据从`Map`传输到`Reduce`
    + 相同`key`的`Map`输出会分配给同一个`Reduce`任务
    + 利用了`partition`函数，比如`hash(key) % number_of_reducers`
3. 并行`Reduce`任务
    + `Reduce`任务相互独立
4. 实现存储
    + 数据通常会有三个副本位于三个不同的服务器上
    + `Map Input`: 来自分布式文件系统
    + `Map Output`: `Map`节点的本地磁盘(本地文件系统)
    + 中间数据对外部用户不可见，也不必写到分布式文件系统上
    + `Reduce Input`: 远程磁盘(本地文件系统)
    + `Reduce Output`: 分布式文件系统

理论上，`Reduce`阶段只能在所有`Map`阶段结束之后启动(未结束的`Map`任务可能产生新的`key/value`对，对应该`key`的`Reduce`任务需要等待`Map`完成)。这种两个阶段之间的隔离操作叫做`barrier`。

<br>

*事实上部分`Reduce`任务是可以提早开始的。`MapReduce`中也是这样实现的。但是这种操作不利于我们理解`MapReduce`范式，所以我们先忽略这件事。*

*`Barrier`不成立的原因之一，是在`Map`阶段和`Reduce`阶段之间存在`Shuffle`阶段。`Shuffle`可以和`Map`并行执行。*
> *PS.推荐两篇文章[《MapReduce:详解Shuffle过程》](https://langyu.iteye.com/blog/992916)[《MapReduce的shuffle过程详解（分片、分区、合并、归并）》](https://blog.csdn.net/ASN_forever/article/details/81233547)，对这段shuffle的梳理实在是妙。大致解释一下：* <br>
> Map任务的结果不会立刻写入磁盘，而是写到一个叫**环形内存缓冲区**的地方（这个操作叫`spill`）。`spill`的时候，会根据key进行分区`(partition)`。缓冲区默认最大是`100M`，当写入达到阈值(默认是`80%`)的时候，会启动一个线程将缓冲区文件写到磁盘临时文件。而这个线程会执行一个排序`(sort)`和一个合并`(combine)`操作。整个spill执行完之后，会对所有临时文件进行归并`(merge)`。`merge`时会继续进行`sort`和`combine`来减少最终输出大小。<br>
> 上面这段流程就是`map`端的`shuffle`操作，里面的`combine`是可选的，部分情况下其实执行的是`reduce`。

![](https://github.com/s09g/notes/raw/master/cloud/8.%20MapReduce%E8%B0%83%E5%BA%A6/assets/1.png)

所以，`spill`时首先进行`partition`，然后`partition`内`sort`、`combine`，最后写出到磁盘。而`combine`可以是`reduce`，所以`Map`和`Reduce`之间不存在`Barrier`。

#### YARN

`YARN = Yet Another Resource Negotiator`. `YARN`是从`Hadoop 2.x` 开始引入的资源调度器。

`YARN`将每个服务器看成一组`容器(container)`。`Container = some CPU + some memory`。每个容器可以执行一个任务
> 如果服务器有4个CPU和4GB内存，而每个容器中有一个CPU和1GB的RAM。那么这个服务器有4个容器，可以运行四个任务。

YARN有三个主要部分：
+ `Resource Manager` 资源管理器 `RM`
    + `Resource Manager`是全局进程
    + 负责调度
+ `Node Manager` 节点管理器 `NM`
    + `Node Manager`在每个`server`都有一个
    + 作为守护进程和运行特定服务器进程（比如，任务监控）
+ `Application Master` 应用管理`AM`
    + 应用级别 `per-application(job)`
    + 负责`container`与`Resource Manager`、`Node Manager`之间协商通信
    + 与`Node Manager`通信，检测任务挂起和重新调度

#### `YARN`分配`container`

![](https://github.com/s09g/notes/raw/master/cloud/8.%20MapReduce%E8%B0%83%E5%BA%A6/assets/2.png)

两台服务器A、B：每个服务器有一个`Node Manager`在运行<br>
两个任务1、2：每个任务有一个`Application Master`<br>
全局有一个`Resource Manager`在运行<br>

Timeline:

|sequence|environment|action|
|--|--|--|
|0|开始时，`Job2(App2)`刚刚运行结束，`Job1(App1)`即将启动|N/A|
|1|`Job1(App1)`即将启动|`Application Master1(AM_1)`通知`Resource Manager(RM)` <`App1`即将启动，需要分配一个`container`> |
|2|`RM`收到`AM_1`的消息，但无可分配的`container`|`RM`将`AM_1`消息放入队列挂起，随后`Node Manager B(NM_B)`向`RM`发送消息 <`container`空闲>|
|3|`RM`收到`NM_2`的消息|`RM`通知`AM_1`, `node B`有空闲`container`|
|4|`AM_1`收到`RM`消息|`AM_1`通知`NM_B`执行`Job1`|

*实际运行中，每个任务会申请多个container，Resource Manager会根据申请的顺序分配container*