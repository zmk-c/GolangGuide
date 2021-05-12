# Redis中的主从复制原理

## 前言

持久化保证了即使 redis 服务重启也会丢失数据，因为 redis 服务重启后会将硬盘上持久化的数据恢复到内存中，但是当 redis 服务器的硬盘损坏了可能会导致数据丢失，如果通过 redis 的主从复制机制就可以避免这种单点故障。如下图：

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/20210512165817.png" alt="image-20210512165817442" style="zoom:50%;" />



先说下为啥要用主从这样的架构模式，前面提到了单机**QPS**是有上限的，而且Redis的特性就是必须支撑读高并发的，那你一台机器又读又写，这谁顶得住啊，不当人啊！但是你让这个master机器去写，数据同步给别的slave机器，他们都拿去读，**实现读写分离**，且slave分发掉大量的请求，减少master负担，而且扩容的时候还可以轻松实现**水平扩容**。

![img](https://user-gold-cdn.xitu.io/2019/11/7/16e43d17dfaf05bb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 回归正题，他们数据怎么同步的呢？

你启动一台slave 的时候，他会发送一个`psync`命令给master ，如果是这个slave第一次连接到master，他会触发一个**全量复制**。master就会启动一个线程，生成**RDB快照**，还会把新的写请求都缓存在内存中，RDB文件生成后，master会将这个RDB发送给slave的，slave拿到之后做的第一件事情就是写进本地的磁盘，然后加载进内存，然后master会把内存里面缓存的那些新命名都发给slave。

主从复制完整的工作流程分为以下三个阶段：

- 建立连接过程：这个过程就是slave跟master连接的过程
- 数据同步过程：是master给slave同步数据的过程
- 命令传播过程：是反复同步数据

![image-20210512165901957](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/20210512165902.png)

### 第一阶段：建立连接过程

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/20210512165916.png" alt="image-20210512165916557" style="zoom: 67%;" />

在建立连接的过程中，从节点会保存master的地址和端口、主节点master保存从节点slave的端口。

### 第二阶段：数据同步过程

#### 全量复制

Redis全量复制一般发生在**slave初始化阶段**，这时slave需要将master上的所有数据都复制一份。具体步骤如下： 

1. slave连接master，发送`SYNC`指令； 
2. master接收到SYNC指令后，开始执行`BGSAVE`命令生成RDB文件并使用缓冲区记录此后执行的所有写命令；
3. master的BGSAVE执行完后，向所有slave发送快照文件，并在发送期间继续记录被执行的写命令； 
4. slave收到快照文件后丢弃所有旧数据，载入收到的快照；
5. master快照发送完毕后开始向slave发送缓冲区中的写命令； 
6. slave完成对快照的载入，开始接收命令请求，并执行来自主服务器缓冲区的写命令；

#### 增量复制

slave连接maser后，会主动发起`PSYNC`命令（非首次同步），slave提供 master 的 `runid`(机器标识，随机生成的一个串) 和 `offset`（数据偏移量，如果offset主从不一致则说明数据不同步），master验证 runid 和 offset 是否有效，runid 相当于主机身份验证码，用来验证从机上一次连接的主机，如果 runid 验证未通过则，则进行全同步，如果验证通过则说明曾经同步过，根据 offset 同步部分数据。

### 第三阶段：命令传播过程

当数据同步完成以后，在此后的时间里 master-slave 之间维护着**心跳**检查来确认对方是否在线，每隔一段时间（默认10秒，通过 `repl-ping-slave-period` 参数指定）master 向 slave 发送 PING 命令判断 slave 是否在线，而 slave 每秒一次向 master 发送 `REPLCONF ACK {offset} `命令，其中 offset 指 slave 保存的复制偏移量，作用有：

- 汇报自己复制偏移量，master 会对比复制偏移量向 slave 发送未同步的命令
- 判断 master 是否在线

slave 接送命令并执行，最终实现与主库数据相同。



> 参考：
>
> - [《我们一起进大厂》系列-Redis哨兵、持久化、主从、手撕LRU](https://juejin.cn/post/6844903989184577550)
>- [写给大忙人的Redis主从复制，花费五分钟让你面试不尴尬](https://juejin.cn/post/6844904178519654414)
> - [（三）Redis主从复制介绍](https://blog.csdn.net/tianya3530/article/details/88388084)