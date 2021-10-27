# Golang常用到的库

| 库                                                     | 地址                                                         |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| 数据库操作 需要装驱动                                  | github.com/go-sql-driver/mysql                      github.com/jinzhu/gorm |
| 搜索es                                                 | [github.com/olivere/elastic](https://github.com/olivere/elastic) |
| rocketmq操作                                           | [github.com/apache/rocketmq-client-go/v2](https://github.com/apache/rocketmq-client-go/v2) |
| rabbitmq 操作                                          | [github.com/streadway/amqp](https://github.com/streadway/amqp) |
| redis 操作                                             | github.com/go-redis/redis                       github.com/gomodule/redigo |
| etcd 操作                                              | [github.com/coreos/etcd/clientv3](https://pkg.go.dev/go.etcd.io/etcd/clientv3) |
| kafka                                                  | https://github.com/Shopify/sarama https://github.com/bsm/sarama-cluste |
| pprof                                                  | github.com/gin-contrib/pprof                                 |
| testfiy测试库                                          | github.com/stretchr/testify                                  |
| excel 操作                                             | [github.com/360EntSecGroup-Skylar/excelize](https://github.com/360EntSecGroup-Skylar/excelize) |
| ppt 操作                                               | [golang.org/x/tools/cmd/present](https://golang.org/x/tools/cmd/present) |
| 可以操作任何数据的包 比方sting 转换类型啊 方便编写代码 | [github.com/Unknwon/com](github.com/Unknwon/com)             |
| 表单验证器                                             | github.com/go-playground/validator                           |
| json相关                                               | https://github.com/bitly/go-simplejson                       |
| LRU Cache实现                                          | [https://github.com/bluele/gcache ](https://github.com/bluele/gcache)https://github.com/hashicorp/golang-lru |
| go运行时函数替换                                       | https://github.com/bouk/monkey                               |
| 命令行程序框架 cli                                     | https://github.com/urfave/cli                                |
| 命令行程序框架 cobra                                   | https://github.com/spf13/cobra                               |
| 配置读取viper                                          | github.com/spf13/viper                                       |
| go key/value存储                                       | https://github.com/etcd-io/bbolt                             |
| golang的绘图库go-echart                                | github.com/go-echarts/go-echarts/v2                          |
| golangweb项目热更新  进入项目目录下：fresh启动         | github.com/pilu/fresh                                        |
| 轻量级的协程池                                         | https://github.com/ivpusic/grpool                            |
| 打印go的详细数据结构                                   | https://github.com/davecgh/go-spew                           |
| jwt鉴权                                                | github.com/dgrijalva/jwt-go                                  |
| 拼音                                                   | https://github.com/go-ego/gpy                                |
| 分词                                                   | https://github.com/go-ego/gse                                |
| 搜索                                                   | https://github.com/go-ego/riot                               |
| windows COM                                            | https://github.com/go-ego/cedar                              |
| session                                                | https://github.com/gorilla/sessions                          |
| 路由                                                   | https://github.com/gorilla/mux                               |
| websocket                                              | https://github.com/gorilla/websocket                         |
| Action handler                                         | https://github.com/gorilla/handlers                          |
| csrf                                                   | https://github.com/gorilla/csrf                              |
| context                                                | https://github.com/gorilla/context                           |
| 过滤html标签                                           | https://github.com/grokify/html-strip-tags-go                |
| 可配置的HTML标签过滤                                   | https://github.com/microcosm-cc/bluemonday                   |
| 根据IP获取地理位置信息                                 | https://github.com/ipipdotnet/ipdb-go                        |
| html转markdown                                         | https://github.com/jaytaylor/html2text                       |
| goroutine 本地存储                                     | https://github.com/jtolds/gls                                |
| 彩色输出                                               | https://github.com/mgutz/ansi                                |
| 表格打印                                               | https://github.com/olekukonko/tablewriter                    |
| reflect 更高效的反射API                                | https://github.com/modern-go/reflect2                        |
| msgfmt (格式化字符串，将%更换为变量名)                 | https://github.com/modern-go/msgfmt                          |
| 可取消的goroutine                                      | https://github.com/modern-go/concurrent                      |
| 深度拷贝                                               | https://github.com/mohae/deepcopy                            |
| 安全的类型转换包                                       | https://github.com/spf13/cast                                |
| 从文本中提取链接                                       | https://github.com/mvdan/xurls                               |
| 字符串格式处理（驼峰转换）                             | https://godoc.org/github.com/naoina/go-stringutil            |
| 文本diff实现                                           | https://github.com/pmezard/go-difflib                        |
| uuid相关                                               | https://github.com/satori/go.uuid https://github.com/snluu/uuid |
| 去除UTF编码中的BOM                                     | https://github.com/ssor/bom                                  |
| 图片缩放                                               | https://github.com/nfnt/resize                               |
| 生成 mock server                                       | https://github.com/otokaze/mock                              |
| go 性能上报到influxdb                                  | https://github.com/rcrowley/go-metrics                       |
| go zookeeper客户端                                     | https://github.com/samuel/go-zookeeper                       |
| go thrift                                              | https://github.com/samuel/go-thrift                          |
| go 性能上报到influxdb                                  | https://github.com/rcrowley/go-metrics                       |
| go 性能上报到prometheus                                | https://github.com/deathowl/go-metrics-prometheus            |
| ps utils                                               | https://github.com/shirou/gopsutil                           |
| 小数处理                                               | https://github.com/shopspring/decimal                        |
| 结构化日志处理(json)                                   | https://github.com/sirupsen/logrus                           |
| grpc操作                                               | google.golang.org/grpc              github.com/golang/protobuf/protoc-gen-go |
| ceph私有云库                                           | go get gopkg.in/amz.v1/aws                                                                             go get gopkg.in/amz.v1/s3 |
| 阿里云oss库                                            | go get github.com/aliyun/aliyun-oss-go-sdk/oss               |



