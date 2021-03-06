#### Chapter 1 导论

操作系统的作用

1. 资源管理：

   1. 跟踪记录资源的使用状况
   2. 确定资源分配策略——算法：静态分配策略、动态分配策略（大多数OS使用，效率高）
   3. 实施资源的分配和回收
   4. 提供资源的利用率，保护资源的使用
   5. 协调多进程请求资源的冲突

>五大基本功能：
>
>进程/线程管理（CPU管理）、存储管理、文件管理、设备管理、用户接口

2. 向用户提供服务
3. 对硬件的拓展

操作系统的特征：
1. 并发：同时处理多个活动
2. 共享：OS与多个用户程序共同使用系统资源（互斥共享/同时共享）
3. 虚拟：一个物理实体映射为多个对应的逻辑实体（分时或分空间），以提高资源利用率
4. 随机：对以不可预测的次序发生的时间进行响应并处理

OS分类
1. 批处理操作系统（单道、多道），技术：SPOOLING
2. 分时操作系统，技术：时间片
通用操作系统：分时系统与批处理系统的结合
原则：分时优先，批处理在后
3. 实时操作系统
4. 个人计算机操作系统：在某一时间只为单用户服务
5. 网络操作系统
6. 分布式操作系统
7. 嵌入式操作系统
   
etc.

---
#### Chapter 2 操作系统结构
控制和状态寄存器：PC(程序计数器),IR(指令寄存器),PSW(程序状态字寄存器)

操作系统需要两种CPU状态：
Kernel Mode, User Mode

CPU状态之间的转换：
1. 用户态 -> 内核态：唯一途径：中断/异常/陷入机制
2. 内核态 -> 用户态：设置程序状态字

中断/异常机制
操作系统由“中断驱动”或者“事件驱动”

中断/异常的概念
CPU对系统发生的某个事件作出的一种反应
（事件的发生改变了处理器的控制流）
特点：随机发生，自动处理，可恢复

中断的引入：为了支持CPU和设备之间的并行操作
异常的引入：表示CPU执行指令时本身出现的问题（算术溢出、除零、地址越界、“陷入”指令，etc.）

中断和异常，统称事件
中断：外部事件，正在运行的程序所不期望的
Interrupt：来自I/O或其他硬件，异步，处理完返回到下一条指令

异常：由正在执行的指令引发
Trap陷入：有意识安排的，同步，处理完返回到下一条指令
Fault故障：可恢复的错误，同步，返回到当前指令
Abort终止：不可恢复的错误，同步，不返回

中断/异常机制的工作原理
硬件：中断/异常响应（发现中断，接受中断）
软件：中断/异常处理

开始->取下一条指令->执行指令->（允许中断）->检查指令处理中断
在每条指令执行周期的最后时刻扫描中断寄存器，查看是否有中断信号
若有中断，PSW写入中断码，通过查  中断向量表 引出中断处理程序
若无中断，继续执行下一条指令

中断向量表
中断向量：一个内存单元，存放 中断处理程序入口地址 和程序运行时所需的 处理机状态字

中断响应：
1. 设备发出中断信号
2. 硬件保存现场
3. 根据中断码查表
4. 把中断处理程序入口地址等推送到相应的寄存器
5. 执行中断处理程序

系统调用机制的设计

- 中断/异常机制：支持系统调用服务的实现
- 陷入指令（亦称访管指令）：引发异常，完成用户态到内核态的切换
- 系统调用号和参数：每个系统调用都事先给定一个编号（功能号）
- 系统调用表：存放系统调用服务程序的入口地址


<br>
参数传递的3中实现方法：
- 由陷入指令自带参数
- 通过通用寄存器传递参数（常用）
- 在内存中开辟专用栈区来传递参数

<br>
系统条用的执行过程
当CPU执行到陷入指令时：

- 中断/异常机制：硬件保护现场；通过查中断向量表把控制权转给 系统调用总入口程序
- 系统调用总入口程序：保存现场；将参数保存在内存栈区；通过查系统调用表把控制权转给相应的系统调用处理例程或内核函数
- 执行系统调用例程
- 恢复现场，返回用户程序

---
#### Chapter 3 进程 & Chapter 4 线程
并发环境：在一段时间间隔内，单处理器上又两个或两个以上的程序 同时 处于 开始运行但尚未结束的状态 并且 次序不是事先确定的

进程(Process，对CPU的抽象)：具有独立功能的程序关于 某个数据集合上 的 一次运行活动，是系统进行 资源分配和调度 的独立单位，又称任务(Task)

- 程序的一次执行过程
- 是正在运行程序的抽象
- 将一个CPU变成多个虚拟的CPU
- 系统资源以进程为单位分配，如内存、文件、……
- 每个进程具有独立的地址空间
- 操作系统将CPU调度给需要的进程

进程控制块(PCB)
Process Control Block

- 又称 进程描述符、进程属性
- 操作系统用于管理控制进程的一个专门的数据结构
- 记录进程的各种属性，描述进程的动态变化过程

PCB是系统感知进程存在的唯一标志，进程与PCB一一对应
进程表：所有进程的PCB集合

PCB包括：

- 进程描述信息
- 进程控制信息
- 所拥有的资源和使用情况
- CPU现场信息

进程的三种基本状态：

- 运行态：占有CPU，并在CPU上运行
- 就绪态：已经具备运行条件，但由于没有空闲CPU，而暂时不能运行
- 等待态(阻塞态、封锁态、睡眠态)：因等待某一事件而暂时不能运行，如：等待读盘结果

状态转换：

- 就绪->运行：调度程序选择一个新的进程运行
- 运行->就绪：运行进程用完了时间片；或一个高优先级进程进入就绪状态，抢占正在运行的进程
- 运行->等待：当一个进程等待某个事件发生时 e.g.请求OS服务、等待I/O结果……
- 等待->就绪：所等待的事件发生了

进程的其他状态：
创建态、终止态、挂起态

进程队列

- OS为每一类进程建立一个或多个队列
- 队列元素为PCB
- 伴随进程状态的改变，PCB会进入其他队列

进程控制
进程控制操作完成进程各状态之间的转换，由具有特定功能的原语完成

原语 primitive：
完成某种特定功能的一段程序，具有不可分割性或不可中断性
即原语的执行必须是连续的，在执行过程中不允许被中断（原子操作）

1. 进程的创建

- 给新进程分配一个唯一 标识 以及 进程控制块
- 为进程分配地址空间
- 初始化进程控制块
- 设置相应的队列指针

2. 进程的撤销
即结束进程

- 收回进程所占有的资源
- 撤销该进程的PCB

3. 进程阻塞
当被等待的事件未发生时，由 进程自己执行阻塞原语，是自己由运行态变为阻塞态

上下文(Context)切换

- 将CPU硬件状态从一个进程切换到另一个进程的过程
- 进程运行时，其硬件状态保存在CPU上的寄存器中
- 进程不运行时，这些寄存器的值保存在PCB中；当OS要运行一个新的进程时，将PCB中的相关值送到对应的寄存器中

进程的两个基本属性：资源的拥有者，CPU调度单位
线程继承“CPU调度单位”
进程依然是“资源的拥有者”
线程：进程中的一个运行实体，是CPU的调度单位，有时将线程称为 轻量级进程

线程的属性

- 标识符ID
- 状态及状态转换
- 上下文环境
- 有自己的栈和栈指针
- 共享所在进程的地址空间和其他资源
- 可以创建、撤销另一个线程（程序开始是以一个单线程进程方式运行的）

线程的实现
1. 用户级线程

- 在用户空间建立线程库
- 由运行时系统完成管理
- 内核管理的还是进程，不知道线程的存在
- 线程切换不需要内核态特权

e.g.UNIX

2. 核心级线程

- 内核管理所有的线程
- 内核维护进程和线程的上下文
- 线程的切换需要内核支持
- 以线程为基础进行调度

e.g.Windows

3. 混合模型

- 线程创建在用户空间完成
- 线程调度等在核心态完成

e.g.Solaris

---
#### Chapter 5 CPU调度
CPU调度——控制、协调进程对CPU的竞争

- 按一定的调度算法从就绪队列中选择一个进程，把CPU的使用权交给被选中的进程
- 如果没有就绪进程，系统会安排一个系统空闲进程或idle进程

CPU调度要解决三个问题：

- 调度算法
- 调度时机
- 调度过程(进程的上下文切换)

CPU调度的时机：就绪队列改变，引发重新调度

- 进程正常终止 或  由于某种错误而终止
- 新进程创建 或 一个等待进程变成就绪
- 当一个进程从运行态进入阻塞态
- 当一个进程从运行态变为就绪态

进程调度程序从就绪队列选择了要运行的进程——发生进程切换
进程切换：一个进程让出处理器，由另一个进程占用处理区的过程

- 切换全局页目录以加载一个新的地址空间
- 切换内核栈和硬件上下文（硬件上下文包括了内核执行新进程需要的全部信息）

切换过程包括了对原来运行进程各种状态的保存和对新进程各种状态的恢复

上下文切换开销
直接开销：内核完成切换所用的CPU时间

- 保存和恢复寄存器
- 切换地址空间

间接开销

- 高速缓存(Cache)、缓冲区缓存(Buffer Cache)和TLB(Translation Lookup Buffer)失效

CPU调度算法衡量指标

- 吞吐量(Throughput)：每单位时间完成的进程数目
- 周转时TT(Turnaround Time)：每个进程从提出请求到运行完成的时间
- 响应时间RT(Response Time)：从提出请求到第一次回应的时间
- CPU利用率(Cpu Utilization)：CPU做有效工作的时间比例
- 等待时间(Waiting Time)：每个进程在就绪队列(ready queue)中等待的时间

CPU调度算法要考虑的几个问题
1. 进程优先级(数)

- 静态优先级：进程创建时指定，运行过程中不再改变
- 动态优先级：进程创建时指定了一个优先级，运行过程中可以动态变化

2. 进程就绪队列组织：按优先级排队
3. 占用CPU的方式：

- 可抢占式Preemptive（可剥夺式）：当有比正在运行的进程优先级更高的进程就绪时，系统可强行剥夺正在运行进程的CPU，提供给具有更高优先级的进程使用
- 不可抢占式Non-preemptive（不可剥夺式）：某一进程被调度运行后，除非由于它自身的原因不能运行，否则一直运行下去

4. 按进程执行过程中的行为划分：

- I/O密集型或I/O型(I/O-bound)：频繁的进行I/O，通常会花费很多时间等待I/O操作的完成
- CPU密集型或CPU型或计算密集型(CPU-bound)：需要大量的CPU时间进行计算

5. 时间片(Time slice 或 quantum)：一个时间段，分配给调度CPU的进程，确定了允许该进程运行的时间长度

批处理系统中采用的调度算法：
1. 先来先服务(FCFS, First Come First Serve)

- 先进先出(FIFO)
- 按照进程就绪的先后顺序使用CPU
- 非抢占
- 优缺点：公平，实现简单；长进程后面的短进程等待时间比较长

2. 最短作业优先(SJF, Shortest Job First)

- 具有最短完成时间的进程优先执行
- 非抢占式
- 优缺点：最短的平均周转时间(所以进程同时可运行时)；不公平，产生”饥饿"(starvation)现象

3. 最短剩余时间优先(SRTN, Shortest Remaining Time Next)

- SJF抢占式版本
- 当一个新就绪的进程比当前运行进程具有更短的完成时间时，系统抢占当前进程，选择新就绪的进程执行
- 同SJF

4. 最高响应比优先(HRRN, Highest Response Ratio Next)

- 是一个综合算法：前面算法的折衷权衡(tradeoff)
- 非抢占
- 调度时，首先计算每个进程的响应比R; 之后，总是选择R最高的进程执行
- 响应比R = 周转时间 / 处理时间 = (处理时间 + 等待时间) / 处理时间 = 1 + (等待时间 / 处理时间)

交互式系统中采用的调度算法
1. 轮转调度(RR, Round Robin)

- 改善平均响应时间
- 周期性切换
- 每个进程分配一个时间片
- 时钟中断——轮换
- 在时间片到达时可以抢占
- 优缺点：公平，响应时间快；进程切换开销

2. 最高优先级调度(HPF, Highest Priority First)

- 选择优先级最高的进程投入运行
- 通常：系统进程>用户进程，前台进程>后台进程，操作系统偏好I/O型进程
- 优先级可以是静态不变的，也可以动态调整
- 就绪队列可以按照优先级组织
- 优缺点：实现简单；不太公平(饥饿)

3. 多级反馈队列(Multiple feedback queue)

- 是一个综合调度算法
- 设置多个就绪队列，第一级队列优先级最高
- 给不同就绪队列中的进程分配长度不同的时间片：第一级队列时间片最小；随着队列优先级降低，时间片增大
- 当第一级队列为空是，在第二级队列调度，以此类推
- 各级队列按照时间片轮转方式进行调度
- 当一个新创建进程就绪后，进入第一级队列
- 进程用完时间片而放弃CPU，进入下一级就绪队列
- 由于阻塞而放弃CPU的进程进入相应的等待队列，一旦等待的事件发生，该进程回到原来一级就绪队列
- 在时间片到达时可以抢占

4. 最短进程优先(Shortest Process Next)

---
#### Chapter 6 进程同步
竞争条件(Race Condition)：两个或多个进程读写某些共享数据，而最后的结果取决于进程运行的精确时序
进程互斥(Mutual Exclusive)：由于各进程要求使用共享资源(变量、文件等)，而这些资源需要排他性使用。各进程之间竞争使用这些资源。
临界资源(critical resource)：系统中某些资源一次只允许一个进程使用，称这样的资源为 临界资源 或 互斥资源 或 共享变量
临界区(互斥区, critical section/region)：各个进程中对某个临界资源(共享变量)实施操作的程序片段

临界区(互斥区)的使用原则

- 没有进程在临界区时，想进入临界区的进程可进入
- 不允许两个进程同时处于其临界区中
- 临界区外运行的进程不得阻塞其他进程进入临界区
- 不得使进程无限期等待进入临界区

实现进程互斥的解决方案

- 软件方案：Dekker解法、Peterson解法
- 硬件方案：屏蔽中断、TSL(XCHG)指令

进程同步(synchronization)
系统中多个进程中发生的事件存在某种时序关系，需要相互合作，共同完成一项任务。
一个进程运行到某一点时，要求另一协作进程为它提供消息，在未获得消息之前，该进程进入阻塞态，获得消息后被唤醒进入就绪态。

生产者/消费者问题，又称有界缓冲区问题
问题描述：

- 一个或多个生产者，生产某种类型的数据，放置在缓冲区中
- 有消费者从缓冲区中取数据，每次取一项
- 只能有一个 生产者或消费者 对缓冲区进行操作

要解决的问题：

- 当缓冲区已满时，生产者不会继续向其中添加数据
- 当缓冲区为空时，消费者不会从中移走数据
- 生产者和消费者不能同时对缓冲区进行操作

避免忙等待：
睡眠 与 唤醒 操作（原语）

信号量

- 一个特殊的变量
- 用于进程间传递信息的一个整数值
- 对信号量可以实施的操作：初始化、 P、V
- P、V操作为原语操作，(P:+1; V:-1)

用PV操作解决进程间互斥问题

- 分析并发进程的关键活动，划定临界区
- 设置信号量mutex，初值为1
- 在临界区前实施P(mutex)
- 在临界区之后实施V(mutex)

管程

- 是一种特殊的模块
- 每一个管程有一个名字
- 由关于共享资源的数据结构 及在其上操作的一组过程组成

进程与管程
进程只能通过调用管程中的结构来间接地访问管程中的数据结构

管程需要解决两个问题

- 互斥：管程是互斥进入的。管程的互斥性是由编译器负责保证的
- 同步：管程中设置条件变量及等待/唤醒操作以解决同步问题

进程间通信

- 消息传递
- 共享内存
- 管道
- 套接字
- 远程过程调用

---
#### Chapter 7 死锁
死锁：一组进程中，每个进程都无限等待被该组中另一进程所占有的资源，因而永远无法得到的资源

产生死锁的必要条件

- 互斥使用(资源独占)：一个资源每次只能给一个进程使用
- 占有且等待(请求和保持，部分分配)：进程在申请新的资源的同时，保持对原有资源的占有
- 不可抢占(不可剥夺)：资源申请者不能强行的从资源占有者手中夺取资源，资源只能由占有者自愿释放
- 循环等待：存在一个进程等待队列，形成一个进程等待环路

资源分配图：用有效图描述系统资源和进程的状态

死锁定理

- 如果资源分配图中没有环路，则系统中没有死锁；如果图中存在环路，则系统中可能存在死锁
- 如果每个资源类中只包含一个资源实例，则环路是死锁存在的充分必要条件

解决死锁

- 鸵鸟算法：不考虑此问题
- 死锁预防：静态策略：设计合适的资源分配算法，不让死锁发生
- 死锁避免：动态策略：以不让死锁发生为目标，跟踪并评估资源分配过程，根据评估结果决策是否分配
- 死锁检测与解除：让死锁发生

死锁预防(Deadlock Prevention)

- 在设计系统时，通过确定资源分配算法，排除发生死锁的可能性
- 防止产生死锁的四个必要条件中任何一个发生
    - 破坏“资源独占”
        - 资源转换技术：把独占资源变为共享资源
        - SPOOLing技术的引入
    - 破坏“占有且等待”
        - 方案1：每个进程在运行前必须一次性申请要求的所有资源(问题：资源利用率低；饥饿现象)
        - 方案2：进程等待时，必须释放已占有的全部资源，若需要在重新申请
    - 破坏“不可抢占”
        - 用过操作系统抢占该资源（两个进程优先级不同）
    - 破坏“循环等待”
        - 定义资源类型的线性顺序实现：资源有序分配法

死锁避免：在系统运行过程中，对进程发出的每一个系统能够满足的资源申请进行动态检查，并根据检查的结果决定是否分配资源
安全状态：系统中存在一个由所有进程构成的安全序列
安全状态一定没有死锁发生；不安全状态一定导致死锁(但目前还没有)

银行家算法

- 死锁避免算法
- 应用条件：
    - 在固定数量的进程中共享数量固定的资源
    - 每个进程预先指定完成工作所需的最大资源数量
    - 进程不能申请比系统中可用资源总数还多的资源
    - 进程等待资源的时间是有限的
    - 如果系统满足了进程对资源的最大需求，那么，进程应该在有限的时间内使用资源，然后归还给系统

死锁的检测与解除

- 允许死锁发生，但是操作系统会不断监视系统进展情况，半段死锁是否真的发生
- 一旦死锁发生则采取专门的措施，解除死锁并以最小的代价恢复操作系统运行

---
#### Chapter 8 内存管理

- 逻辑地址(相对地址，虚拟地址)
    - 用户程序经过编译、汇编后形成目标代码，目标代码通常采用相对地址的形式，其首地址为0，其余地址都相对于首地址而编址
    - 不能用逻辑地址在内存中读取信息
- 物理地址(绝对地址，实地址)
    - 内存中存储单元的地址
    - 可直接寻址
- 地址重定位:
    - 为了保证CPU执行指令时可正确访问内存单元，需要将用户程序中的逻辑地址转换为运行时可由及其直接寻址的物理地址

- 静态重定位：
    - 当用户程序加载到内存时，一次性实现逻辑地址到物理地址的转换
    - 一般可以由软件完成
- 动态重定位（常用）：
    - 在进程执行过程中进行地址变换，即逐条指令执行时完成地址转换
    - 需要硬件部件支持

内存管理单元(Memory Management Unit, MMU)
重定位寄存器，将逻辑地址变为物理地址

空闲内存管理

- 等长划分：位图
- 不等长划分：空闲区表、已分配区表，空闲块链表

内存分配算法：

- 首次适配(first fit)：在空闲区表中找到第一个满足进程要求的空闲区
- 下次适配(next fit)：从上次找到的空闲区处接着查找
- 最佳适配(best fit)：查找整个空闲区表，找到能够满足进程要求的最小空闲区
- 最差适配(worst fit)：总是分配满足进程要求的最大空闲区

伙伴系统

- 一种经典的内存分配算法
- Linux底层内存管理采用
- 一种特殊的“分离适配算法”
- 主要思想：将内存按2的幂进行划分，组成若干空闲块链表；查找该链表找到能满足进程需求的最佳匹配块

基本内存管理方案

- 单一连续区
- 固定分区
- 可变分区
- 页式
- 段式
- 段页式

单一连续区

- 特点：一段时间内只有一个进程在内存
- 简单，内存利用率低

固定分区

- 把内存空间分割成若干区域，称为分区
- 每个分区的大小可以相同也可以不同
- 分区大小固定不变
- 每个分区装一个且只能装一个进程

可变分区

- 根据进程的需要，把内存空闲空间分配给该进程
- 剩余部分成为新的空闲区
- 产生外碎片，导致内存利用率下降
- 碎片问题：
    - 碎片：很小的、不易利用的空闲区->导致内存利用率下降
    - 解决方案：紧缩技术(memory compaction, 又称压缩技术、紧致技术、搬家技术)：在内存移动程序，将所有小的空闲区合并为较大的空闲区

页式

- 用户进程地址空间被划分为大小相等的部分，称为页(page)或页面，从0开始编号
- 内存空间按同样大小划分为大小相等的区域，称为页框(page frame，物理页面/页帧/内存块)，从0开始编号
- 内存分配：以页为单位进行，并按照进程需要的页数来分配；逻辑上相邻的页，物理上不一定相邻
- 典型页面尺寸：4K 或 4M
- 逻辑地址：页号(高20位)+页内地址(偏移, 低12位)；划分是由系统自动完成的，对用户是透明的
- 页表：
    - 页表项：记录了逻辑页号与页框号的对应关系
    - 每一个进程一个页表，存放在内存
    - 页表起始地址保存在页表基址寄存器(page table base register),简称PTBR
- 地址转换(硬件支持)：CPU取到逻辑地址(页号+页内偏移)，自动划分为页号和页内偏移；用页号查页表，取得页框号，再与页内偏移拼接成为物理地址
- 空闲内存管理：内碎片

段式

- 用户进程地址空间：按程序自身的逻辑关系划分为若干个程序段，每个程序段都有一个段名
- 内存空间被动态划分为若干长度不相同的区域，称为物理段，每个物理段由起始地址和长度确定
- 内存分配：以段为单位进行分配，每段在内存中占据连续空间，但各段之间可以不相邻
- 逻辑地址：段号+段内地址；不是自动划分，必须显示给出
- 段表：
    - 每项记录了段号、段首地址和段长度之间的关系
    - 每一个进程一个段表，存放在内存
    - 段表起始地址保存在段表基址寄存器
- 地址转换(硬件支持)：CPU取到逻辑地址(段号+段内地址)，用段号查段，得到该段在内存的起始地址，与段内偏移地址计算出物理地址

段页式

- 用户进程划分：先按段划分，每一段再按页面划分
- 逻辑地址：段号+段内地址，段内地址=页号+页内地址
- 内存划分：等长划分(同页式存储管理方案)
- 内存分配：以页为单位进行分配(同页式存储管理方案)
- 段表：记录了每一段的页表起始地址和页表长度
- 页表：记录了逻辑页号与页框号的对应关系
- 每一段有一张页表，一个进程有多个页表
- 空闲区管理：同页式存储管理方案

交换技术
内存“扩充”技术：在较小的内存空间运行较大的进程

- 内存紧缩技术(例如：可变分区)
- 覆盖技术 overlaying
- 交换技术 swapping
- 虚拟存储技术 virtual memory

覆盖技术 overlaying

- 解决的问题：程序大小超过物理内存总和
- 程序执行过程中，程序的不同部分在内存中相互替代：按照其自身的逻辑结构，将那些不会同时执行的程序共享同一块内存区域；要求程序各模块之间有明确的调用结构
- 程序员声明覆盖结构，操作系统完成自动覆盖(主要用于早期的操作系统)

交换技术 swapping

- 内存空间紧张时，系统将内存中某些进程暂时移到外存，把外存中某些进程换进内存，占据前者所占用的区域（进程在内存与磁盘之间的动态调度）
- 交换的内容：运行时创建或修改的内容——栈和堆
- 交换区：一般系统会指定一块特殊的磁盘区域作为交换空间(swap space)，包含连续的磁道，操作系统可以使用底层的磁盘读写操作对其高效的访问
- 交换发生的时机：只要不用就换出(很少再用)；内存空间不够或有不够的危险时换出。与调度器结合使用
- 考虑进程的各种属性：不应换出处于等待I/O状态的进程

---
#### Chapter 9 虚拟内存

- 虚拟内存：当进程运行时，先将其一部分装入内存，另一部分暂留在磁盘，当要执行的指令或访问的数据不在内存时，由操作系统自动完成将它们从磁盘调入内存的工作
- 虚拟地址空间：分配给进程的虚拟内存
- 虚拟地址：在虚拟内存中指令或数据的位置，该位置可以被访问，仿佛它是内存的一部分

虚拟页式存储管理：虚拟存储技术+页式存储管理方案

- 请求调页(demand paging)
- 预先调页(pre-paging)

页表项设计：

- 页框号（内存块号、物理页面号、页帧号）：对应具体页面
- 有效位（驻留位、中断位）：表示该页是在内存还是在磁盘
- 访问号（引用位）：该页是否被访问过
- 修改位：此页在内存中是否被修改过（被修改后，需写回到磁盘）
- 保护位：只读/可读写

通常，页表项是硬件设计的

内存管理单元(MMU)：通过虚拟地址，计算出物理地址
为了加快地址映射速度，引入块表(TLB)：程序访问的局部性原理

快表

- TLB (Translation Look-aside Buffers)：一直随机存取型存储器
- 相联存储器(associative memory)：特点：按内容并行查找
- 保存正在运行进程的页表的子集(部分页表项)

页错误(PAGE FAULT)

- 又称 页面错误、页故障、页面失效
- 地址转换过程中硬件产生的异常
- 原因：
    - 所访问的虚拟页面没有调入物理内存——缺页异常
    - 页面访问违反权限（读/写、用户/内核）
    - 错误的访问地址

缺页异常处理

- 在地址映射过程中，硬件检查页表时发现所要访问的页面不在内存，则产生该异常
- 操作系统执行缺页异常处理程序：获得磁盘地址，启动磁盘，将该页调入内存
    - 如果内存中有空闲页框，则分配一个页框，将新调入页装入，并修改页表中相应页表项的有效位及相应的页框号
    - 若内存中没有空闲页框，则要置换内存中某一页框；若该页框内容被修改过，则要将其写回磁盘

驻留集：给每个进程分配多少页框

- 固定分配策略
    - 进程创建时确定
    - 可以根据进程类型（交互、批处理、应用类）或者基于程序员或系统管理员的需要来确定
- 可变分配策略
    - 根据缺页率评估进程局部性表现
        - 缺页率高->增加页框数
        - 缺页率低->减少页框数
    - 会带来新的系统开销

置换范围

- 局部置换策略：仅在产生本次缺页的进程的驻留集中选择
- 全局置换策略：将内存中所有未锁定的页框都作为置换的候选

置换策略：决定置换当前内存中的哪一个页框

- 目标：置换最近最不可能访问的页
- 根据局部性原理，最近的访问历史和最近将要访问的模式间存在相关性。因此，大多数策略都是基于过去的行为预测将来的行为
- 约束：不能置换被锁定的页框

页面锁定：

- 采用虚拟内存后，新的开销使进程运行时间变得不确定
- 给每一页框增加一个锁定位，通过设置相应的锁定位，不让操作系统将进程使用的页面换出内存，避免产生由交换带来的不确定的延迟
- 例如：操作系统核心代码、关键数据结构、I/O缓冲区

清除策略

- 清除：从进程的驻留集中收回页框
- 虚拟页式系统工作的最佳状态：发生缺页异常时，系统中有大量的空闲页框
- 结论：在系统中保存一定数目的空闲页框供给 比使用所有内存并在需要时搜索一个页框有更好的性能
- 设计一个分页守护进程：
    - 多数时间处于睡眠，可定期唤醒以检查内存的状态
    - 如果空闲页框过少，分页守护进程通过预定的页面置换算法选择页面换出内存
    - 如果页面装入内存后被修改过，则将它们写回磁盘分页守护进程可保证所有的空闲页框是“干净”的
- 页缓冲技术：
    - 不丢弃置换出的页，将它们放入两个表之一：如果未被修改，则放到空闲页链表中，如果修改了，则放到修改页链表中
    - 被修改的页定期写回磁盘
    - 被置换的页仍然保留在内存中，一旦进程又要访问该页，可以迅速将它加入该进程的驻留集合

页面置换算法
最佳页面置换算法(OPT)

- 设计思想：置换以后不再需要的或最远的将来才会用到的页面
- 作为一种标准来衡量其他算法的性能

先进先出算法(FIFO)

- 选择在内存中驻留时间最长的页并置换它
- 实现：页面链表法

第二次机会算法(SCR, Second Chance)

- 按照先进先出算法选择某一页面，检查其访问位R：如果为0，则置换该页；如果为1，则给第二次机会，并将访问位置0
- 摘链挂链带来新的开销
- 改进算法：时钟算法(CLOCK)

最近未使用算法(NRU, Not Recently Used)

- 选择在最近一段时间内未使用的一页并置换
- 实现：设置页表表项的两位：访问位（R），修改位（M）
- 启动一个进程时，R、M位置0；R位被定期清零
- 算法思想：随机从编号最小的非空类中选择一页置换：
    - 无访问，无修改
    - 无访问，有修改
    - 有访问，无修改
    - 有访问，有修改

最近最少使用算法(LRU, Least Recently Used)

- 选择最后一次访问时间距离当前时间最长的一页并置换，即置换未使用时间最长的一页
- 性能接近OPT
- 实现：时间戳 或 维护一个访问页的栈，但开销大

最不经常使用算法(NFU, Not Frequently Used)

- 选择访问次数最少的页面置换
- LRU的一种软件解决方案
- 实现：
    - 软件计数器，一页一个，初值为0
    - 每次时钟中断时，计数器加R
    - 发生缺页中断时，选择计数器值最小的一页置换
- 改进算法：老化算法

老化算法(AGING)
改进（模拟LRU）：计数器在加R前先右移一位，R加到计数器的最左端

BELAGY现象
FIFO页面置换算法会产生异常现象（Belady现象）：当分配给进程的物理页面数增加时，缺页次数反而增加

颠簸(Thrashing, 抖动)
虚拟内存中，页面在内存与磁盘之间频繁调度，使得调度页面所需的时间比进程实际运行的时间还多，这样导致系统效率急剧下降

工作集模型

- 一般情况下，进程在一段时间内总是集中访问一些页面，这些页面称为 活跃页面
- 如果分配给一个进程的物理页面数太少，使该进程所需的活跃页面不能全部装入内存。则进程在运行过程中将频繁发生中断
- 如果能为进程提供与活跃页面数相等的物理页面数，则可减少缺页中断次数

工作集：一个进程当前正在使用的页框集合

工作集算法

- 找出一个不在工作集的页面并置换它
- 每个页表项中有一个字段：记录该页面最后一次被访问的时间
- 设置一个时间值T
- 判断：根据一个页面的访问时间是否落在“当前时间 - T”之前还是之中，决定其在工作集之外还是之内

内存映射文件：进程通过一个系统调用(mmap)将一个文件（或部分）映射到其虚拟地址空间的一部分

---
#### Chapter 10 文件系统
文件

- 是对磁盘的抽象
- 一组带标识(即文件名)的、在逻辑上有完整意义的信息项的序列
- 信息项：构成文件内容的基本单位，各信息项之间有顺序关系

文件系统

- 统一管理磁盘空间，实施磁盘空间的分配与回收
- 实现文件的按名存取：名字空间→磁盘空间
- 实现文件信息的共享，并提供文件的保护、保密手段
- 向用户提供接口
- 提高性能
- 提供与I/O系统的统一接口

文件控制块(File Control Block)：为管理文件而设置的数据结构，保存管理文件所需的所有有关信息(文件属性或元数据)

文件目录

- 统一管理每个文件的元数据，以支持文件名到文件物理地址的转换
- 将所有文件的管理信息组织在一起，即构成文件目录

目录文件

- 将文件目录以文件的形式存放在磁盘上

目录项

- 构成文件目录的基本单元
- 目录项可以是FCB，目录是文件控制块的有序集合

文件的物理结构

- 连续(顺序)结构：文件的信息存放在若干连续的物理块中
- 链接结构：一个文件的信息存放在若干不连续的物理块中，各块之间通过指针连接
- 索引结构：
    - 一个文件的信息存放在若干不连续的物理块中
    - 系统为每一个文件建立一个专用数据结构——索引表，并将这些物理块的块号存放在该索引表中
    - 索引表就是磁盘块地址数组，其中第i个条目指向文件的第i块

磁盘分区(partition)：把一个物理磁盘的存储空间划分为几个相互独立的部分，称为分区
文件卷(volume)：磁盘上的逻辑分区，由一个或多个物理块组成
格式化：在一个文件卷上建立文件系统，即建立并初始化用于文件分配和磁盘空闲空间管理的管理数据
