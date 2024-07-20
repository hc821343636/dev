

## Hive面试题整理（一）

### 1. Hive表关联查询，如何解决数据倾斜的问题？
&emsp; 1）倾斜原因：数据倾斜问题主要有以下几种：
1. 空值引发的数据倾斜-> 滤去空值
2. 不同数据类型引发的数据倾斜-> cast as hive.groupby.mapaggr，在mapper端做聚合
3. 不可拆分大文件引发的数据倾斜->采用bzip2和Zip等支持文件分割的压缩算法
4. 数据膨胀引发的数据倾斜（rollup）->hive.new.job.grouping.set.cardinality使得数据随机均匀地分不到不同的reducer上 配置的方式自动控制作业的拆解 为group by 默认为30个
5. 表连接时引发的数据倾斜(表连接的键存在倾斜)->将倾斜的数据存到分布式缓存中，分发到各个Map任务所在节点。在Map阶段完成join操作，即MapJoin，这避免了 Shuffle，从而避免了数据倾斜。


### 2. Hive内部表和外部表的区别？
1. 未被 external 修饰的是内部表，被 external 修饰的为外部表。
内部表数据由 Hive 自身管理，外部表数据由 HDFS 管理
2. 删除内部表会直接删除元数据（metadata）及存储数据；删除外部表仅仅会删除元数据，HDFS 上的文件并不会被删除。    

### 3、所有的Hive任务都会有MapReduce的执行吗？
&emsp; 不是，对于简单的不需要聚
合的类似SELECT col from table LIMIT n语句，直接通过Fetch task获取数据。  



### 4、数据建模的模型
- 星型模型
- 雪花模型
- 星座模型



### 5、Fetch抓取  
&emsp; Fetch抓取是指，Hive中对某些情况的查询可以不必使用MapReduce计算。例如：SELECT * FROM employees;在这种情况下，Hive可以简单地读取employee对应的存储目录下的文件，然后输出查询结果到控制台。   
&emsp; 在hive-default.xml.template文件中hive.fetch.task.conversion默认是more，老版本hive默认是minimal，该属性修改为more以后，在全局查找、字段查找、limit查找等都不走mapreduce。  

### 4、大表Join大表  
1）空KEY过滤 --直接过滤null值，防止大量null打到一个reduce上。  
2）空key转换 --有时候需要key为null的 value做计算，为了防止数据倾斜，可以将key设置为随机值使得数据随机均匀地不到不同的reducer上


### 5、Group By  
&emsp; 默认情况下，Map阶段同一Key数据分发给一个reduce，当一个key数据过大时就倾斜了。  
&emsp; 并不是所有的聚合操作都需要在Reduce端完成，很多聚合操作都可以先在Map端进行部分聚合，最后在Reduce端得出最终结果。  
1）开启Map端聚合参数设置  
&emsp; &emsp; （1）是否在Map端进行聚合，默认为True  
&emsp; &emsp; &emsp; hive.map.aggr = true  
&emsp; &emsp; （2）有数据倾斜的时候进行负载均衡（默认是false）     
&emsp; &emsp; &emsp; hive.groupby.skewindata = true  
&emsp; **当选项设定为 true，生成的查询计划会有两个MR Job**。第一个MR Job中，Map的输出结果会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是**相同的Group By Key有可能被分发到不同的Reduce中**，从而达到负载均衡的目的；第二个MR Job再根据预处理的数据结果按照Group By Key分布到Reduce中（这个过程可以保证相同的Group By Key被分布到同一个Reduce中），最后完成最终的聚合操作。   

### 6、Count(Distinct) 去重统计  
&emsp; 数据量大的情况下，由于COUNT DISTINCT操作需要用一个Reduce Task来完成，这一个Reduce需要处理的数据量太大，一般COUNT DISTINCT使用先GROUP BY再COUNT的方式替换 .

### 7、笛卡尔积  
&emsp; 尽量避免笛卡尔积，**Hive只能使用1个reducer来完成笛卡尔积**  
  



### 10、小文件进行合并  
&emsp; 在map执行前合并小文件，减少map数：CombineHiveInputFormat具有对小文件进行合并的功能（系统默认的格式）。HiveInputFormat没有对小文件合并功能。  











<style>
h1 {font-size: 2.5rem;}
h2 {font-size: 2rem;}
h3 {font-size: 1.8rem;}
p {font-size: 1.5rem;}
 ol, li {font-size: 1.5rem;} /* 设置有序列表和列表项的字体大小 */