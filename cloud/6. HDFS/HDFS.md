## HDFS

`HDFS`是`Hadoop`分布式文件系统的简称，特点：
1. 由若干计算机组成，尤其适合部署在廉价的机器上
2. 适合存储非常大的文件 （TB, PB, EB）
3. 每份文件可以有多个副本，所以`HDFS`是高容错、高冗余的文件系统

## 常见的分布式架构

|结构|描述|优势|劣势|
|--|--|--|--|
|Peer-to-Peer|N台机器相互通信，互为备份|任何结点宕机都不影响工作|机器之间经常需要通讯，保持一致性|
|Master Slave（工业界90%采用此设计）|1组机器负责管理，余下负责运算|结构简单；数据很容易保持一致|Master宕机，单点故障|

![P2P](https://github.com/s09g/notes/raw/master/cloud/6.%20HDFS/assets/1.jpg)
![Master Slave](https://github.com/s09g/notes/raw/master/cloud/6.%20HDFS/assets/2.jpg)

`HDFS`采取`Master-Slave`结构

## HDFS 组件
![hdfs](https://github.com/s09g/notes/raw/master/cloud/6.%20HDFS/assets/3.jpg)

1. `Client`: 负责与HDFS通信
2. `NameNode`
    + 存放文件元数据（镜像文件`fsimage` + 变更日志`edit logs`）
        + 整个`HDFS`集群只有一个`NameNode`，它存储整个集群文件分别的元数据信息。这些信息以`fsimage`和`edit logs`两个文件存储在本地磁盘，`Client`通过这些元数据信息可以找到相应的文件。
    + 记录数据块所在`DataNode`的信息
        + `NameNode`还负责监控`DataNode`的健康情况，一旦发现`DataNode`异常，就将其踢出，并拷贝其上数据至其它`DataNode`
3. `DataNode`
	+ 存储并检索数据块
        + `DataNode`负责数据的实际存储。当一个文件上传至`HDFS`集群时，它以`Block`为基本单位分布在各个`DataNode`中
        + 为了保证数据的可靠性，每个`Block`会同时写入多个`DataNode`中（默认为3）
            + `Hadoop` 1.x 默认`Block`大小为64MB
            + `Hadoop` 2.x 默认`Block`大小为128MB
	+ 向`NameNode`更新数据块列表（心跳）
4. `Secondary NameNode`
    + `Secondary NameNode`负责定期合并`NameNode`的`fsimage`和`edit logs`
    + 它不是`NameNode`的热备，所以`NameNode`依然可能造成单点故障（Single Point of Failure）

## HDFS读文件

1. `client`向`namenode`发送读数据请求
2. `namenode`将`datanode`信息发给`client`
3. `client`从`datanode`下载数据
![client reading data from HDFS](https://github.com/s09g/notes/raw/master/cloud/6.%20HDFS/assets/5.jpg)

`HDFS`会自动根据距离选取最近的数据源。距离从近到远：**同一机器 < 同一机架 < 同一数据中心 < 不同数据中心**
![network distance in Hadoop](https://github.com/s09g/notes/raw/master/cloud/6.%20HDFS/assets/6.jpg)


## HDFS写文件

1. `client` 向`namenode` 发送写数据请求
2. `namenode` 分块 写入`datanode`
3. `datanode` 自动完成副本备份
4. `datanode` 完成后向`namenode`汇报
5. `namenode`向`client`发送通知
![client writing data to HDFS](https://github.com/s09g/notes/raw/master/cloud/6.%20HDFS/assets/7.jpg)

数据多副本备份又`datanode`自行完成。一个典型的复制流程是：`namenode`向附近的`datanode_1`写入第一份数据，`datanode_1`自动向不同机架的`datanode_2`复制出第二份数据。`datanode_2`自动寻找可用的`datanode_3`, 向`datanode_3`复制出第三份数据。
![replica pipeline](https://github.com/s09g/notes/raw/master/cloud/6.%20HDFS/assets/8.jpg)


## HDFS指令
`HDFS` 与`Linux`指令基本一致
```bash
hdfs dfs -ls /  # 列出/路径下的文件
hdfs dfs -mkdir /test  #创建/test文件夹
hdfs dfs -rm -r /test  #删除/test文件夹
hdfs dfs -put words.txt /test   #将words.txt上传到/test
hdfs dfs -ls /test  # 列出/test路径下的文件
hdfs dfs -cat /test/words.txt   #在控制台打印出/test/words.txt文件的内容
hdfs dfs -get /test/words.txt . #将/test/words.txt从HDFS下载到本地
```
