---
layout: page
breadcrumb: true
title: Kafka面试题
category: kafka
categoryStr: Kafka面试题
tags: [kafka java 面试题]
keywords: [kafka java 面试题]
description: Kafka面试题，消息不丢失不重复，一致性
---


## 1、如何获取 topic 主题的列表
bin/kafka-topics.sh --list --zookeeper localhost:2181

## 2、生产者和消费者的命令行是什么？
生产者在主题上发布消息：

bin/kafka-console-producer.sh --broker-list 192.168.43.49:9092 --topicHello-Kafka

注意这里的 IP 是 server.properties 中的 listeners 的配置。接下来每个新行就是输入一条新消息。

消费者接受消息：

bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topicHello-Kafka --from-beginning

## 3、consumer 是推还是拉？
customer 应该从 brokes 拉取消息还是 brokers 将消息推送到 consumer，也就是 pull 还 push。在这方面，Kafka 遵循了一种大部分消息系统共同的传统的设计：producer 将消息推送到 broker，consumer 从broker 拉取消息。

push 模式，将消息推送到下游的 consumer。这样做有好处也有坏处：由 broker 决定消息推送的速率，对于不同消费速率的 consumer 就不太好处理了。消息系统都致力于让 consumer 以最大的速率最快速的消费消息，但不幸的是，push 模式下，当 broker 推送的速率远大于 consumer 消费的速率时，consumer 恐怕就要崩溃了。最终 Kafka 还是选取了传统的 pull 模式。

## 4、 kafka 维护消费状态跟踪的方法有什么？
大部分消息系统在 broker 端的维护消息被消费的记录：一个消息被分发到consumer 后 broker 就马上进行标记或者等待 customer 的通知后进行标记。这样也可以在消息在消费后立马就删除以减少空间占用。

那这样会不会有什么问题呢？

如果一条消息发送出去之后就立即被标记为消费过的，旦 consumer 处理消息时失败了（比如程序崩溃）消息就丢失了。为了解决这个问题，很多消息系统提供了另外一个个功能：当消息被发送出去之后仅仅被标记为已发送状态，当接到 consumer 已经消费成功的通知后才标记为已被消费的状态。这虽然解决了消息丢失的问题，但产生了新问题，

首先，如果 consumer处理消息成功了但是向 broker 发送响应时失败了，这条消息将被消费两次。第二个问题时，broker 必须维护每条消息的状态，并且每次都要先锁住消息然后更改状态然后释放锁。这样麻烦又来了，且不说要维护大量的状态数据，比如如果消息发送出去但没有收到消费成功的通知，这条消息将一直处于被锁定的状态。

Kafka 采用了不同的策略。Topic 被分成了若干分区，每个分区在同一时间只被一个 consumer 消费。这意味着每个分区被消费的消息在日志中的位置仅仅是一个简单的整数：offset。这样就很容易标记每个分区消费状态就很容易了，仅仅需要一个整数而已。这样消费状态的跟踪就很简单了。

这带来了另外一个好处：consumer 可以把 offset 调成一个较老的值，去重新消费老的消息。这对传统的消息系统来说看起来有些不可思议，但确实是非常有用的，谁规定了一条消息只能被消费一次呢？

如果对于分布式感兴趣的话，还可以了解一下我写的分布式消息中间件-RabbitMQ



同时你也可以对比一下Redis，他们的性能都还是可以的。你根据不同的应用场景去选择合适的内容做项目系统，也可以结合使用。



## 5、讲一下主从同步
Kafka允许topic的分区拥有若干副本，这个数量是可以配置的，你可以为每个topic配置副本的数量。Kafka会自动在每个个副本上备份数据，所以当一个节点down掉时数据依然是可用的。

Kafka的副本功能不是必须的，你可以配置只有一个副本，这样其实就相当于只有一份数据。

## 6、为什么需要消息系统，mysql 不能满足需求吗？
（1）解耦：

允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。

（2）冗余：

消息队列把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失风险。许多消息队列所采用的”插入-获取-删除”范式中，在把一个消息从队列中删除之前，需要你的处理系统明确的指出该消息已经被处理完毕，从而确保你的数据被安全的保存直到你使用完毕。

（3）扩展性：

因为消息队列解耦了你的处理过程，所以增大消息入队和处理的频率是很容易的，只要另外增加处理过程即可。

（4）灵活性 & 峰值处理能力：

在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。

（5）可恢复性：

系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。

（6）顺序保证：

在大多使用场景下，数据处理的顺序都很重要。大部分消息队列本来就是排序的，并且能保证数据会按照特定的顺序来处理。（Kafka 保证一个 Partition 内的消息的有序性）

（7）缓冲：

有助于控制和优化数据流经过系统的速度，解决生产消息和消费消息的处理速度不一致的情况。

（8）异步通信：

很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。

## 7、Zookeeper 对于 Kafka 的作用是什么？
Zookeeper 是一个开放源码的、高性能的协调服务，它用于 Kafka 的分布式应用。

Zookeeper 主要用于在集群中不同节点之间进行通信

在 Kafka 中，它被用于提交偏移量，因此如果节点在任何情况下都失败了，它都可以从之前提交的偏移量中获取除此之外，它还执行其他活动，如: leader 检测、分布式同步、配置管理、识别新节点何时离开或连接、集群、节点实时状态等等。

## 8、数据传输的事务定义有哪三种？

和 MQTT 的事务定义一样都是 3 种。

（1）最多一次: 消息不会被重复发送，最多被传输一次，但也有可能一次不传输

（2）最少一次: 消息不会被漏发送，最少被传输一次，但也有可能被重复传输.

（3）精确的一次（Exactly once）: 不会漏传输也不会重复传输,每个消息都传输被一次而且仅仅被传输一次，这是大家所期望的

## 9、Kafka 判断一个节点是否还活着有那两个条件？
（1）节点必须可以维护和 ZooKeeper 的连接，Zookeeper 通过心跳机制检查每个节点的连接

（2）如果节点是个 follower,他必须能及时的同步 leader 的写操作，延时不能太久

## 10、Kafka 与传统 MQ 消息系统之间有什么区别
(1).Kafka 持久化日志，这些日志可以被重复读取和无限期保留

(2).Kafka 是一个分布式系统：它以集群的方式运行，可以灵活伸缩，在内部通过复制数据提升容错能力和高可用性

(3).Kafka 支持实时的流式处理

## 11、消费者如何不自动提交偏移量，由应用提交？
将 enable.auto.commit 设为 false，然后在处理一批消息后在手动提交或者异步提交

即：
```java


//设置使用手动提交offset
$conf->set('enable.auto.commit', 'false');
$topicConf = new RdKafka\TopicConf();

// Set where to start consuming messages when there is no initial offset in
// offset store or the desired offset is out of range.
// 'smallest': start from the beginning
$topicConf->set('auto.offset.reset', 'smallest');

// Set the configuration to use for subscribed/assigned topics
$conf->setDefaultTopicConf($topicConf);

$consumer = new RdKafka\KafkaConsumer($conf);

// Subscribe to topic 'test'
$consumer->subscribe(['test']);

while (true) {
$message = $consumer->consume(120*1000);
switch ($message->err) {
case RD_KAFKA_RESP_ERR_NO_ERROR:
var_dump($message);
//消费完成后手动提交offset
//$consumer->commit($message);
break;
case RD_KAFKA_RESP_ERR__PARTITION_EOF:
echo "No more messages; will wait for more\n";
break;
case RD_KAFKA_RESP_ERR__TIMED_OUT:
echo "Timed out\n";
break;
default:
throw new \Exception($message->errstr(), $message->err);
break;
}
}
```
## 12、消费者故障，出现活锁问题如何解决？
出现“活锁”的情况，是它持续的发送心跳，但是没有处理。为了预防消费者在这种情况下一直持有分区，我们使用 max.poll.interval.ms 活跃检测机制。 在此基础上，如果你调用的 poll 的频率大于最大间隔，则客户端将主动地离开组，以便其他消费者接管该分区。 发生这种情况时，你会看到 offset 提交失败。这是一种安全机制，保障只有活动成员能够提交 offset。所以要留在组中，你必须持续调用 poll。

## 13、如何控制消费的位置
kafka 使用 seek(TopicPartition, long)指定新的消费位置。用于查找服务器保留的最早和最新的 offset 的特殊的方法也可用（seekToBeginning(Collection) 和seekToEnd(Collection)）

## 14、kafka 的高可用机制是什么？
多副本冗余的高可用机制

producer、broker 和 consumer 都会拥有多个

分区选举机制 、 消息确认机制

## 15、kafka 如何减少数据丢失
Kafka到底会不会丢数据(data loss)? 通常不会，但有些情况下的确有可能会发生。下面的参数配置及Best practice列表可以较好地保证数据的持久性(当然是trade-off，牺牲了吞吐量)。
```
• block.on.buffer.full = true

• acks = all

• retries = MAX_VALUE

• max.in.flight.requests.per.connection = 1

• 使用KafkaProducer.send(record, callback)

• callback逻辑中显式关闭producer：close(0)

• unclean.leader.election.enable=false

• replication.factor = 3

• min.insync.replicas = 2

• replication.factor > min.insync.replicas

• enable.auto.commit=false
```
• 消息处理完成之后再提交位移

## 16、kafka 如何不消费重复数据？比如扣款，我们不能重复的扣。
你拿个数据要写库，你先根据主键查一下，如果这数据都有了，你就别插入了，update 一下。

比如你是写 Redis，那没问题了，反正每次都是 set，天然幂等性。

比如你不是上面两个场景，那做的稍微复杂一点，你需要让生产者发送每条数据的时候，里面加一个全局唯一的 id，类似订单 id 之类的东西，然后你这里消费到了之后，先根据这个 id 去比如 Redis 里查一下，之前消费过吗？如果没有消费过，你就处理，然后这个 id 写 Redis。如果消费过了，那你就别处理了，保证别重复处理相同的消息即可。

比如基于数据库的唯一键来保证重复数据不会重复插入多条。因为有唯一键约束了，重复数据插入只会报错，不会导致数据库中出现脏数据。

解决:

1.幂等操作，重复消费不会产生问题
2.事务

对每个partitionID，产生一个uniqueID,.只有这个partition的数据被完全消费，才算成功，否则失败回滚。下次若重复执行，就skip

## 17、谈谈 Kafka 吞吐量为何如此高？
多分区、batch send、kafka Reator 网络模型、pagecache、sendfile 零拷贝、数据压缩

1>顺序读写


上图就展示了Kafka是如何写入数据的， 每一个Partition其实都是一个文件 ，收到消息后Kafka会把数据插入到文件末尾（虚框部分）。

这种方法有一个缺陷—— 没有办法删除数据 ，所以Kafka是不会删除数据的，它会把所有的数据都保留下来，每个消费者（Consumer）对每个Topic都有一个offset用来表示 读取到了第几条数据 。

2>Page Cache

为了优化读写性能，Kafka利用了操作系统本身的Page Cache，就是利用操作系统自身的内存而不是JVM空间内存。

3>零拷贝

零拷贝就是一种避免 CPU 将数据从一块存储拷贝到另外一块存储的技术。

linux操作系统 “零拷贝” 机制使用了sendfile方法， 允许操作系统将数据从Page Cache 直接发送到网络，只需要最后一步的copy操作将数据复制到 NIC 缓冲区， 这样避免重新复制数据 。示意图如下：


通过这种 “零拷贝” 的机制，Page Cache 结合 sendfile 方法，Kafka消费端的性能也大幅提升。这也是为什么有时候消费端在不断消费数据时，我们并没有看到磁盘io比较高，此刻正是操作系统缓存在提供数据。

4>分区分段+索引

Kafka的message是按topic分类存储的，topic中的数据又是按照一个一个的partition即分区存储到不同broker节点。每个partition对应了操作系统上的一个文件夹，partition实际上又是按照segment分段存储的。这也非常符合分布式系统分区分桶的设计思想。

5>批量读写

Kafka数据读写也是批量的而不是单条的。

6>批量压缩

如果每个消息都压缩，但是压缩率相对很低，所以Kafka使用了批量压缩，即将多个消息一起压缩而不是单个消息压缩