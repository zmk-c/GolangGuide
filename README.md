# GolangGuide	制作人——无__忧👦 
总结了golang常见的面试题，汇总了一些资料提供查看，后续会继续补充完善，欢迎大家star~:smiley:

|    基础归纳    |    源码分析    |    常见面试题    |    算法    |    数据库    |    操作系统    |    计算机网络    |    工具    |
| :------------: | :------------: | :--------------: | :--------: | :----------: | :------------: | :--------------: | :--------: |
| [📓](#基础归纳) | [📃](#源码分析) | [🕕](#常见面试题) | [⌛️](#算法) | [💾](#数据库) | [💻](#操作系统) | [☁️](#计算机网络) | [🔧](#工具) |

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/GolangGuide.jpg" alt="go_monkey" style="zoom:50%;" />

## 基础归纳📓 

- [Go语言基础知识细节](golang/base/base.md)
- [Go语言常用包](golang/package/package.md)

## 源码分析📃

1. [Go语言源码分析之slice](golang/source_code/slice.md)
2. [Go语言源码分析之map](golang/source_code/map.md)
3. Go语言源码分析之channel
4. Go语言源码分析之context
5. [Go语言源码分析之unsafe](golang/source_code/unsafe.md)
6. Go语言源码分析之reflect
7. Go语言源码分析之interface
8. Go语言源码分析之内存分配
10. [Go语言源码分析之GPM调度器](golang/source_code/gpm.md)
11. Go语言源码分析之GC

## 常见面试题🕕 

1. [字符串转换成byte数组，会发生内存拷贝吗?有没有什么办法可以在转换时不发生拷贝呢?](golang/faq/1.md)
2. [能说说uintptr和unsafe.Pointer的区别吗?](golang/faq/2.md)
3. [拷贝大切片一定比小切片代价大吗?](golang/faq/3.md)
4. [知道Golang的内存逃逸吗?什么情况下回发生内存逃逸?](golang/faq/4.md)
5. [怎么避免逃逸分析?](golang/faq/5.md)
6. [reflect (反射包)如何获取字段tag? 为什么json包不能导出私有变量的tag?](golang/faq/6.md)
7. [对已经关闭的chan进行读写会怎么样?为什么?](golang/faq/7.md)
8. [对未初始化的chan进行读写，会怎么样?为什么?](golang/faq/8.md)
9. [for循环select时， 如果通道关闭会怎么样?如果select中的case只有一 个，又会怎么样?](golang/faq/9.md)
10. [10道Go语言并发题目测试](golang/faq/10.md)

## 算法 ⌛️ 

- [《剑指offer》解析](https://github.com/zmk-c/go-offer)
- [leetcode刷题顺序解析](https://github.com/zmk-c/leetcode)
- [常见排序算法](sort/algorithm.md)
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

## 操作系统💻

- 计算机操作系统
- Linux系统

## 计算机网络☁️

- [计算机网络基础](network/network.md)
- [在B站看猫片被老板发现？不如按下F12学学HTTP](https://mp.weixin.qq.com/s/T41YBEmG4lkxokDLzRxVgA)
- [TCP粘包 数据包：我只是犯了每个数据包都会犯的错 |硬核图解](https://mp.weixin.qq.com/s/0H8WL6QeZ2VbO1hHPkn8Ug)
- [硬核图解！30张图带你搞懂！路由器，集线器，交换机，网桥，光猫有啥区别？](https://mp.weixin.qq.com/s/BJqp72EyEMahxi2XOfSrBQ)

## 工具🔧 

- ### [Git](git/git.md)

- ### Docker
  
  - [docker](docker/docker.md)
  - [docker-compose](docker/docker-compose.md)
  
- ### Kubernetes


### 未完待续...