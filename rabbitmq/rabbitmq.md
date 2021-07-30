# RabbitMQ

### 简介

RabbitMQ 是轻量级的，易于在本地和云中部署。它支持多种消息传递协议。RabbitMQ 可以部署在分布式和联合配置中，以满足大规模、高可用性的需求。

RabbitMQ是一个消息代理：它接受并转发消息。你可以把它想象成一个邮局：当你把你想要邮寄的邮件放进一个邮箱时，你可以确定邮递员最终会把邮件送到你的收件人那里。在这个比喻中，RabbitMQ是一个邮箱、一个邮局和一个邮递员。

RabbitMQ和邮局的主要区别在于它不处理纸张，而是接受、存储和转发`二进制数据块`——**消息**。

#### 一些术语

- *生产者*仅意味着发送。发送消息的程序是生产者：

![img](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/producer.png)

- *队列*是位于RabbitMQ内部的邮箱的名称。尽管消息通过RabbitMQ和你的应用程序流动，但它们只能存储在队列中。队列只受主机内存和磁盘限制的限制，实际上它是一个大的消息缓冲区。许多生产者可以向一个队列发送消息，而许多消费者可以尝试从一个队列接收数据。以下是我们表示队列的方式：

![img](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/queue.png)

- *消费者*与接收具有相似的含义。消费者是一个主要等待接收消息的程序：

![img](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/consume.png)

请注意，生产者，消费者和代理（broker）不必位于同一主机上。实际上，在大多数应用程序中它们不是。一个应用程序既可以是生产者，也可以是消费者。

> 参考：[RabbitMQ教程](https://www.rabbitmq.com/getstarted.html)

### 安装RabbitMQ

可以安装带管理界面`manager`版本，这里选择安装`3.7.7-management`版本

```shell
docker pull rabbitmq:3.7.7-management
```

#### 启动rabbitmq

关于 RabbitMQ 需要注意的重要事情之一是它**根据它所谓的“节点名称”存储数据，默认为主机名**。这对于在 Docker 中的使用意味着我们应该为每个守护进程明确指定`-h`/`--hostname`以便我们不会获得随机主机名并可以跟踪我们的数据：

```shell
docker run -d --hostname my-rabbit -p 5672:5672 -p 15672:15672 --name some-rabbit  rabbitmq:3.7.7-management
#-p 指定服务运行的端口（5672：应用访问端口；15672：控制台Web端口号）；
```

还可以有其他参数选择

```shell
-e 指定环境变量
	RABBITMQ_DEFAULT_VHOST	默认虚拟机名
	RABBITMQ_DEFAULT_USER	默认的用户名
	RABBITMQ_DEFAULT_PASS	默认用户名的密码
```

这将启动一个侦听默认端口 5672 的 RabbitMQ 容器。如果您给它一分钟，然后执行`docker logs some-rabbit`，您将在输出中看到一个类似于以下内容的块：

```shell
=INFO REPORT==== 6-Jul-2015::20:47:02 ===
node           : rabbit@my-rabbit
home dir       : /var/lib/rabbitmq
config file(s) : /etc/rabbitmq/rabbitmq.configsh
cookie hash    : UoNOcDhfxW9uoZ92wh6BjA==
log            : tty
sasl log       : tty
database dir   : /var/lib/rabbitmq/mnesia/rabbit@my-rabbit
```

请注意`database dir`那里，特别是它在文件存储的末尾附加了我的“节点名称”。`/var/lib/rabbitmq`默认情况下，此图像构成一个卷。

下面登录web端页面，访问`http://IP地址:15672`，默认用户名/密码为`guest/guest`进行登录。

![image-20210714113401733](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/rabbitmqweb%E7%AB%AF%E6%9F%A5%E7%9C%8B.png)

### 案例

#### 01 ["Hello World!"](https://www.rabbitmq.com/tutorials/tutorial-one-python.html)

发送者

```go
package main

import (
	"github.com/streadway/amqp"
	"log"
)

func FailOnError(err error, msg string) {
	if err != nil {
		log.Fatalf("%s: %s", msg, err)
	}
}

func main() {
	// 1.链接RabbitMQ，建立连接
	//conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	conn, err := amqp.Dial("amqp://guest:guest@116.62.196.151:5672/") // 应用访问端口
	FailOnError(err, "Failed to connect to RabbitMQ")
	defer conn.Close()

	// 2.创建通道
	ch, err := conn.Channel()
	FailOnError(err, "Failed to open a channel")
	defer ch.Close()

	// 3.声明消息要发送到的队列
	q, err := ch.QueueDeclare(
		"hello", // name
		false,   // durable
		false,   // delete when unused
		false,   // exclusive
		false,   // no-wait
		nil,     // arguments
	)
	FailOnError(err, "Failed to declare a queue")

	body := "Hello World!"
	// 4.将消息发布到声明的队列
	err = ch.Publish(
		"",     // exchange
		q.Name, // routing key
		false,  // mandatory
		false,  // immediate
		amqp.Publishing{
			ContentType: "text/plain",
			Body:        []byte(body),
		})
	FailOnError(err, "Failed to publish a message")
	log.Printf(" [x] Sent %s", body)
}
```

接收者

```go
package main

import (
	"github.com/streadway/amqp"
	"log"
)

func failOnError(err error, msg string) {
	if err != nil {
		log.Fatalf("%s: %s", msg, err)
	}
}

func main() {
	// 1.建立连接
	//conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	conn, err := amqp.Dial("amqp://guest:guest@116.62.196.151:5672/")
	failOnError(err, "Failed to connect to RabbitMQ")
	defer conn.Close()

	// 2.获取管道
	ch, err := conn.Channel()
	failOnError(err, "Failed to open a channel")
	defer ch.Close()

	// 3.这里也声明队列。因为我们可能在发布者之前启动使用者，所以我们希望在尝试使用队列中的消息之前确保队列存在。
	q, err := ch.QueueDeclare(
		"hello", // name
		false,   // durable
		false,   // delete when unused
		false,   // exclusive
		false,   // no-wait
		nil,     // arguments
	)
	failOnError(err, "Failed to declare a queue")

	// 4.进行消费
	msgs, err := ch.Consume(
		q.Name, // queue
		"",     // consumer
		true,   // auto-ack
		false,  // exclusive
		false,  // no-local
		false,  // no-wait
		nil,    // args
	)
	failOnError(err, "Failed to register a consumer")

	forever := make(chan bool)

	go func() {
		for d := range msgs {
			log.Printf("Received a message: %s", d.Body)
		}
	}()

	log.Printf(" [*] Waiting for messages. To exit press CTRL+C")
	<-forever
}
```

结果展示

![image-20210714151625467](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/send.png)

![image-20210714151643204](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/receive.png)

#### 02 ["Work queues"](https://www.rabbitmq.com/tutorials/tutorial-two-python.html)

![img](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E5%B7%A5%E4%BD%9C%E9%98%9F%E5%88%97.png)

在第一个教程中，我们编写程序从命名的队列发送和接收消息。在这一节中，我们将创建一个**工作队列**，该队列将用于在多个工人之间分配耗时的任务。

工作队列（又称任务队列）的**主要思想是避免立即执行某些资源密集型任务并且不得不等待这些任务完成**。相反，我们安排任务异步地同时或在当前任务之后完成。我们将任务封装为消息并将其发送到队列，在后台运行的工作进程将取出消息并最终执行任务。当你运行多个工作进程时，任务将在他们之间共享。

##### 准备工作

在本教程的上一部分，我们发送了一条包含“ Hello World！”的消息。现在，我们将发送代表复杂任务的字符串。我们没有实际的任务，例如调整图像大小或渲染pdf文件，所以我们通过借助`time.Sleep`函数模拟一些比较耗时的任务。我们会将一些包含`.`的字符串封装为消息发送到队列中，其中每有一个`.`就表示需要耗费1秒钟的工作，例如，`hello...`表示一个将花费三秒钟的假任务。

我们将稍微修改上一个示例中的`send.go`代码，以允许从命令行发送任意消息。该程序会将任务安排到我们的工作队列中，因此我们将其命名为`new_task.go`

```go
...
// 从参数中获取要发送的消息正文 控制台输入（更改处）
body := bodyFrom(os.Args)
// 4.将消息发布到声明的队列
err = ch.Publish(
    "",     // exchange
    q.Name, // routing key
    false,  // mandatory
    false,  // immediate
    amqp.Publishing{
        DeliveryMode: amqp.Persistent,
        ContentType: "text/plain",
        Body: []byte(body),
    })

func bodyFrom(args []string) string {
	var s string
	if len(args)<2 || args[1]==""{
		s = "hello"
	}else {
		s = strings.Join(args[1:]," ")
	}
	return s
}
```

我们以前的`receive.go`程序也需要进行一些更改：它需要为消息正文中出现的每个`.`伪造一秒钟的工作。它将从队列中弹出消息并执行任务，因此我们将其称为`worker.go`：

```go
...
go func() {
    for d := range msgs {
        log.Printf("Received a message: %s", d.Body)
        // 统计d.Body中的.个数（更改处）
        dot_count := bytes.Count(d.Body,[]byte("."))
        t := time.Duration(dot_count)
        time.Sleep(t * time.Second) // 模拟耗时任务 一个.算一秒钟
        log.Printf("Done")
    }
}()
```

##### 循环调度

使用任务队列的优点之一是能够轻松并行化工作。如果我们的工作正在积压，我们可以增加更多的工人，这样就可以轻松扩展。

首先，让我们尝试同时运行两个`worker.go`脚本。它们都将从队列中获取消息，但是究竟是怎样呢？让我们来看看。

你需要打开三个控制台。其中两个将运行`worker.go`脚本。这些控制台将成为我们的两个消费者——C1和C2。

```bash
# shell 1
go run worker.go
# => [*] Waiting for messages. To exit press CTRL+C
# shell 2
go run worker.go
# => [*] Waiting for messages. To exit press CTRL+C
```

在第三个控制台中，我们将发布新任务。启动消费者之后，你可以发布一些消息：

```bash
# shell 3
go run new_task.go msg1.
go run new_task.go msg2..
go run new_task.go msg3...
go run new_task.go msg4....
go run new_task.go msg5.....
```

然后我们在`shell1`和 `shell2` 两个窗口看到如下输出结果了：

```bash
# shell 1
go run worker.go
# => [*] Waiting for messages. To exit press CTRL+C
# => [x] Received a message: msg1.
# => [x] Received a message: msg3...
# => [x] Received a message: msg5.....
# shell 2
go run worker.go
# => [*] Waiting for messages. To exit press CTRL+C
# => [x] Received a message: msg2..
# => [x] Received a message: msg4....
```

默认情况下，RabbitMQ将按顺序将每个消息发送给下一个消费者。平均而言，每个消费者都会收到相同数量的消息。这种分发消息的方式称为`轮询`。使用三个或者更多worker试一下。

##### 消息确认

worker完成任务可能需要耗费几秒钟，如果一个worker在任务执行过程中**宕机了该怎么办呢**？我们当前的代码中，**RabbitMQ一旦向消费者传递了一条消息，便立即将其标记为删除**。在这种情况下，如果你终止一个worker那么你就可能会丢失这个任务，我们还将**丢失所有已经交付给这个worker的尚未处理的消息**。

我们不想丢失任何任务，如果一个worker意外宕机了，那么我们希望将任务交付给其他worker来处理。

为了确保消息永不丢失，RabbitMQ支持`消息确认`。消费者发送回一个确认（acknowledgement），以告知RabbitMQ已经接收，处理了特定的消息，并且RabbitMQ可以自由删除它。

如果消费者在不发送确认的情况下死亡（其通道已关闭，连接已关闭或TCP连接丢失），RabbitMQ将了解消息未完全处理，并将对其**重新排队**。如果同时有其他消费者在线，它将很快将其**重新分发给另一个消费者**。这样，您可以确保即使worker偶尔死亡也不会丢失任何消息。

没有任何消息超时；RabbitMQ将在消费者死亡时重新传递消息。即使处理一条消息需要很长时间也没关系。

在本教程中，我们将使用手动消息确认，方法是为“`auto-ack`”参数传递一个false，然后在完成任务后，使用`d.Ack(false)`从worker发送一个正确的确认（这将确认一次传递）。

```go
// 4.进行消费
msgs, err := ch.Consume(
    q.Name, // queue
    "",     // consumer
    false,   // auto-ack 更改为false 取消自动确认
    false,  // exclusive
    false,  // no-local
    false,  // no-wait
    nil,    // args
)
failOnError(err, "Failed to register a consumer")

forever := make(chan bool)

go func() {
    for d := range msgs {
        log.Printf("Received a message: %s", d.Body)
        // 统计d.Body中的.个数
        dot_count := bytes.Count(d.Body,[]byte("."))
        t := time.Duration(dot_count)
        time.Sleep(t * time.Second) // 模拟耗时任务 一个.算一秒钟
        log.Printf("Done")
        d.Ack(false) // 手动传递消息确认
    }
}()
```

使用这段代码，我们可以确保即使你在处理消息时使用CTRL+C杀死一个worker，也不会丢失任何内容。在worker死后不久，所有未确认的消息都将被重新发送。

**注意：**消息确认必须在接收消息的**同一通道**（Channel）上发送。尝试使用不同的通道（Channel）进行消息确认将导致`通道级协议异常`。

##### 公平分发

你可能已经注意到调度仍然不能完全按照我们的要求工作。例如，在一个有两个worker的情况下，当**所有的奇数消息都是重消息而偶数消息都是轻消息时，一个worker将持续忙碌，而另一个worker几乎不做任何工作**。RabbitMQ对此一无所知，仍然会均匀地发送消息。

这是因为RabbitMQ只是在消息进入队列时发送消息。它不考虑消费者未确认消息的数量。只是盲目地向消费者发送信息。

![img](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E5%85%AC%E5%B9%B3%E5%88%86%E5%8F%91.png)

为了避免这种情况，我们可以将`预取计数设置为1`。这告诉RabbitMQ不要一次向一个worker发出多个消息。或者，换句话说，在处理并确认前一条消息之前，不要向worker发送新消息。相反，它将把它发送给下一个不忙的worker。

```go
err = ch.Qos(
  1,     // prefetch count
  0,     // prefetch size
  false, // global
)
```

最后完整的代码如下：

`new_task.go`

```go
package main

import (
	"github.com/streadway/amqp"
	"log"
	"os"
	"strings"
)

func FailOnError(err error, msg string) {
	if err != nil {
		log.Fatalf("%s: %s", msg, err)
	}
}

func main() {
	// 1.链接RabbitMQ，建立连接
	//conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	conn, err := amqp.Dial("amqp://guest:guest@116.62.196.151:5672/")
	FailOnError(err, "Failed to connect to RabbitMQ")
	defer conn.Close()

	// 2.创建通道
	ch, err := conn.Channel()
	FailOnError(err, "Failed to open a channel")
	defer ch.Close()

	// 3.声明消息要发送到的队列
	q, err := ch.QueueDeclare(
		"hello", // name
		false,   // durable
		false,   // delete when unused
		false,   // exclusive
		false,   // no-wait
		nil,     // arguments
	)
	FailOnError(err, "Failed to declare a queue")

	// 从参数中获取要发送的消息正文 控制台输入
	body := bodyFrom(os.Args)
	// 4.将消息发布到声明的队列
	err = ch.Publish(
		"",     // exchange
		q.Name, // routing key
		false,  // mandatory
		false,  // immediate
		amqp.Publishing{
			DeliveryMode: amqp.Persistent,
			ContentType: "text/plain",
			Body: []byte(body),
		})
	FailOnError(err, "Failed to publish a message")
	log.Printf(" [x] Sent %s", body)
}

func bodyFrom(args []string) string {
	var s string
	if len(args)<2 || args[1]==""{
		s = "hello"
	}else {
		s = strings.Join(args[1:]," ")
	}
	return s
}
```

`worker.go`

```go
package main

import (
	"bytes"
	"fmt"
	"github.com/streadway/amqp"
	"log"
	"time"
)

func failOnError(err error, msg string) {
	if err != nil {
		log.Fatalf("%s: %s", msg, err)
	}
}

func main() {
	// 1.建立连接
	//conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	conn, err := amqp.Dial("amqp://guest:guest@116.62.196.151:5672/")
	failOnError(err, "Failed to connect to RabbitMQ")
	defer conn.Close()

	// 2.获取管道
	ch, err := conn.Channel()
	failOnError(err, "Failed to open a channel")
	defer ch.Close()

	// 3.这里也声明队列。因为我们可能在发布者之前启动使用者，所以我们希望在尝试使用队列中的消息之前确保队列存在。
	q, err := ch.QueueDeclare(
		"hello", // name
		false,   // durable
		false,   // delete when unused
		false,   // exclusive
		false,   // no-wait
		nil,     // arguments
	)
	failOnError(err, "Failed to declare a queue")

	err = ch.Qos(
		1,     // prefetch count
		0,     // prefetch size
		false, // global
	)
	if err != nil {
		fmt.Printf("ch.Qos() failed, err:%v\n", err)
		return
	}

	// 4.进行消费
	msgs, err := ch.Consume(
		q.Name, // queue
		"",     // consumer
		false,   // auto-ack 更改为false 取消自动确认
		false,  // exclusive
		false,  // no-local
		false,  // no-wait
		nil,    // args
	)
	failOnError(err, "Failed to register a consumer")

	forever := make(chan bool)

	go func() {
		for d := range msgs {
			log.Printf("Received a message: %s", d.Body)
			// 统计d.Body中的.个数
			dot_count := bytes.Count(d.Body,[]byte("."))
			t := time.Duration(dot_count)
			time.Sleep(t * time.Second) // 模拟耗时任务 一个.算一秒钟
			log.Printf("Done")
			d.Ack(false) // 手动传递消息确认
		}
	}()

	log.Printf(" [*] Waiting for messages. To exit press CTRL+C")
	<-forever
}
```

#### 03 "Publish/Subscribe"

在02中，我们创建了一个工作队列。工作队列背后的假设是每个任务只传递给一个工人。在这一部分中，我们将做一些完全不同的事情——我们将向多个消费者传递一个消息。这就是所谓的“订阅/发布模式”。

为了说明这种模式，我们将构建一个简单的日志系统。它将由两个程序组成——**第一个程序将发出日志消息**，**第二个程序将接收并打印它们**。

在我们的日志系统中，每一个运行的接收器程序副本都会收到消息。这样，我们就可以运行一个接收器并将日志定向到磁盘；同时，我们还可以运行另一个接收器并在屏幕上查看日志。

本质上，已发布的日志消息将被广播到所有接收者。

##### Exchanges（交换器）



