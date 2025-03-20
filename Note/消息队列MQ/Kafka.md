# Kafka

## 1. 基本知识

### 1.1 **前置知识**

1. topic表示一个类型/业务的数据的组
2. 为方便扩展，提高吞吐率，一个topic分为多个partition。
3. 配合分区的设计，提出消费者组的概念，每个消费者并行消费，同时，一个分区的数据，只能由一个消费者组的一个消费者来消费。
4. 为提高可用性，每个partition都会有若干个副本，分为leader(当前正在执行的partition)和follower(备用的partition)，当leader挂掉是，被选中的follower就会成为新的leader，跟redis的集群中的主从比较类似。
5. kafka的部分数据存储在zookeeper中，记录了正在运行的节点以及每个分区的leader选举等信息，值得一提的是，在kafka2.8.0之后，kafka就可以不依赖于zookeeper，独立进行运行了。

如果通过客户端自动创建的话，partition默认只有一个，而我们可以在命令行输入`kafka-topics.sh --topic create-test --bootstrap-server kafka-1:9092 --partitions 11 --create`来创建一个有11个分区的topic，而如果是想要更新已有的topic的partition大小，应该将`--create`修改为`--alter`，如果是在docker环境中，也可以在进入容器之后输入`kafka-topics.sh`来查看命令的参数。其他的比如`--describe`用于查看topic的详细信息，`--list`查看所有主题。

而我们也可以在命令行进行消费者和生产者的操作，生产者输入`kafka-console-producer.sh --bootstrap-server ip:port --topic [your topic]`，消费者则是将producer换成consumer即可，如下图所示：

![QQ_1742437593900](./assets/QQ_1742437593900.png)

除此之外，加上`--from-beginning`字段之后，consumer会加载所有的消息。

### 1.2 **生产者**

当生产者生产消息的时候，会经过producer->拦截器(可选)->序列化器->分区器->RecordAccumulator

分区器会将消息的数据进行分区，而对应的消息会被发到`RecordAccumulator`，此时还没有将数据发送，当数据积累到batch.size(默认16k)之后，Sender才会发送数据，当然，如果数据量比较少，滞留的时间超过`linger.ms`设定的时间，就会发送消息，但是默认`linger.ms`是0ms，也就是拿到消息就会立即发送数据，但实际可能因线程调度略有延迟。

当通过Sender发送数据时，会为每个Broker维护独立的请求队列。Kafka通过`max.in.flight.requests.per.connection`参数(默认5)控制每个Broker连接允许的最大未确认请求数。当某个Broker的`in-flight`请求数达到该限制时，针对该Broker的发送将暂停，直到收到对应的请求确认，之后才能继续发送新的请求。这个机制可以防止单个Broker堆积过多未确认请求，同时保证全局吞吐量，当然，如果等待的时间超过了`request.timeout.ms`(默认30s)，生产者则会认为请求失败，随后进行重试，当然应答有三个级别，0代表无需等待数据落盘就可以应答，1代表leader收到数据就可以应答，-1代表leader和follower全都同步完毕之后，才可以应答。.

#### **Go中的Kafka**

相较于kafka-go还是感觉sarama好用一点，虽然不支持context。

```go
func main() {
	brokers := []string{"localhost:29092", "localhost:29093", "localhost:29094"}
	topic := "cluster_test_topic"

	config := sarama.NewConfig()
	config.Producer.Return.Successes = true
	config.Producer.Partitioner = sarama.NewRoundRobinPartitioner

	producer, err := sarama.NewAsyncProducer(brokers, config)
	if err != nil {
		log.Fatalf("Failed to start Kafka async producer: %v", err)
	}
	defer producer.Close()

	// 监听成功和失败的消息
	go func() {
		for msg := range producer.Successes() {
			fmt.Printf("Message sent successfully: topic:%s partition:%d offset:%d\n", msg.Topic, msg.Partition, msg.Offset)
		}
	}()

	go func() {
		for err := range producer.Errors() {
			fmt.Printf("Failed to send message: %v\n", err)
		}
	}()
	wg := sync.WaitGroup{}
	wg.Add(1000000)
	// 发送消息
	for i := 0; i < 1000; i++ {
		go func() {
			for j := 0; j < 1000; j++ {
				msg := &sarama.ProducerMessage{
					Topic: topic,
					Value: sarama.StringEncoder(fmt.Sprintf("Hello Kafka!! My id is %d", i)),
				}
				producer.Input() <- msg
				defer wg.Done()
			}
		}()
	}
	wg.Wait()
}
```

这里是一个简单的go的生产者客户端，可以向kafka发送异步的发送消息(取决于`sarama.NewAsyncProducer`这一方法，如果需要同步调用，则需要做一些修改)，同时我们在启动生产者客户端的程序时，在终端上会显示我们的partition，而且可以明显的看见不同的消息，被分到了不同的partition上面，下面来细说一下分区的好处

#### **分区(Partition)**

Partition是Topic的子集，如果一个topic只设置在一个broker(机器)上面，则在传输巨大的数据量的时候，多台机器的负载不均匀，可能会导致broker压力过大，造成**性能瓶颈**，而且该Broker一旦故障，所有数据都会不可用，**可靠性低**。

所以引入了partition，一个topic可以具有多个partition，而每个partition可以存放在不同的broker上面，实现 **数据分布式存储**，这样，只要将消息均匀的发送到不同的partition上面，就能够实现broker的负载均衡，与此同时，默认情况下，如果消息带有key字段，那么kafka会根据这个key计算哈希值，将其放到合适的分区上面。

值得一提的是，和java客户端不同，go的Sarama客户端在不指定分区。并且不设定Key的时候，会采取轮询的策略来选择分区，而java客户端则是使用黏性分区来选择分区。

但是，我们在Sarama客户端，也可以自定义分区器，事实上，只需要自定义一个合乎规范的函数签名然后实现一个分区器的接口即可：
```go
// 自定义分区器
type MyPartitioner struct {
	topic          string
}

var _ sarama.Partitioner = (*MyPartitioner)(nil)

// Partition implements sarama.Partitioner.
func (m *MyPartitioner) Partition(message *sarama.ProducerMessage, numPartitions int32) (int32, error) {
	return 0, nil
}

// RequiresConsistency implements sarama.Partitioner.
func (m *MyPartitioner) RequiresConsistency() bool {
	return true
}
// 自定义构造函数
func NewMyPartitioner(topic string) sarama.Partitioner {
	return &MyPartitioner{
		topic: topic,
	}
}

func main() {
	...
	config.Producer.Partitioner = NewMyPartitioner
	...
}

```

另外，我们之前提到了拦截器，拦截器事实上就是在发送消息之前要处理的事情，我们可以在Sarama客户端中通过实现`func (m *MyInterceptor) OnSend(*sarama.ProducerMessage)`这个方法来实现自定义的拦截器！而使用这个拦截器，

具体操作如下：

```go
type MyInterceptor struct{}

func (m *MyInterceptor) OnSend(*sarama.ProducerMessage) {
	fmt.Println("OnSend")
}

var _ sarama.ProducerInterceptor = (*MyInterceptor)(nil)

func main() {
	...
	config.Producer.Interceptors = []sarama.ProducerInterceptor{
		&MyInterceptor{},
	}
	...
}
```



