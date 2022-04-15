# Kafka
### 一、消息队列
**概念**：消息队列（Message Queue, MQ）是一种应用间的通信方式，消息发送后可以立即返回，有消息系统来确保信息的可靠传递，消息发布者只管把消息发布到MQ中而不管谁来取，消息使用者只管从MQ中取出消息而不管谁发布的，这样发布者和消费者都不用知道对方的存在。

**应用场景**：
1. **应用解耦**：多应用间通过消息队列对同一消息进行处理，避免调用接口失败导致整个过程失败；（例如：用户上传头像，人脸识别系统对图片进行人脸识别）
   传统做法：服务器接收到图片后，图片上传系统立即调用人脸识别系统，调用完成后再返回成功。但是这种方法有缺点：
   - 人脸识别系统调用失败导致图片上传失败
   - 延迟高，需要人脸识别系统处理完成之后再返回给客户端
   - 图片上传系统和人脸识别系统之家相互调用，需要做耦合
   ![20220415105042](https://raw.githubusercontent.com/zmk-c/cloudImage/master/img/20220415105042.png)
   使用消息队列：客户端上传图片后，图片上传系统将图片信息批次写入MQ，直接返回成功。而人脸识别系统则定时从MQ中取数据，完成对新增图片的识别。
   ![20220415105444](https://raw.githubusercontent.com/zmk-c/cloudImage/master/img/20220415105444.png)
2. **异步处理**：多应用对消息队列中同一消息进行处理，应用间并发处理消息；（例如：用户注册，系统需要发送注册邮件并验证短信）
   传统的串行方式：新注册信息生成后，先发送邮件再发送验证短信。假设每个三个子系统注册信息写入、发送注册邮件、发送注册信息处理时间均为50ms，则串行方式花费150ms
   ![20220415104140](https://raw.githubusercontent.com/zmk-c/cloudImage/master/img/20220415104140.png)
   并行方式：新注册信息写入后，发送邮件和短信并行处理。并行方式花费100ms
   ![20220415104224](https://raw.githubusercontent.com/zmk-c/cloudImage/master/img/20220415104224.png)
   使用消息队列：在写入消息队列后立即返回成功给客户端，则总的响应时间依赖于写入MQ的时间，而写入MQ的时间很快基本可以忽略，因此总的处理时间相比串行提高了2倍，相比并行提高了1倍。
   ![20220415104802](https://raw.githubusercontent.com/zmk-c/cloudImage/master/img/20220415104802.png)
3. **限流削峰**：广泛应用于秒杀或抢购活动中，避免流量过大导致应用系统挂掉的情况；（例如：购物网站的秒杀活动，短时间访问量过大）
   使用消息队列：请求先入消息队列，而不是由业务处理系统直接处理，做了一次缓冲，极大地减少了业务处理系统的压力。
   ![20220415105713](https://raw.githubusercontent.com/zmk-c/cloudImage/master/img/20220415105713.png)

**消息队列的两种模式**
1. 点对点模式（point to point, queue）
   ![20220415110232](https://raw.githubusercontent.com/zmk-c/cloudImage/master/img/20220415110232.png)
   在点对点模式中，消息发送者生产消息发送到queue中，然后消费者接受者从queue中取出并消费。消息被消费后，queue中不再有存储，所以消费者不可能消费到已经被消费的消息。
2. 发布/订阅模式（publish/subscribe, topic）
   ![20220415110302](https://raw.githubusercontent.com/zmk-c/cloudImage/master/img/20220415110302.png)
   在发布/订阅模式中，每个消息可以有多个消费者

**其他消息队列与Kafka对比**
![20220415111506](https://raw.githubusercontent.com/zmk-c/cloudImage/master/img/20220415111506.png)

### 二、kafka架构及组件
![20220415111919](https://raw.githubusercontent.com/zmk-c/cloudImage/master/img/20220415111919.png)
**组件**
- **Producer**：消息的生产者，是消息的入口
- **Kafka cluster**：kafka集群，由一台或多台服务器组成
  - **Broker**：Broker是指部署了kafka实例的服务器节点，每个kafka集群内的broker都有一个不重复的编号
  - **Topic**：消息的主题，可以理解为消息的分类，kafka的数据就保存在topic。在==每个broker上可以创建多个topic==，实际应用中通常是一个业务线建立一个topic
  - **Partition**：Topic的分区，==每个topic可以有多个分区==，分区的作用是作为负载，提高kafka的吞吐量。==同一个topic在不同分区的数据是不重复的==，partition的表现形式就是一个个文件夹
  - **Replication**：==每一个分区都有多个副本==，当主分区（leader）故障时会选择一个副本（follower）成为leader。在kakfa中默认副本的最大数量是10个，且副本的数量不能大于broker的数量，follower和leader不能放置在同一机器上，同一机器对同一分区也只可能存放一个副本
- **Consumer**：消息的消费者，是消息的出口
  - **Consumer Group**：我们可以将多个消费者组成一个消费者组，在kafka的设计中==同一个分区的数据只能被消费者组中的某一个消费者消费==。同一个消费者组的消费者可以消费同一个topic中的不同分区的数据，这是为了提高kafka的吞吐量
  
**工作流程**
producer在写入数据的时候会把数据写入到leader中，不会直接写入follower。那么leader如何寻找？写入的流程是什么样子？如下图所示：
![20220415114758](https://raw.githubusercontent.com/zmk-c/cloudImage/master/img/20220415114758.png)
1. 生产者从kafka集群获取分区leader的信息
2. 生产者将信息发送给leader
3. leader将消息写入本地磁盘
4. follower从leader拉取消息数据
5. follower将消息写入本地磁盘后向leader发送ACK
6. leader收到所有的follower的ACK后向生产者发送ACK

**ACK应答机制**
producer在向kafka写入消息的时候，可以设置参数来确认kafka是否接收到数据，这个参数可设置的值为`0`，`1`，`all`
- 0代表producer往集群发送数据不需要等到集群确认的返回，不确保消息发送成功。安全性最低但效率最高
- 1代表producer往集群发送数据只需要leader应答就可以发送下一条，只确保leader发送成功
- all代表producer往集群发送数据需要所有的follower都完成从leader的同步才会发送下一条，确保leader发送成功和所有副本都完成备份。安全性最高，但是效率最低

==注意📢==：如果往不存在的topic写数据，kafka会自动创建topic，partition和replication的数量默认配置都是1

**Topic和数据日志**
topic是同一类别的消息记录的集合，在kakfa中，一个topic通常有多个订阅者，对于每个主题，kafka集群维护了一个分区数据日志文件
![20220415150540](https://raw.githubusercontent.com/zmk-c/cloudImage/master/img/20220415150540.png)
每个partition都是有一个有序且不可变的消息记录集合，当新的数据写入时，就会被追加到partition的末端。在每个partition中，每条消息都会被分配一个顺序的唯一标识，这个标识被称为`offset`，即偏移量
==注意📢==：kafka只保证在同一个partition内部消息是有序的，在不同partition之间并不能保证消息有序
kafka可以配置一个保留期限，用来标识日志会在kafka集群内保留多长时间。kafka集群会保留在保留期限内所有发布的消息，不管这些消息是否被消费过，过了保留期限，这些消息就会被清空。由于kafka会将数据进行持久化存储（即写到硬盘上），所以保留的数据大小可以设置为一个比较大的值

**选择partition的原则**
在kafka中，如果某个topic有多个partition，producer又怎么知道该将数据发往哪个partition呢？kafka中有几个原则：
1. partition在写入的时候可以指定需要写入的partition，如果有指定，则写入对应的partition
2. 如果没有指定partition，但设置了数据的key，则会根据key的值hash出一个partition
3. 如果既没有指定partition也没有设置key，则会采用轮训的方式，即每次取一小段时间的数据写入某个partition

**partition的结构**
partition在服务器上的表现形式为一个个的文件夹，每个partition的文件夹下面会有很多segment文件，每组segment文件又包含`.index文件`、`.log文件`、`.timeindex文件`这三种，其中==.log文件就是实际存储消息的地方==，而.index和.timeindex为索引文件，用于检索消息。

### 三、操作kafka
Docker安装kafka中间件
```bash
# 持久化zookeeper
docker run --name zookeeper -p 2181:2181  -v /home/docker/zookeeper/data:/data -v /etc/localtime:/etc/localtime -d wurstmeister/zookeeper
```

```bash
# 持久化kafka
docker run  --name kafka -t --restart=always -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=172.18.242.24:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://116.62.196.151:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -v /etc/localtime:/etc/localtime -v /home/docker/kafka:/kafka -d wurstmeister/kafka:latest
```

参数说明：
- -e KAFKA_BROKER_ID=0  在kafka集群中，每个kafka都有一个BROKER_ID来区分自己

- -e KAFKA_ZOOKEEPER_CONNECT=172.16.0.13:2181/kafka 配置zookeeper管理kafka的路径172.16.0.13:2181/kafka

- -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://172.16.0.13:9092  把kafka的地址端口注册给zookeeper，如果是远程访问要改成外网IP,类如Java程序访问出现无法连接。

- -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 配置kafka的监听端口

- -v /etc/localtime:/etc/localtime 容器时间同步虚拟机的时间

测试
- ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning

- ./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test < test.json