## MapReduce 范式

#### 什么是MapReduce

`Map`和`Reduce`源自于`Functional Language`
![](https://github.com/s09g/notes/raw/master/cloud/4.%20MapReduce%20Paradigm/assets/1.png)

+ `map`是一种**顺序且独立**处理集合中每个元素的元函数
+ `reduce`是**分批**处理集合元素的元函数

## Example : Word Count
我们以以下文档作为样例

> Welcome Everyone <br>
> Hello Everyone

#### Map阶段
`Map`任务会对每个`record`产生一个键值对`(key/value pair)`。
例如，`(Welcome, 1)` 这意味着曾经遇到过`Welcome`这个词
这些`record`可以相互独立地处理。
![](https://github.com/s09g/notes/raw/master/cloud/4.%20MapReduce%20Paradigm/assets/2.png)


在这个例子中只有一个`Map`任务，所以它按顺序处理这些`record`。
当数据量特别巨大时候，这个过程很容易被中断。
![](https://github.com/s09g/notes/raw/master/cloud/4.%20MapReduce%20Paradigm/assets/3.png)
需要对输入数据集进行分片或拆分，分配给每个镜像或每个独立的`Map`任务, 并产生输出
![](https://github.com/s09g/notes/raw/master/cloud/4.%20MapReduce%20Paradigm/assets/4.png)


#### Reduce
`Reduce`基于每个键合并、处理中间结果
![](https://github.com/s09g/notes/raw/master/cloud/4.%20MapReduce%20Paradigm/assets/5.png)

每条记录会根据key的不同，被分配给不同的`Reduce`任务。通常会使用`Hash paritioning`算法来分配`key`。`Reduce`任务根据分配到的`key`并行地处理、合并中间值。
![](https://github.com/s09g/notes/raw/master/cloud/4.%20MapReduce%20Paradigm/assets/6.png)
