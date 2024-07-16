<style>
h1 {font-size: 2.5rem;}
h2 {font-size: 2rem;}
h3 {font-size: 1.8rem;}
p {font-size: 1.5rem;}
 ol, li {font-size: 1.5rem;} /* 设置有序列表和列表项的字体大小 */
</style>
## Hadoop面试题（一）
Hadoop特点以及三大组件：
分布式存储：HDFS组件
分布式计算：MapReduce组件
分布式资源调度：Yarn组件
### 1、集群的最主要瓶颈  
&emsp; 磁盘IO  

### 2、请列出正常工作的Hadoop集群中Hadoop都分别需要启动哪些进程，它们的作用分别是什么?  
&emsp; 1）NameNode：它是hadoop中的主服务器，管理文件系统名称空间和对集群中存储的文件的访问，保存有metadate。  
&emsp; 2）SecondaryNameNode：它不是namenode的冗余守护进程，而是提供周期检查点和清理任务。帮助NN合并editslog，减少NN启动时间。  
&emsp; 3）DataNode：它负责管理连接到节点的存储（一个集群中可以有多个节点）。每个存储数据的节点运行一个datanode守护进程。  
&emsp; 4）ResourceManager（JobTracker）：JobTracker负责调度DataNode上的工作。每个DataNode有一个TaskTracker，它们执行实际工作。  
&emsp; 5）NodeManager：（TaskTracker）执行任务。  
&emsp; 6）DFSZKFailoverController：高可用时它负责监控NN的状态，并及时的把状态信息写入ZK。它通过一个独立线程周期性的调用NN上的一个特定接口来获取NN的健康状态。FC也有选择谁作为Active NN的权利，因为最多只有两个节点，目前选择策略还比较简单（先到先得，轮换）。  
&emsp; 7）JournalNode：高可用情况下存放namenode的editlog文件。  

### 3.基本成员
## Namenode：

接受客户端的读写服务、

管理文件系统命名空间、存储元数据信息—metadata。

- Metadata是存储在Namenode上的元数据信息，edits的文件记录对metadata的操作日志，记录了Metadata中的权限信息和文件系统目录树、文件包含哪些块、确定块到DataNode的映射、Block存放在哪些DataNode上(由DataNode启动时上报)、以及HDFS的每一次操作以及本次操作所影响的block（新增、移动、删除）。
- 合并操作：如果edits数据量太大，会触发合并操作，合并结果储存在fsImage文件中。先去fsImage中寻找，找不到再去edits中寻找。

    - 当Namenode启动时，它从硬盘中读取Edits和FsImage，将所有Edits中的事务作用在内存中的FsImage上，并将这个新版本的FsImage从内存中保存到本地磁盘上，然后删除旧的Edits，这个过程称为一个检查点(checkpoint)。

    - 合并参数（满足一个即可）：

    namenode.checkpoint.priod：默认3600s，即一小时

    namenode.checkpoint.txns 默认100w，即100w次事务

    定时检查：check.priod 默认60s

 NameNode将这些信息加载到内存并进行拼装，就成为了一个完整的元数据信息。Namenode在内存中保存着整个文件系统的命名空间和文件数据块映射(Blockmap)的映像。因而一个有4G内存的Namenode足够支撑大量的文件和目录。

 ## SecondaryNameNode
&emsp; 他的目的使帮助NameNode合并编辑日志，减少NameNode 启动时间。它不是NameNode的备份，但可以作为NameNode的备份，SecondNameNode的主要功能是帮助NameNode合并edits和fsimage文件，从而减少NameNode启动时间。



![alt text](pics/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98Pics/image.png)

首先生成一个名叫edits.new的文件用于记录合并过程中产生的日志信息；  

当触发到某一时机时（时间间隔达到1小时或Edits中的事务条数达到1百万）时SecondaryNamenode将edits文件、与fsimage文件使用http请求从NameNode上读取到SecondNamenode上；

将edits文件与fsimage进行合并操作，合并成一个fsimage.ckpt文件；

将生成的合并后的文件fsimage.ckpt文件转换到NameNode上；

将fsimage.ckpt在NameNode上变成fsimage文件替换NameNode上原有的fsimage文件，并将edits.new文件上变成edits文件替换NameNode上原有的edits文件。
