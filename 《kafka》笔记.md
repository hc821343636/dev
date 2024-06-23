# Kafka是什么
Kafka 是一个分布式流式处理平台。
流平台具有三个关键功能：
1. 消息队列：发布和订阅消息流，这个功能类似于消息队列，这也是 Kafka 也被归类为消息队列的原因。一个消息队列需要求：异步处理、流量控制和服务解耦。
2. 容错的持久方式存储记录消息流：Kafka 会把消息持久化到磁盘，有效避免了消息丢失的风险。
3. 流式处理平台： 在消息发布的时候进行处理，Kafka 提供了一个完整的流式处理类库。

# 队列模型
## 队列模型：早期的消息模型

![alt text](image.png)
使用队列（Queue）作为消息通信载体一条消息只能被一个消费者使用，未被消费的消息在队列中保留直到被消费或超时。  
队列模型存在的问题：需要将生产者产生的消息分发给多个消费者，并且每个消费者都能接收到完整的消息内容。

## 发布-订阅模型:Kafka 消息模型
![alt text](image-1.png)


发布订阅模型（Pub-Sub） 使用主题（Topic） 作为消息通信载体，类似于广播模式；发布者发布一条消息，该消息通过主题传递给所有的订阅者，在一条消息广播之后才订阅的用户则是收不到该条消息的。

在发布 - 订阅模型中，如果只有一个订阅者，那它和队列模型就基本是一样的了。所以说，发布 - 订阅模型在功能层面上是可以兼容队列模型的。

# Kafka核心概念
Kafka 将生产者发布的消息发送到 Topic（主题） 中，需要这些消息的消费者可以订阅这些 Topic（主题）
![alt text](image-2.png)
1. Producer（生产者） : 产生消息的一方。
2. Consumer（消费者） : 消费消息的一方。
3. Broker（代理） : 可以看作是一个独立的 Kafka 实例。多个 Kafka Broker 组成一个 Kafka Cluster。

同时，你一定也注意到每个 Broker 中又包含了 Topic 以及 Partition 这两个重要的概念：

● Topic（主题） : Producer 将消息发送到特定的主题，Consumer 通过订阅特定的 Topic(主题) 来消费消息。 

● Partition（分区） : Partition 属于 Topic 的一部分。一个 Topic 可以有多个 Partition ，并且同一 Topic 下的 Partition 可以分布在不同的 Broker 上，这也就表明一个 Topic 可以横跨多个 Broker 。

## Kafka 的多副本机制
 Kafka 为分区（Partition）引入了多副本（Replica）机制。分区（Partition）中的多个副本之间会有一个叫做 leader 的家伙，其他副本称为 follower。我们发送的消息会被发送到 leader 副本，然后 follower 副本才能从 leader 副本中拉取消息进行同步。

 生产者和消费者只与 leader 副本交互。其他副本只是 leader 副本的拷贝，它们的存在只是为了保证消息存储的安全性。当 leader 副本发生故障时会从 follower 中选举出一个 leader,但是 follower 中如果有和 leader 同步程度达不到要求的参加不了 leader 的竞选。

 ## Kafka 的多分区（Partition）以及多副本（Replica）机制有什么好处
 
 1. Kafka 通过给特定 Topic 指定多个 Partition, 而各个 Partition 可以分布在不同的 Broker 上, 这样便能提供比较好的并发能力（负载均衡）。
2. Partition 可以指定对应的 Replica 数, 这也极大地提高了消息存储的安全性, 提高了容灾能力，不过也相应的增加了所需要的存储空间。

## 重平衡机制

多个消费者实例共同组成的一个 Consumer Group（消费者组）通过 Group ID（字符串） 唯一标识 Consumer

为了保证消息处理的**有序性**和**避免重复消费**：Group.Topic 下的每个 Partition 只从属于 Consumer Group 中的一个 Consumer，不可能出现 Consumer Group 中的两个 Consumer 负责同一个 Partition。

一个消费组中的消费者和订阅的主题分区数量建议相等。
###  假如某个  Consumer Group  突然加入或者退出了一个 Consumer，会发生什么情况呢？
重平衡（Rebalance）。什么时候会重平衡？

● 订阅的 Topic 内的 Partition 发生变更

● 订阅的 Topic 发生变更

## 如何保证Kafka不丢失消息?

丢失消息有 3 种不同的情况，针对每一种情况有不同的解决方案。

###  生产者丢失消息的情况  
生产者(Producer) 调用send方法发送消息之后，消息可能因为网络问题并没有发送过去。所以，我们不能默认在调用 send() 方法发送消息之后消息消息发送成功了。  
我们要判断消息发送的结果。  但是，要注意的是 Producer 使用 send() 方法发送消息实际上是异步的操作，我们可以通过 get()方法获取调用结果，但是这样也让它变为了同步操作，示例代码如下：
```java
SendResult<String, Object> sendResult = kafkaTemplate.send(topic, o).get();
if (sendResult.getRecordMetadata() != null) {
  logger.info("生产者成功发送消息到" + sendResult.getProducerRecord().topic() + "-> " + sendRe
              sult.getProducerRecord().value().toString());
}
```
但是一般不推荐这么做！可以采用为其添加回调函数的形式，示例代码如下：

``` java 

ListenableFuture<SendResult<String, Object>> future = kafkaTemplate.send(topic, o);
future.addCallback(result -> logger.info("生产者成功发送消息到topic:{} partition:{}的消息", result.getRecordMetadata().topic(), result.getRecordMetadata().partition()),ex -> logger.error("生产者发送消失败，原因：{}", ex.getMessage()));
```
如果消息发送失败的话，我们检查失败的原因之后重新发送即可！

另外，这里推荐为 Producer 的 retries（重试次数）设置一个比较合理的值，一般是 3 ，但是为了保证消息不丢失的话一般会设置比较大一点。设置完成之后，当出现网络问题之后能够自动重试消息发送，避免消息丢失。另外，建议还要设置重试间隔，因为间隔太小的话重试的效果就不明显了，网络波动一次你 3 次一下子就重试完了.

### 消费者丢失消息的情况
消息在被追加到 Partition(分区)的时候都会分配一个特定的偏移量（offset）。offset 表示 Consumer 当前消费到的 Partition(分区)的所在的位置。Kafka 通过偏移量（offset）可以保证消息在分区内的顺序性。
![alt text](image-3.png)


当消费者拉取到了分区的某个消息之后，消费者会自动提交了 offset。自动提交的话会有一个问题，试想一下，当消费者刚拿到这个消息准备进行真正消费的时候，突然挂掉了，消息实际上并没有被消费，但是 offset 却被自动提交了。

对此可以选择：手动关闭自动提交 offset，每次在真正消费完消息之后之后再自己手动提交 offset。手动提交 offset 虽然可以解决这个问题，但也会带来消息重复消费的风险。
更好的解决办法：
- 幂等处理：
在应用层面确保消息处理的幂等性，即使同一条消息被处理多次，结果也是一样的。可以通过使用唯一的消息 ID 或者事务来实现幂等性。
事务性消费：

- Kafka 0.11.0 及以上版本支持事务性消费，可以在一个事务中读取、处理和提交 offset，这样可以保证消息处理和 offset 提交的原子性，避免重复消费和消息丢失的问题。


### Kafka 弄丢了消息  
Leader 副本所在的 Broker 突然挂掉，那么就要从 Fllower 副本重新选出一个  Leader ，但是  Leader 的数据还有一些没有被 Follower 副本的同步的话，就会造成消息丢失。  

解决办法：
- 设置 acks = all  
acks 的默认值即为1，代表我们的消息被leader副本接收之后就算被成功发送。当我们配置 acks = all 表示只有所有 ISR 列表的副本全部收到消息时，生产者才会接收到来自服务器的响应. 这种模式是最高级别的，也是最安全的，可以确保不止一个 Broker 接收到了消息. 该模式的延迟会很高.

- 设置 replication.factor >= 3  
这样就可以保证每个 Partition 至少有 3 个副本。虽然造成了数据冗余，但是带来了数据的安全性。


- min.insync.replicas > 1  
一般情况下我们还需要设置 min.insync.replicas> 1 ，这样配置代表消息至少要被写入到 2 个副本才算是被成功发送。min.insync.replicas 的默认值为 1 ，在实际生产中应尽量避免默认值 1。但是，为了保证整个 Kafka 服务的高可用性，你需要确保 replication.factor > min.insync.replicas 。为什么呢？设想一下假如两者相等的话，只要是有一个副本挂掉，整个分区就无法正常工作了。这明显违反高可用性！一般推荐设置成 replication.factor = min.insync.replicas + 1。

- 设置 unclean.leader.election.enable = false  
当 Leader 副本发生故障时就不会从 Follower 副本中和 Leader 同步程度达不到要求的副本中选择出 Leader ，这样降低了消息丢失的可能性。