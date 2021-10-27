# GolangGuide	制作人——无__忧👦 
汇总了一些关于Golang的资料提供查看，后续会继续补充完善，欢迎大家star~:smiley:

感谢：公众号【 脑子进煎鱼了】，【码农桃花源】，【小白debug】，【小林coding】

|    面试题    |    深度解析    |    算法    |    数据库    |    操作系统    |    网络    |    Gin框架    |    容器化    |    消息队列    |    工具    |
| :----------: | :------------: | :--------: | :----------: | :------------: | :--------: | :-----------: | :----------: | :------------: | :--------: |
| [📝](#面试题) | [📃](#深度分析) | [⌛](#算法) | [💾](#数据库) | [💻](#操作系统) | [☁](#网络) | [🔲](#Gin框架) | [🎰](#容器化) | [✉](#消息队列) | [🔧](#工具) |

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/GolangGuide.jpg" alt="go_monkey" style="zoom:50%;" />

## 面试题 📝

1. [Go 面试题： new 和 make 是什么，差异在哪？](https://mp.weixin.qq.com/s/tZg3zmESlLmefAWdTR96Tg)
2. [Go 群友提问：Goroutine 数量控制在多少合适，会影响 GC 和调度？](https://mp.weixin.qq.com/s/uWP2X6iFu7BtwjIv5H55vw)
3. [Go 群友提问：学习 defer 时很懵逼，这道不会做！](https://mp.weixin.qq.com/s/lELMqKho003h0gfKkZxhHQ)
4. [Go 面试题：Go interface 的一个 “坑” 及原理分析](https://mp.weixin.qq.com/s/vNACbdSDxC9S0LOAr7ngLQ)
5. [Go 群友提问：进程、线程都有 ID，为什么 Goroutine 没有 ID？](https://mp.weixin.qq.com/s/qFAtgpbAsHSPVLuo3PYIhg)
6. [ Go 面试题：GMP 模型，为什么要有 P？](https://mp.weixin.qq.com/s/an7dml9NLOhqOZjEGLdEEw)
7. [Go 面试题：Go 结构体是否可以比较，为什么？](https://mp.weixin.qq.com/s/HScH6nm3xf4POXVk774jUA)
8. [Go 面试题：单核 CPU，开两个 Goroutine，其中一个死循环，会怎么样？](https://mp.weixin.qq.com/s/h27GXmfGYVLHRG3Mu_8axw)
9. [Go 群友提问：你知道 Go 结构体和结构体指针调用有什么区别吗？](https://mp.weixin.qq.com/s/g-D_eVh-8JaIoRne09bJ3Q)
10. [跟读者聊 Goroutine 泄露的 N 种方法](https://mp.weixin.qq.com/s/ql01K1nOnEZpdbp--6EDYw)
11. [详解 Go 程序的启动流程，你知道 g0，m0 是什么吗？](https://mp.weixin.qq.com/s/YK-TD3bZGEgqC0j-8U6VkQ)
12. [用 Go struct 不能犯的一个低级错误！](https://mp.weixin.qq.com/s/K5B2ItkzOb4eCFLxZI5Wvw)
13. [嗯，你觉得 Go 在什么时候会抢占 P？](https://mp.weixin.qq.com/s/WAPogwLJ2BZvrquoKTQXzg)
14. [Go 面试官：什么是协程，协程和线程的区别和联系？](https://mp.weixin.qq.com/s/vW5n_JWa3I-Qopbx4TmIgQ)
15. [用 Go map 要注意这 1 个细节，避免依赖他！](https://mp.weixin.qq.com/s/MzAktbjNyZD0xRVTPRKHpw)
16. [为什么 Go map 和 slice 是非线性安全的？](https://mp.weixin.qq.com/s/TzHvDdtfp0FZ9y1ndqeCRw)
17. [一口气搞懂 Go sync.map 所有知识点](https://mp.weixin.qq.com/s/8aufz1IzElaYR43ccuwMyA)
18. [Go 面试官问我如何实现面向对象？](https://mp.weixin.qq.com/s/2x4Sajv7HkAjWFPe4oD96g)
19. [Go 是传值还是传引用？](https://mp.weixin.qq.com/s/qsxvfiyZfRCtgTymO9LBZQ)
20. [回答我，停止 Goroutine 有几种方法？](https://mp.weixin.qq.com/s/tN8Q1GRmphZyAuaHrkYFEg)

## 深度解析 📃

1. [Go语言深度解析之slice](source_code/slice.md)
2. [Go语言深度解析之map](source_code/map.md)
3. [Go语言深度解析之channel](source_code/channel.md)
4. [Go语言深度解析之context](source_code/context.md)
5. [Go语言深度解析之unsafe](source_code/unsafe.md)
6. [Go语言深度解析之interface](source_code/interface.md)
7. [Go语言深度解析之reflect](source_code/reflect.md)
8. [Go语言深度解析之内存分配](source_code/memory_distribution.md)
9. [Go语言深度解析之垃圾回收机制](source_code/gc.md)
10. [Go语言深度解析之GPM调度器](source_code/gmp.md)


## 算法 ⌛️ 

- [《剑指offer》](https://leetcode-cn.com/study-plan/lcof/)
- 常见共识算法
  - [Raft协议](consensus/raft.md)
  - [PBFT协议](consensus/pbft.md)
  - [Gossip协议](consensus/gossip.md)

## 数据库 💾 

- ### MySQL
  
  - [MySQL基础](mysql/base.md)
  - [MySQL数据库经典面试题解析](mysql/mysql100.md)
  - [MySQL InnoDB MVCC 机制的原理及实现](mysql/mysql_mvcc.md)
  - [为什么MySQL使用B+树做索引？](mysql/mysql-B+.md)
  
- ### Redis

  - [Redis基础](redis/base.md)
  - [Redis持久化原理及优化](redis/persistence.md)
  - [Redis中的主从复制原理](redis/master-slave.md)
  - [Redis中的内存淘汰策略](redis/memory.md)
  - [Redis缓存雪崩、击穿、穿透及解决方案](redis/solution.md)
  - [Redis常见问题](redis/faq.md)

- ### MongDB

## 操作系统 💻

- [图解操作系统-小林coding](os/os.pdf)

## 网络 ☁️

- [图解计算机网络-小林coding](network/network.pdf)

## Gin框架 🔲

- [Go Gin 系列一：Go 介绍与环境安装](https://mp.weixin.qq.coam/s?__biz=MzUxMDI4MDc1NA==&mid=2247483714&idx=1&sn=0b536199884cb45a1316c77998895baf&chksm=f904141fce739d0978e02147507dc29fadee2e19ac312d34a3190062ae40e62a490fc58df6ae&scene=178&cur_album_id=1383459655464337409#rd)
- [Go Gin 系列二：初始化项目及公共库](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483807&idx=1&sn=9c7aede4f675f2de49ddc08ab1a95a71&chksm=f90414c2ce739dd4b8711c0043286fba9744b8d9c86c75c7ac7750d28cd2fed43f749eb5de99&scene=178&cur_album_id=1383459655464337409#rd)
- [Go Gin 系列三：开发标签模块](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483807&idx=2&sn=513f8e5620db9cc37fea62fe6ff69796&chksm=f90414c2ce739dd4ccc217360b50618c085ec2327e4149dfbc1d136566ef6543dadd80b1e20e&scene=178&cur_album_id=1383459655464337409#rd)
- [Go Gin 系列四：开发文章模块](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483807&idx=3&sn=d24c23a03579f9ab662826c15174e3f4&chksm=f90414c2ce739dd42a4829099cc1229b51f4770d887f55a5995c584d0015d32fc8b9fe16d751&scene=178&cur_album_id=1383459655464337409#rd)
- [Go Gin 系列五：使用 JWT 进行身份校验](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483807&idx=4&sn=fae0d5ec098860038bb4de5c45d5d624&chksm=f90414c2ce739dd4b6fb2356afef5304057a49cbf527951000da107456ad07e87d1e69b32370&scene=178&cur_album_id=1383459655464337409#rd)
- [Go Gin 系列六：编写一个简单的文件日志](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483807&idx=5&sn=dbfc85b5a612a364f323de4703ae98ec&chksm=f90414c2ce739dd484a2c0583c424e59104809da9304ad8d23a85d9b4ff42f3e7d7f146d3930&scene=178&cur_album_id=1383459655464337409#rd)
- [Go Gin 系列七：优雅的重启服务](https://github.com/gravityblast/fresh)
- [Go Gin 系列八：为它加上Swagger](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483807&idx=7&sn=b73f0fd0ee14cdb43bc28ab6cb7c5644&chksm=f90414c2ce739dd43173eaec770dba45e04417a0849b676a0fa12af8e45a3db69e19eab3ab04&scene=178&cur_album_id=1383459655464337409#rd)
- [Go Gin 系列九：将Golang应用部署到Docker](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483807&idx=8&sn=b2827c18847397e6d1d37bfe49b2065f&chksm=f90414c2ce739dd4061203ea791b35846a3ecb0aa40680783676fb3e4a39115bda9abe14fbf0&scene=178&cur_album_id=1383459655464337409#rd)
- [Go Gin 系列十：定制 GORM Callbacks](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483819&idx=1&sn=90a68030b7d3f40b5ccfb9f91ce571d7&chksm=f90414f6ce739de092938728fe189e8d7b490aecaa19dddaa2c1c43eab971df29df0c37aa04a&scene=178&cur_album_id=1383459655464337409#rd)
- [Go Gin 系列十一：Cron定时任务](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483819&idx=2&sn=a85e39912a709d22dc3529ea9bdc3322&chksm=f90414f6ce739de02d20484b3368476a4ecf19c0b1e38f8e263703432af3c1776365d096c12e&scene=178&cur_album_id=1383459655464337409#rd)
- [Go Gin 系列十二：优化配置结构及实现图片上传](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483819&idx=3&sn=e76373b6bd530a552f08472d4987854e&chksm=f90414f6ce739de07dc82412e9c7e684a5058921253d1541b58e6ae205301be2fe782df9d6d6&scene=178&cur_album_id=1383459655464337409#rd)
- [Go Gin 系列十三：优化应用结构和实现Redis缓存](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483819&idx=4&sn=e6f85aa6196688198f3514e1efbbbeca&chksm=f90414f6ce739de0570a358c84023373a4021ed9e74bbaf7eec7c931e61a2c6c292bacae399d&scene=178&cur_album_id=1383459655464337409#rd)
- [Go Gin 系列十四：实现导出、导入 Excel](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483819&idx=5&sn=780affae40072df28ae6f6e4e226fdd8&chksm=f90414f6ce739de08b373523ea53b11575c64fd2db8ee04ee9237b9d24c93c8f6a0153918afd&scene=178&cur_album_id=1383459655464337409#rd)
- [Go Gin 系列十五：生成二维码、合并海报](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483819&idx=6&sn=57f8d9031249f61d039477b11d62612f&chksm=f90414f6ce739de0e0c36a5ad3784e2ebd82e7a8941805d162dbd660e54fe169cd87573b7f34&scene=178&cur_album_id=1383459655464337409#rd)
- [Go Gin 系列十六：在图片上绘制文字](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483819&idx=7&sn=1929b2cf09de3ec6222281def551a901&chksm=f90414f6ce739de04400958b1f4aebbd331715914b03efad26204ac8ba284d59d89f3af86099&scene=178&cur_album_id=1383459655464337409#rd)
- [Go Gin 系列十七：用Nginx部署Go应用](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483819&idx=8&sn=c64f86744121ba7f4c2f7b8539de8b7d&chksm=f90414f6ce739de012bbdef88a31e18332a21ea24ba773ef676d5d1ba7e3ab7ebed941aca5c1&scene=178&cur_album_id=1383459655464337409#rd)
- [番外：Golang 交叉编译](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483826&idx=1&sn=0d01b194dfba78f093134c69bab30e10&chksm=f90414efce739df9a3274ed894cd2e132b8616662f56178f3821478d9eddac812dca9d54688c&scene=178&cur_album_id=1383459655464337409#rd)
- [番外：请入门 Makefile](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483826&idx=2&sn=386ad6739a1a076b643ff526aff703e7&chksm=f90414efce739df9f6ab6a7d86518b96b5f46c08c8d6f4e29ce03e33ae46053a8efda7d080a4&scene=178&cur_album_id=1383459655464337409#rd)

## 容器化 🎰

- ### gRPC

  - [gRPC及相关介绍](https://mp.weixin.qq.com/s/bbHqWqtmk_k3-X_1XEDEJw)
  - [gRPC Client and Server](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483721&idx=2&sn=5fab143b3cd50209fafc658aaba7c0e9&chksm=f9041414ce739d023611ac6ff38dbfe81d48591ab24ba37eefb3fe6cb121e89dd46fa2fbb1a9&cur_album_id=1383472721040064512&scene=189#rd)
  - [gRPC Streaming, Client and Server](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483721&idx=3&sn=b61db0379afd96e0149c279564d8efea&chksm=f9041414ce739d02c1554318a6e86942a0450266f27360913882860f24bc59268d315142f79b&cur_album_id=1383472721040064512&scene=189#rd)
  - [gRPC TLS 证书认证](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483719&idx=1&sn=34b3a6a6fd63106a4c369b3a0eaef330&chksm=f904141ace739d0cc5ecd1f40ed03688934a380fd5006ffd10947e45638277b0fcd197ab7ff8&scene=178&cur_album_id=1383472721040064512#rd)
  - [gRPC 基于 CA 的 TLS 证书认证](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483719&idx=2&sn=e8208b347f8a38c98fd4f5986bd0df4a&chksm=f904141ace739d0c7106280b5332832353cd1022204089ab46bab6c5b95c8687f34c13a755b3&scene=178&cur_album_id=1383472721040064512#rd)
  - [gRPC Unary and Stream interceptor](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483718&idx=1&sn=ae0f6ea8111e7e9aeb152a247a333e68&chksm=f904141bce739d0dac96d1e3276fa141069681740a95c390b7c965f4381a14934075aa01d1c3&cur_album_id=1383472721040064512&scene=189#rd)
  - [让你的服务同时提供 HTTP 接口](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483718&idx=2&sn=0e592098eb5c1a837db12387fafe5f9c&chksm=f904141bce739d0d98ec188879258dd81c750a0404ba1a38b0c08610e3318b71fe65025e573c&cur_album_id=1383472721040064512&scene=189#rd)
  - [gRPC 对 RPC 方法做自定义认证](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483716&idx=1&sn=2b173c55cbe242cafda64a042b30669e&chksm=f9041419ce739d0fb6d4b210dd70962d96a72b1290d5138246c4b236b14ff1a57798f01969ae&cur_album_id=1383472721040064512&scene=189#rd)
  - [gRPC 超时控制](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483716&idx=2&sn=60a9d2e9c6a91c369aba0293e8bdb95b&chksm=f9041419ce739d0f0070b5e7bebeb112cd48ea86dbf9e36ad91ffe7943a944d85cf487ef0fb2&cur_album_id=1383472721040064512&scene=189#rd)
  - [gRPC + Zipkin 分布式链路追踪](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483716&idx=3&sn=71c2f616b4bed0af7a6a914e1ee2c1df&chksm=f9041419ce739d0fc3839eaffa7d7075f3be8cda92df241bd3e0e961d7a93b9eafdbf33d2335&cur_album_id=1383472721040064512&scene=189#rd)
  - [总结：万字长文 | 从实践到原理，带你参透 gRPC](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247484984&idx=1&sn=392e258f24aec08f58c84ccaba96b2ae&chksm=f9041365ce739a73054b01edcf31fdf3590fb403b1b48aa7dbeccc74c568e5b0e8a4e838c65e&scene=178&cur_album_id=1383472721040064512#rd)

- ### Docker

  - [docker](docker/docker.md)
  - [docker-compose](docker/docker-compose.md)

- ### Kubernetes

## 消息队列 ✉️

## 工具 🔧 

- ### [Git](tools/git.md)

- ### [Linux常用指令](tools/linux.md)

- ### [golang常用第三方库](tools/package.md)

  

