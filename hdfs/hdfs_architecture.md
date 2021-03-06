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
7. HDFS使用java构建。任何支持java的机器都可以运行NameNode和DataNode。Java的高度可移植的特点可以让HDFS被部署到大部分的机器上。一个标准的部署中，通常会有一台机器专门用来运行NameNode。这个架构并不妨碍你在一台机器中部署运行多个DataNode，但是这在真实的部署中是非常少见的。
8. 一个单独的NameNode简化了整个系统的架构。NameNode在整个系统中是一个仲裁者，并负责HDFS的元数据信息。系统被设计成这样使得用户的数据不会经过NameNode

## 文件系统的命名空间

1. HDFS支持传统的等级划分的文件组织方式。文件被用户或应用程序存放在目录里。
2. HDFS支持用户quotas和准入权限。
3. HDFS不支持硬连接和软连接。但是HDFS也不排除实现这些特性。
4. HDFS服从文件系统的命名约定，有些目录和命名被保留了（比如/ .reserved和.snapshot）。一些特性，比如**透明加密**和**snapshot**用到了这些被保留了的命名。
5. NameNode负责管理整个文件系统的命名空间。任何有关命名空间或者是命名空间的配置的改动都会被NameNode所记录。应用程序可以设定文件的备份数，这个信息会被NameNode存储。

## Data Replication

1. HDFS被设计成在一个大的集群里存储大文件的的可靠系统。
2. HDFS将文件存储成一个序列的block
3. block将被存储多份，以满足错误容忍
4. 每个文件的block的大小和文件的大小都是可配置的
5. 一个文件，除了最后一个block的大小是不固定的之外，其他的都是固定的大小。用户可以在不填充满最后一个block的时候启动新的block。
6. 文件的备份数是随时可以修改的。一个文件只能被写入一次（除非是appends和truncates操作）并且在任何时候永远只有一个写入者。
7. NameNode的所有决策都与block的备份数有关。它不停的收到从DataNode发来的心跳和Blockreport。心跳代表DataNode目前正常工作。Blockreport包含该DataNode的所有block的list
![image](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/images/hdfsdatanodes.png)

### Replica Placement: The First Baby Steps

Replica被防止的位置对于HDFS的可靠性和性能至关重要。优化Replica placement将HDFS同其他分布式系统区分开来。这是一个需要大量调试和经验的特性。一个机架可感知的replica placement policy会提高数据的可靠性和性能以及带宽性能。
NameNode通过一个程序决定某台DataNode属于哪个rackid。通常同一个Rack里面机器之前的带宽要高于不同Rack的机器之间的带宽。一个简单的没有优化过的策略就是把不同的replica放在单独的rack上。这个策略的好处就是可以防止当某个rack挂了的时候数据丢失问题，并且允许使用不同rack上的带宽去读取数据。坏处就是增加了写入的时候的rack之间的带宽的消耗。
通常的做法是3份replica的其中2个放在一个rack另一个放在另外一个rack上。
除了rack awareness之外，hdfs还加入了存储类型和存储策略。NameNode首先基于rack awareness选择nodes，然后再基于文件的存储策略去选择nodes。

### Replica Selection

HDFS client 偏向于去读靠近client位置的replica，比如同一个rack，比如同一个data center。

### safemode

在HDFS启动的时候，NameNode会进入一个safemode的状态。 在这个状态下，block不会被复制。NameNode接受DataNode上报的心跳和Blockreport。Blockreport包含一系列的data blocks。每一个block都会有一个最小的replicas数。如果NameNode检测到某个block的replica数达到了最小的配置数之后，这个block就被标记为safe。当检测到safe的block比例达到一个阈值之后，NameNode就会跳出safemode。如果这个时候有一些block的复制数比配置的小的话，NameNode就开始复制数据。

## The Persistence of File System Metadata

1. HDFS的元信息的任何change都被存储在一个EditLog的文件里。元信息的镜像被存储在FsImage里面。
2. NameNode在内存里面存储一份整个File system namespace 和 blockmap
![image](http://i1.fuimg.com/712525/145622c1f413ab11.png)
3. checkpoint 可以被配置xx时间后执行一次，也可以被配置xxxtranscations被执行一次。相关的配置是dfs.namenode.checkpoint.period/dfs.namenode.checkpoint.txns。如果两个配置都加了的话，先到达触发条件的触发一次checkpoint。
4. DataNode会给每个Block存放在一个file里面。也会决定每个目录下放多少文件。

## The Communication Protocols

HDFS Communication 建立在TCP/IP协议上。CLient 协议和DataNode协议被包含在一个RPC抽象里。NameNode不调用任何RPC，它只是响应client和DataNode的RPC请求。

## 健壮性

### 磁盘损坏，心跳，重新复制

DataNode会定期向NameNode发送心跳。如果一段时间（默认是10分钟，可配置）NameNode收不到DataNode的心跳，那么NameNode就会认为这个DataNode已经死亡。NameNode不会将IO的请求发送给已经死亡了的DataNode。另外，NameNode会持续的跟踪哪些需要被复制的Block，并复制他们直到达到他们的默认的复制数。

### 集群重新平衡

HDFS架构支持数据重新平衡计划。某个计划可能会在某个DataNode的剩余空间如果降低到了某个阈值之后，将这个DataNode的数据迁移到另外一个机器上。也有可能自动给读取比较热的数据创建多余的replica，并且在集群内重新rebalance。这些schema目前的都没应用。

### 数据准确性

存储在hdfs的block是有可能会被损坏的。原因可能是磁盘损坏，网络错误，或者软件bug。hdfs client会执行一个checksum的检查在hdfs文件内容上。当一个客户端创建了一个hdfs文件的时候，它会给文件的每个block计算一个checksum值，并存储在hdfs数据文件相同目录的一个隐藏文件里。当一个client收到这个数据的时候，它会计算这个数据内容的checksum值和穿过来的checksum file是否一致。如果不一致就重传。

### 元数据磁盘损坏

FsImage和EditLog是hdfs的中心数据结构。这些文件的损坏会导致hdfs整个不可用。因为这个原因，NameNode会被配置成支持多个FsImage和EditLog备份。对任意FsImage和EditLog的更新都会触发其他FsImage和EditLog的同步。这些同步操作会降低NameNode每秒可支撑的namespace transaction数。
但是这个降低是能忍受的，因为虽然hdfs天然是数据密集的，但是并不是元数据密集的。

还有一个选择去提升容错是使用多个NameNode，使用一个共享的存储空间在NFS上，或者用一个分布式的edit log（称为Journal）。

### Snapshots

Snapshots支持给hdfs上的某个数据创建一个快照。

## 数据组织

### 数据块

### replication pipelining

### 访问通道

1. [Java API](http://hadoop.apache.org/docs/current/api/)
2. [c](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/LibHdfs.html)
3. [REST API](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/WebHDFS.html)
4. [NFS gateway](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsNfsGateway.html)
5. [FS Shell](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html)
