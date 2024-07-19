## Hadoop面试题（四）——YARN  

### 1、简述hadoop1与hadoop2 的架构异同  
&emsp; 1）加入了yarn解决了资源调度的问题。  
&emsp; 2）加入了对zookeeper的支持实现比较可靠的高可用。  
    
### 2、为什么会产生 yarn,它解决了什么问题，有什么优势？  
&emsp; 1）Yarn最主要的功能就是解决运行的用户程序与yarn框架完全解耦。  
&emsp; 2）Yarn上可以运行各种类型的分布式运算程序（mapreduce只是其中的一种），比如mapreduce、storm程序，spark程序……  

### 3、HDFS的数据压缩算法?（☆☆☆☆☆）  

&emsp; **企业开发用的比较多的是snappy**。  

### 4、MapReduce 2.0 容错性（☆☆☆☆☆）  
Map Task/Reduce  
&emsp; Task Task周期性向MRAppMaster汇报心跳；一旦Task挂掉，则MRAppMaster将为之重新申请资源，并运行之。最多重新运行次数可由用户设置，默认4次。 

### 5、 Yarn架构

![alt text](pics/Hadoop面试题Pics/YARN-Pics/image.png)

- ResourceManager 负责整个系统的资源管理和分配，
主要包括两个组件，即调度器（Scheduler）和应用程序管理器（Applications Manager）
    - 调度器接收来自ApplicationMaster的应用程序资源请求，把集群中的资源以“容器”的形式分配给提出申请的应用程序。
    - 应用程序管理器（Applications Manager）负责系统中所有应用程序的管理工作，主要包括应用程序提交、与调度器协商资源以启动、监控ApplicationMaster。
- ApplicationMaster： 
    - 用户提交的一个应用程序会对应于一个ApplicationMaster，它的主要功能与 RM 调度器协商以获得资源，资源以 Container 表示。
    - 监控所有的内部任务（Map任务或Reduce任务）状态，并在任务运行失败的时候重新为任务申请资源以重启任务。
    - 定时向ResourceManager发送“心跳”消息，报告资源的使用情况和应用的进度信息。
    - 作业完成时，像ResourceManager注销容器，执行周期完成。

- NodeManager： NodeManager 是每个节点上的资源和任务管理器.
    - 定期地向 RM 汇报本节点上的资源使用情况和各个 Container 的运行
状态；
    - 接收并处理来自 AM 的 Container 启动和停止请求。

- Container： Container 是 YARN 中的资源抽象，封装了各种资源。一个应
用程序会分配一个 Container，这个应用程序只能使用这个 Container 中描述的
资源。


### 6、 YARN 的任务提交流程
client 向 YARN 提交一个应用程序时：
1. 用户向 YARN 提交一个应用程序，并指定 ApplicationMaster 程序、启动
ApplicationMaster 的命令、用户程序。
2. RM 为这个应用程序分配第一个 Container，并与之对应的 NM 通讯，要求
它在这个 Container 中启动应用程序 ApplicationMaster。
3. ApplicationMaster 向 RM 注册，然后拆分为内部各个子任务，为各个内部
任务申请资源，并监控这些任务的运行，直到结束。
4. AM 采用轮询的方式向 RM 申请和领取资源。
5. RM 为 AM 分配资源，以 Container 形式返回。
6. AM 申请到资源后，便与之对应的 NM 通讯，要求 NM 启动任务。
7. NodeManager 为任务设置好运行环境，将任务启动命令写到一个脚本中，
并通过运行这个脚本启动任务
8. 各个任务向 AM 汇报自己的状态和进度，以便当任务失败时可以重启任务。
9. 应用程序完成后，ApplicationMaster 向 ResourceManager 注销并关闭自己

### 7、 YARN调度
1. FIFO Scheduler 先来先服务
2. Capacity Scheduler 能力调度器
3. Fair Scheduler 公平调度器
