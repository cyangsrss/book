# HDFS Architecture

## 简介

### hdfs 特性

1. HDFS 容错性很高 并且被设计部署在廉价的硬件上
2. 对应用数据提供高吞吐量的通道，非常适合有大数据量的应用
3. HDFS为了使得文件系统的流数据成为可能，松开了一些POSIX协议

## 假设和目标

### 硬件损坏

HDFS是由成千上完的机器组成的，硬件损坏是很正常，因此自动发现问题，并迅速自我恢复是HDFS的一个架构目标

### 流数据入口

由于HDFS的重点更多的是为了高吞吐量的数据，而并非低延迟的数据接入。POSIX的一些要求并不太适合HDFS，这些语义将被放弃以满足提高数据吞吐量。

### 大数据集合

hdfs的一个标准文件应该是GB到TB的。一个HDFS可以支持的文件数应该是上千万的。

### 简单的一致性模型

hdfs支持一次写入多次读取的模式。一个文件一旦被写入，支持append 和truncate操作，但是不支持修改。这个假设，将一致性的问题简单化了，并且使得高吞吐量的实现更为容易。

### 移动计算能力比移动存储开销更小

如果计算靠近存储的话，效率会更高。特别是在数据集特别大的时候，更是如此。这减少了网络拥堵，并且增加了整个系统的吞吐量。
我们假设把计算移动到靠近数据的节点比把数据移动到靠近计算的节点更好。
HDFS提供接口让应用程序可以移动到靠近数据的位置。

### 易于从不同的软件和硬件平台移植

HDFS很容易从一个平台移动到另外一个平台。

## NameNode and DataNode

1. HDFS是一个master/slave架构。一个HDFS集群包含1个NameNode和多个DataNode。
2. NameNode负责管理文件系统的命名空间以及文件的权限。
3. DataNode通常是每台节点一个，负责管理它所运行的机器上的存储。
4. 在HDFS里面，文件被拆分成一个或者多个BLOCK。block被存储在不同的datanode上。
5. NameNode执行一些命名空间的操作。类似，打开、关闭和重命名文件和目录。另外，HDFS决定block应该被存储在哪个datanode上。
6. DataNode负责处理文件系统的客户端的写入和读取请求。DataNode也处理从Namenode下达的创建、删除、复制block的指令。
![hdfs](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/images/hdfsarchitecture.png)
