# GolangGuide	制作人——无__忧👦 
汇总了一些关于Golang的资料提供查看，后续会继续补充完善，欢迎大家star~:smiley:

感谢：公众号【 脑子进煎鱼了】，【码农桃花源】，【小白debug】

|    常见面试题    |    深度解析    |    算法    |    数据库    |    操作系统    |    网络    |    容器化    |    消息队列    |    工具    |
| :--------------: | :------------: | :--------: | :----------: | :------------: | :--------: | :----------: | :------------: | :--------: |
| [📝](#常见面试题) | [📃](#深度分析) | [⌛️](#算法) | [💾](#数据库) | [💻](#操作系统) | [☁️](#网络) | [🎰](#容器化) | [✉️](#消息队列) | [🔧](#工具) |

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/GolangGuide.jpg" alt="go_monkey" style="zoom:50%;" />

## 常见面试题📝

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

## 深度解析📃

1. [Go语言深度解析之slice](golang/source_code/slice.md)
2. [Go语言深度解析之map](golang/source_code/map.md)
3. Go语言深度解析之channel
4. Go语言深度解析之context
5. [Go语言深度解析之unsafe](golang/source_code/unsafe.md)
6. Go语言深度解析之interface
7. Go语言深度解析之reflect
8. Go语言深度解析之内存分配
9. [Go语言深度解析之GC](golang/source_code/gc.md)
10. [Go语言深度解析之GPM调度器](golang/source_code/gpm.md)


## 算法 ⌛️ 

- [《剑指offer》解析](https://github.com/zmk-c/go-offer)
- [leetcode刷题顺序解析](https://github.com/zmk-c/leetcode)
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

## 操作系统💻

- [操作系统基础](os/os.md)

## 计算机网络☁️

- [计算机网络基础](network/network.md)
- [在B站看猫片被老板发现？不如按下F12学学HTTP](https://mp.weixin.qq.com/s/T41YBEmG4lkxokDLzRxVgA)
- [TCP粘包 数据包：我只是犯了每个数据包都会犯的错 |硬核图解](https://mp.weixin.qq.com/s/0H8WL6QeZ2VbO1hHPkn8Ug)
- [硬核图解！30张图带你搞懂！路由器，集线器，交换机，网桥，光猫有啥区别？](https://mp.weixin.qq.com/s/BJqp72EyEMahxi2XOfSrBQ)

## 容器化🎰

- ### gRPC

  - [gRPC及相关介绍](https://mp.weixin.qq.com/s/bbHqWqtmk_k3-X_1XEDEJw)
  - [gRPC Client and Server](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483721&idx=2&sn=5fab143b3cd50209fafc658aaba7c0e9&chksm=f9041414ce739d023611ac6ff38dbfe81d48591ab24ba37eefb3fe6cb121e89dd46fa2fbb1a9&cur_album_id=1383472721040064512&scene=189#rd)
  - [gRPC Streaming, Client and Server](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483721&idx=3&sn=b61db0379afd96e0149c279564d8efea&chksm=f9041414ce739d02c1554318a6e86942a0450266f27360913882860f24bc59268d315142f79b&cur_album_id=1383472721040064512&scene=189#rd)
  - [gRPC TLS 证书认证](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483719&idx=1&sn=34b3a6a6fd63106a4c369b3a0eaef330&chksm=f904141ace739d0cc5ecd1f40ed03688934a380fd5006ffd10947e45638277b0fcd197ab7ff8&scene=178&cur_album_id=1383472721040064512#rd)
  - [gRPC 基于 CA 的 TLS 证书认证](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483719&idx=2&sn=e8208b347f8a38c98fd4f5986bd0df4a&chksm=f904141ace739d0c7106280b5332832353cd1022204089ab46bab6c5b95c8687f34c13a755b3&scene=178&cur_album_id=1383472721040064512#rd)
  - [gRPC Unary and Stream interceptor](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247483718&idx=1&sn=ae0f6ea8111e7e9aeb152a247a333e68&chksm=f904141bce739d0dac96d1e3276fa141069681740a95c390b7c965f4381a14934075aa01d1c3&cur_album_id=1383472721040064512&scene=189#rd)
  - [总结：万字长文 | 从实践到原理，带你参透 gRPC](https://mp.weixin.qq.com/s?__biz=MzUxMDI4MDc1NA==&mid=2247484984&idx=1&sn=392e258f24aec08f58c84ccaba96b2ae&chksm=f9041365ce739a73054b01edcf31fdf3590fb403b1b48aa7dbeccc74c568e5b0e8a4e838c65e&scene=178&cur_album_id=1383472721040064512#rd)

- ### Docker

  - [docker](docker/docker.md)
  - [docker-compose](docker/docker-compose.md)

- ### Kubernetes

## 消息队列✉️

- ### RabbitMQ

## 工具🔧 

- ### [Git](tools/git.md)

- ### [Linux常用指令](tools/linux.md)

- ### [golang常用第三方库](tools/package.md)

  

