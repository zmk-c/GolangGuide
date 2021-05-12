# Raft共识算法

## 一.背景

拜占庭将军问题是分布式领域**最复杂、最严格的容错模型**。但在日常工作中使用的分布式系统面对的问题不会那么复杂，更多的是计算机故障挂掉了，或者网络通信问题而没法传递信息，这种情况**不考虑计算机之间互相发送恶意信息**，极大简化了系统对容错的要求，最主要的是达到一致性。

所以将拜占庭将军问题根据常见的工作上的问题进行简化：**假设将军中没有叛军，信使的信息可靠但有可能被暗杀的情况下，将军们如何达成一致性决定？**

对于这个简化后的问题，有许多解决方案，第一个被证明的共识算法是 Paxos，由拜占庭将军问题的作者 Leslie Lamport 在1990年提出，但因为 Paxos 难懂，难实现，所以斯坦福大学的教授Diego Ongaro 和 John Ousterhout在2014年发表了论文**《In Search of an Understandable Consensus Algorithm》**，其中提到了新的分布式协议 `Raft`。与 Paxos 相比，Raft 有着基本相同运行效率，但是更容易理解，也更容易被用在系统开发上。



## 二.概述

Raft实现一致性的机制是这样的：首先选择一个leader全权负责管理日志复制，leader从客户端接收log entries（日志条目），将它们复制给集群中的其它机器，然后负责告诉其它机器什么时候将日志应用于它们的状态机。举个例子，leader可以在无需询问其它server的情况下决定把新entries放在哪个位置，数据永远是从leader流向其它机器（leader的强一致性）。一个leader可以fail或者与其他机器失去连接，这种情形下会有新的leader被选举出来。

在任何时刻，每个server节点有三种状态：`leader`，`candidate`，`follower`。

- leader：作为客户端的接收者，接收客户端发送的日志复制请求，并将日志信息复制到 follower 节点中，维持网络各个节点的账本状态。
- candidate：在leader 选举阶段存在的状态，通过任期号term和票数进行领导人身份竞争，获胜者将成为下一任期的领导人。
- follower：作为leader 节点发送日志复制请求的接收者，与leader节点通信，接收账本信息，并确认账本信息的有效性，完成日志信息的提交和存储。

正常运行时，只有一个leader，其余全是follower。follower是被动的：它们不主动提出请求，只是响应leader和candidate的请求。leader负责处理所有客户端请求（如果客户端先连接某个follower，该follower要负责把它重定向到leader）。candidate状态用于选举领导节点。下图展示了这些状态以及它们之间的转化:

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314220524044-1841690773.png)

Raft将时间分解成任意长度的`terms`，如下图所示：

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314220535693-1767695662.png)

terms有连续单调递增的编号，每个term开始于选举，这一阶段每个candidate都试图成为leader。如果一个candidate选举成功，它就在该term剩余周期内履行leader职责。在某种情形下，可能出现选票分散，没有选出leader的情况，这时新的term立即开始。**Raft确保在任何term都只可能存在一个leader**。term在Raft用作逻辑时钟，servers可以利用term判断一些过时的信息：比如过时的leader。每台server都存储当前term号，它随时间单调递增。term号可以在任何server通信时改变：如果某个server节点的当前term号小于其它servers，那么这台server必须更新它的term号，保持一致；如果一个candidate或者leader发现自己的term过期，则降级成follower；如果某个server节点收到一个过时的请求（拥有过时的term号），它会拒绝该请求。

Raft servers使用RPC交互，基本的一致性算法只需要两种RPC。`RequestVote RPCs`由candidate在选举阶段发起。`AppendEntries RPCs`在leader复制数据时发起，leader在和follower做心跳时也用该RPC。servers发起一个RPC，如果没得到响应，则需要不断重试。另外，发起RPC是并行的。



## 三.具体共识流程

raft算法大致可以划分为两个阶段，即`Leader Selection`和`Log Relocation`，同时使用强一致性来减少需要考虑的状态。

### 3.1 Leader Selection

Raft使用`heartbeat`（心跳机制）来触发选举。当server节点启动时，初始状态都是follower。每一个server都有一个定时器，超时时间为`election timeout`（**时间长度一般为150ms~300ms**），如果某server没有超时的情况下收到来自leader或者candidate的任何RPC，则定时器**重启**，如果超时，它就开始一次选举。leader给followers发RPC要么复制日志，要么就是用来告诉followers自己是leader，不用选举的心跳（告诉followers对状态机应用日志的消息夹杂在心跳中）。如果某个candidate获得了**超过半数**节点的选票（自己投了自己），它就赢得了选举成为新leader。

上述的具体过程如下：

➢ 初始状态下集群中的所有节点都处于 follower 状态。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314220617194-1813732136.png)

➢ 某一时刻，其中的一个 follower 由于没有收到 leader 的 heartbeat 率先发生 election timeout 进而发起选举。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314221034628-908216094.png)

➢ 只要集群中超过半数的节点接受投票，candidate 节点将成为即切换 leader 状态。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314221133536-155588085.png)

➢ 成为 leader 节点之后，leader 将定时向 follower 节点同步日志并发送 heartbeat。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314221226332-2116574232.png)

**如果leader节点出现了故障，那怎么办？**

下面将说明当集群中的 leader 节点不可用时，raft 集群是如何应对的。

➢ 一般情况下，leader 节点定时发送 heartbeat 到 follower 节点。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314221306319-1624406643.png)

➢ 由于某些异常导致 leader 不再发送 heartbeat ，或 follower 无法收到 heartbeat 。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314221422239-219883405.png)

➢ 当某一 follower 发生election timeout 时，其状态变更为 candidate，并向其他 follower 发起投票。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314221442435-917603935.png)

➢ 当超过半数的 follower 接受投票后，这一节点将成为新的 leader，leader 的任期号term加1并开始向 follower 同步日志。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314221515727-1608774094.png)

➢ 当一段时间之后，如果之前的 leader 再次加入集群，则两个 leader 比较彼此的任期号，任期号低的leader将切换自己的状态为follower。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314221529947-1104329632.png)

➢ 较早前 leader 中不一致的日志将被清除，并与现有 leader 中的日志保持一致。
![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210314221625602-1322176874.png)

还有第三种可能性就是candidate既没选举成功也没选举失败：如果多个follower同时成为candidate去拉选票，导致选票分散，任何candidate都没拿到大多数选票，这种情况下Raft使用超时机制`election timeout`来解决。所以同时出现多个candidate的可能性不大，即使机缘巧合同时出现了多个candidate导致选票分散，那么它们就等待自己的election timeout超时，重新开始一次新选举，实验也证明这个机制在选举过程中收敛速度很快。

### 3.2 Log Relocation

在 raft 集群中，所有日志都必须首先提交至 leader 节点。leader 在每个 heartbeat 向 follower 发送AppendEntries RPC同步日志，follower如果发现没问题，复制成功后会给leader一个表示成功的ACK，leader收到超过半数的ACK后应用该日志，返回客户端执行结果。若 follower 节点宕机、运行缓慢或者丢包，则 leader 节点会不断重试AppendEntries RPC，直到所有 follower 节点最终都复制所有日志条目。

上述的具体过程如下：

➢ 首先有一条 uncommitted 的日志条目提交至 leader 节点。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091224975-1159667758.png)

➢ 在下一个 heartbeat，leader 将此条目复制给所有的 follower。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091241400-1910380809.png)

➢ 当大多数节点记录此条目之后，leader 节点认定此条目有效，将此条目设定为已提交并存储于本地磁盘。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091259300-469736576.png)

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091305693-1149042857.png)

➢ 在下一个 heartbeat，leader 通知所有 follower 提交这一日志条目并存储于各自的磁盘内。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091337576-1642812043.png)

**Network Partition 情况下进行复制日志:**

由于网络的隔断，造成集群中多数的节点在一段时间内无法访问到 leader 节点。按照 raft 共识算法，没有 leader 的那一组集群将会通过选举投票出新的 leader，甚至会在两个集群内产生不一致的日志条目。在集群重新完整连通之后，原来的 leader 仍会按照 raft 共识算法从步进数更高的 leader 同步日志并将自己切换为 follower。

➢ 集群的理想状态。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091525429-149847836.png)

➢ 网络间隔造成大多数的节点无法访问 leader 节点。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091610822-1817228267.png)

➢ 新的日志条目添加到 leader 中。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091636052-1248453194.png)

➢ leader 节点将此条日志同步至能够访问到 leader 的节点。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091700355-638880697.png)

➢ follower 确认日志被记录，但是确认记录日志的 follower 数量没有超过集群节点的半数，leader 节点并不将此条日志存档。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091721168-1485930780.png)

➢ 在被隔断的这部分节点，在 election timeout 之后，followers 中产生 candidate 并发起选举。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315091731090-569718079.png)

➢ 多数节点接受投票之后，candidate 成为 leader。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315092036233-1288529334.png)

➢ 一个日志条目被添加到新的 leader并复制给新 leader 的 follower。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315092133129-1417768300.png)

➢ 多数节点确认之后，leader 将日志条目提交并存储。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315092256566-1625387423.png)

➢ 在下一个 heartbeat，leader 通知 follower 各自提交并保存在本地磁盘。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315092304566-153284042.png)

➢ 经过一段时间之后，集群重新连通到一起，集群中出现两个 leader 并且存在不一致的日志条目。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315092348915-2020717248.png)

➢ 新的 leader 在下一次 heartbeat timeout 时向所有的节点发送一次 heartbeat。

![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315092400133-1355454482.png)

➢ leader 在收到任期号term更高的  leader heartbeat 时放弃 leader 地位并切换到 follower 状态。
![img](https://img2020.cnblogs.com/blog/2293300/202103/2293300-20210315092431973-1040440738.png)

➢ 此时leader同步未被复制的日志条目给所有的 follower。

![img](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/20210512211022.png)

通过这种方式，只要集群中有效连接的节点超过总数的一半，集群将一直以这种规则运行下去并始终确保各个节点中的数据始终一致。



> **参考：**
>
> - [共识算法：Raft](https://www.jianshu.com/p/8e4bbe7e276c)
> - [Raft原理动画](http://thesecretlivesofdata.com/raft/)
> - Ongaro D, Ousterhout J. In search of an understandable consensus algorithm［C］// USENIX Annual Technical Conference. [s.l.]: USENIX. 2014: 305-319．