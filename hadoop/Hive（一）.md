

## Hive面试题整理（一）

### 1、Hive表关联查询，如何解决数据倾斜的问题？（☆☆☆☆☆）
&emsp; 1）倾斜原因：数据倾斜问题主要有以下几种：
1. 空值引发的数据倾斜-> 滤去空值
2. 不同数据类型引发的数据倾斜-> cast as
3. 不可拆分大文件引发的数据倾斜->采用bzip2和Zip等支持文件分割的压缩算法
4. 数据膨胀引发的数据倾斜
5. 表连接时引发的数据倾斜
6. 确实无法减少数据量引发的数据倾斜
&emsp; 2）解决方案 ：
&emsp; （1）参数调节：  
&emsp; &emsp; hive.map.aggr = true  
&emsp; &emsp; hive.groupby.skewindata=true  
&emsp; 有数据倾斜的时候进行负载均衡，当选项设定位true,生成的查询计划会有两个MR Job。第一个MR Job中，Map的输出结果集合会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是**相同的Group By Key有可能被分发到不同的Reduce**中，从而达到负载均衡的目的；  
第二个MR Job再根据预处理的数据结果按照Group By Key 分布到 Reduce 中（这个过程可以保证相同的 Group By Key 被分布到同一个Reduce中），最后完成最终的聚合操作。 


### 9、Hive内部表和外部表的区别？
1. 未被 external 修饰的是内部表，被 external 修饰的为外部表。
内部表数据由 Hive 自身管理，外部表数据由 HDFS 管理；
的表名创建一个文件夹，并将属于这个表的数据存放在这里）；
2. 删除内部表会直接删除元数据（metadata）及存储数据；删除外部表仅仅会删除元数据，HDFS 上的文件并不会被删除。    

### 11、所有的Hive任务都会有MapReduce的执行吗？
&emsp; 不是，从Hive0.10.0版本开始，对于简单的不需要聚合的类似SELECT <col> from <table> LIMIT n语句，不需要起MapReduce job，直接通过Fetch task获取数据。  


### 13、说说对Hive桶表的理解？
&emsp; 桶表是对数据进行哈希取值，然后放到不同文件中存储。  
&emsp; 数据加载到桶表时，会对字段取hash值，然后与桶的数量取模。把数据放到对应的文件中。物理上，每个桶就是表(或分区）目录里的一个文件，一个作业产生的桶(输出文件)和reduce任务个数相同。  
&emsp; 桶表专门用于抽样查询，是很专业性的，不是日常用来存储数据的表，需要抽样查询时，才创建和使用桶表。  


### 14、数据建模的模型
- 星型模型
- 雪花模型
- 星座模型










<style>
h1 {font-size: 2.5rem;}
h2 {font-size: 2rem;}
h3 {font-size: 1.8rem;}
p {font-size: 1.5rem;}
 ol, li {font-size: 1.5rem;} /* 设置有序列表和列表项的字体大小 */