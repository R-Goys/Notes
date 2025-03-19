# Kafka

## 1. 基本知识

### 1.1 **前置知识**

1. topic表示一个类型/业务的数据的组
2. 为方便扩展，提高吞吐率，一个topic分为多个partition。
3. 配合分区的设计，提出消费者组的概念，每个消费者并行消费，同时，一个分区的数据，只能由一个消费者组的一个消费者来消费。
4. 为提高可用性，每个partition都会有若干个副本，分为leader(当前正在执行的partition)和follower(备用的partition)，当leader挂掉是，被选中的follower就会成为新的leader，跟redis的集群中的主从比较类似。
5. kafka的部分数据存储在zookeeper中，记录了正在运行的节点以及每个分区的leader选举等信息，值得一提的是，在kafka2.8.0之后，kafka就可以不依赖于zookeeper，独立进行运行了。

如果通过客户端自动创建的话，partition默认只有一个，而我们可以在命令行输入`kafka-topics.sh --topic create-test --bootstrap-server kafka-1:9092 --partitions 11 --create`来创建一个有11个分区的topic，而如果是想要更新已有的topic的partition大小，应该将`--create`修改为`--alter`，如果是在docker环境中，也可以在进入容器之后输入`kafka-topics.sh`来查看命令的参数。其他的比如`--describe`用于查看topic的详细信息，`--list`查看所有主题。

