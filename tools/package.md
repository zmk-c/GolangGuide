# Go语言常用包

## 1.Golang自带包

| [自带常用包](https://studygolang.com/pkgdoc) | 说明                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| fmt                                          | 实现格式化的输入输出操作，其中的fmt.Printf()和fmt.Println()是开发者使用最为频繁的函数。 |
| io                                           | 实现了一系列非平台相关的IO相关接口和实现，比如提供了对os中系统相关的IO功能的封装。我们在进行流式读写（比如读写文件）时，通常会用到该包。 |
| bufio                                        | 它在io的基础上提供了缓存功能。在具备了缓存功能后， bufio可以比较方便地提供ReadLine之类的操作。 |
| strconv                                      | 提供字符串与基本数据类型互转的能力。                         |
| os                                           | 本包提供了对操作系统功能的非平台相关访问接口。接口为Unix风格。提供的功能包括文件操作、进程管理、信号和用户账号等。 |
| sync                                         | 它提供了基本的同步原语。在多个goroutine访问共享资源的时候，需要使用sync中提供的锁机制。 |
| flag                                         | 它提供命令行参数的规则定义和传入参数解析的功能。绝大部分的命令行程序都需要用到这个包。 |
| encoding/json                                | JSON目前广泛用做网络程序中的通信格式。本包提供了对JSON的基本支持，比如从一个对象序列化为JSON字符串，或者从JSON字符串反序列化出一个具体的对象等。 |
| http                                         | 通过http包，只需要数行代码，即可实现一个爬虫或者一个Web服务器，这在传统语言中是无法想象的。 |

## 2.常用第三方包

| 包                                     | 地址                                                         |
| -------------------------------------- | ------------------------------------------------------------ |
| 数据库操作                             | [github.com/jinzhu/gorm](https://github.com/jinzhu/gorm) [github.com/go-xorm/xorm](https://github.com/go-xorm/xorm) |
| 搜索es                                 | [github.com/olivere/elastic](https://github.com/olivere/elastic) |
| rocketmq操作                           | [github.com/apache/rocketmq-client-go/v2](https://github.com/apache/rocketmq-client-go/v2) |
| rabbitmq 操作                          | [github.com/streadway/amqp](https://github.com/streadway/amqp) |
| redis 操作                             | [github.com/go-redis/redis](https://github.com/go-redis/redis) |
| etcd 操作                              | [github.com/coreos/etcd/clientv3](https://pkg.go.dev/go.etcd.io/etcd/clientv3) |
| kafka                                  | https://github.com/Shopify/sarama https://github.com/bsm/sarama-cluster |
| excel 操作                             | [github.com/360EntSecGroup-Skylar/excelize](https://github.com/360EntSecGroup-Skylar/excelize) |
| ppt 操作                               | [golang.org/x/tools/cmd/present](https://golang.org/x/tools/cmd/present) |
| go-svg 操作                            | https://github.com/ajstarks/svgo                             |
| go 布隆过滤器实现                      | https://github.com/AndreasBriese/bbloom                      |
| json相关                               | https://github.com/bitly/go-simplejson                       |
| LRU Cache实现                          | [https://github.com/bluele/gcache ](https://github.com/bluele/gcache)https://github.com/hashicorp/golang-lru |
| go运行时函数替换                       | https://github.com/bouk/monkey                               |
| toml                                   | [https://github.com/toml-lang/toml ](https://github.com/toml-lang/toml)https://github.com/naoina/toml |
| yaml                                   | https://github.com/go-yaml/yaml                              |
| viper                                  | https://github.com/spf13/viper                               |
| go key/value存储                       | https://github.com/etcd-io/bbolt                             |
| 基于ringbuffer的无锁golang workpool    | https://github.com/Dai0522/workpool                          |
| 轻量级的协程池                         | https://github.com/ivpusic/grpool                            |
| 打印go的详细数据结构                   | https://github.com/davecgh/go-spew                           |
| 基于ringbuffer实现的队列               | https://github.com/eapache/queue                             |
| 拼音                                   | https://github.com/go-ego/gpy                                |
| 分词                                   | https://github.com/go-ego/gse                                |
| 搜索                                   | https://github.com/go-ego/riot                               |
| windows COM                            | https://github.com/go-ego/cedar                              |
| session                                | https://github.com/gorilla/sessions                          |
| 路由                                   | https://github.com/gorilla/mux                               |
| websocket                              | https://github.com/gorilla/websocket                         |
| Action handler                         | https://github.com/gorilla/handlers                          |
| csrf                                   | https://github.com/gorilla/csrf                              |
| context                                | https://github.com/gorilla/context                           |
| 过滤html标签                           | https://github.com/grokify/html-strip-tags-go                |
| 可配置的HTML标签过滤                   | https://github.com/microcosm-cc/bluemonday                   |
| 根据IP获取地理位置信息                 | https://github.com/ipipdotnet/ipdb-go                        |
| html转markdown                         | https://github.com/jaytaylor/html2text                       |
| goroutine 本地存储                     | https://github.com/jtolds/gls                                |
| 彩色输出                               | https://github.com/mgutz/ansi                                |
| 表格打印                               | https://github.com/olekukonko/tablewriter                    |
| reflect 更高效的反射API                | https://github.com/modern-go/reflect2                        |
| msgfmt (格式化字符串，将%更换为变量名) | https://github.com/modern-go/msgfmt                          |
| 可取消的goroutine                      | https://github.com/modern-go/concurrent                      |
| 深度拷贝                               | https://github.com/mohae/deepcopy                            |
| 安全的类型转换包                       | https://github.com/spf13/cast                                |
| 从文本中提取链接                       | https://github.com/mvdan/xurls                               |
| 字符串格式处理（驼峰转换）             | https://godoc.org/github.com/naoina/go-stringutil            |
| 文本diff实现                           | https://github.com/pmezard/go-difflib                        |
| uuid相关                               | https://github.com/satori/go.uuid https://github.com/snluu/uuid |
| 去除UTF编码中的BOM                     | https://github.com/ssor/bom                                  |
| 图片缩放                               | https://github.com/nfnt/resize                               |
| 生成 mock server                       | https://github.com/otokaze/mock                              |
| go 性能上报到influxdb                  | https://github.com/rcrowley/go-metrics                       |
| go zookeeper客户端                     | https://github.com/samuel/go-zookeeper                       |
| go thrift                              | https://github.com/samuel/go-thrift                          |
| MQTT 客户端                            | https://github.com/shirou/mqttcli                            |
| hbase                                  | https://github.com/tsuna/gohbase                             |
| go 性能上报到influxdb                  | https://github.com/rcrowley/go-metrics                       |
| go 性能上报到prometheus                | https://github.com/deathowl/go-metrics-prometheus            |
| ps utils                               | https://github.com/shirou/gopsutil                           |
| 小数处理                               | https://github.com/shopspring/decimal                        |
| 结构化日志处理(json)                   | https://github.com/sirupsen/logrus                           |
| 命令行程序框架 cli                     | https://github.com/urfave/cli                                |
| 命令行程序框架 cobra                   | https://github.com/spf13/cobra                               |
| grpc操作                               | [  google.golang.org/grpc ](google.golang.org/grpc )  [github.com/golang/protobuf/protoc-gen-go](google.golang.org/grpc ) |

## 3.分布式系统

| 包                                                        | 说明                                                         |
| --------------------------------------------------------- | ------------------------------------------------------------ |
| [celeriac](https://github.com/svcavallar/celeriac.v1)     | 用于在Go中添加支持以交互和监视Celery工作者，任务和事件的库。 |
| [consistent](https://github.com/buraksezer/consistent)    | 具有受限负载的一致哈希                                       |
| [dht](https://github.com/anacrolix/dht)                   | BitTorrent Kademlia DHT实施。                                |
| [digota](https://github.com/digota/digota)                | grpc电子商务微服务。                                         |
| [dot](https://github.com/dotchain/dot/)                   | 使用操作转换/ OT进行分布式同步。                             |
| [doublejump](https://github.com/edwingeng/doublejump)     | 改进后的Google的跳转一致性哈希。                             |
| [dragonboat](https://github.com/lni/dragonboat)           | Go中功能齐全的高性能多组Raft库。                             |
| [drmaa](https://github.com/dgruber/drmaa)                 | 基于DRMAA标准的集群调度程序的作业提交库。                    |
| [dynamolock](https://cirello.io/dynamolock)               | DynamoDB支持的分布式锁定实现。                               |
| [dynatomic](https://github.com/tylfin/dynatomic)          | 将DynamoDB用作原子计数器的库。                               |
| [emitter-io](https://github.com/emitter-io/emitter)       | 使用MQTT，Websockets和love构建的高性能，分布式，安全和低延迟的发布-订阅平台。 |
| [flowgraph](https://github.com/vectaport/flowgraph)       | 基于流的编程包。                                             |
| [gleam](https://github.com/chrislusf/gleam)               | 用纯围棋和Luajit快速和可扩展的分布式的map / reduce系统，具有Luajit的高性能结合Go的高并发，单独运行或分发。 |
| [glow](https://github.com/chrislusf/glow)                 | 易于使用的可扩展的分布式大数据处理，Map-Reduce，DAG执行，全部在纯Go中进行。 |
| [go-health](https://github.com/InVisionApp/go-health)     | health-用于在服务中启用异步依赖项运行状况检查的库。          |
| [go-jump](https://github.com/dgryski/go-jump)             | Google的“ Jump”一致性哈希函数的端口。                        |
| [go-kit](https://github.com/go-kit/kit)                   | 支持服务发现，负载平衡，可插拔传输，请求跟踪等的微服务工具包 |
| [go-sundheit](https://github.com/AppsFlyer/go-sundheit)   | 建立用于支持为golang服务定义异步服务运行状况检查的库。       |
| [gorpc](https://github.com/valyala/gorpc)                 | 简单，快速和可扩展的RPC库，可实现高负载。                    |
| [grpc-go](https://github.com/grpc/grpc-go)                | gRPC的Go语言实现。基于HTTP / 2的RPC。                        |
| [hprose](https://github.com/hprose/hprose-golang)         | 十分新颖的RPC库，现在支持25种以上的语言。                    |
| [jsonrpc](https://github.com/osamingo/jsonrpc)            | jsonrpc软件包可帮助实现JSON-RPC 2.0。                        |
| [jsonrpc](https://github.com/ybbus/jsonrpc)               | JSON-RPC 2.0 HTTP客户端实现。                                |
| [KrakenD](https://github.com/devopsfaith/krakend)         | 具有中间件的超高性能API网关框架。                            |
| [liftbridge](https://github.com/liftbridge-io/liftbridge) | NATS的轻量级，容错消息流。                                   |
| [micro](https://github.com/micro/micro)                   | 可插拔的microService工具箱和分布式系统平台。                 |
| [NATS](https://github.com/nats-io/gnatsd)                 | 用于微服务，IoT和云本机系统的轻量级高性能消息传递系统。      |
| [outboxer](https://github.com/italolelis/outboxer)        | Outboxer是一个实现库模式的go库。                             |
| [pglock](https://cirello.io/pglock)                       | PostgreSQL支持的分布式锁定实现。                             |
| [raft](https://github.com/hashicorp/raft)                 | HashiCorp的Raft共识协议的Golang实现。                        |
| [raft](https://github.com/coreos/etcd/tree/master/raft)   | 围棋实施筏一致协议，由CoreOS的。                             |
| [rain](https://github.com/cenkalti/rain)                  | BitTorrent客户端和库。                                       |
| [redis-lock](https://github.com/bsm/redislock)            | 使用Redis的简化分布式锁定实现。                              |
| [resgate](https://resgate.io/)                            | 用于构建REST，实时和RPC API的实时API网关，其中所有客户端都可以无缝同步。 |
| [ringpop-go](https://github.com/uber/ringpop-go)          | Go应用程序的可扩展，容错应用程序层分片。                     |
| [rpcx](https://github.com/smallnest/rpcx)                 | 分布式可插拔RPC服务框架，例如阿里巴巴Dubbo。                 |
| [sleuth](https://github.com/ursiform/sleuth)              | 用于在HTTP服务之间进行无主p2p自动发现和RPC的库（[ZeroMQ](https://github.com/zeromq/libzmq)）。 |
| [tendermint](https://github.com/tendermint/tendermint)    | 高性能中间件，用于使用Tendermint共识和区块链协议将以任何编程语言编写的状态机转换为拜占庭容错复制状态机。 |
| [torrent](https://github.com/anacrolix/torrent)           | BitTorrent客户端软件包。                                     |