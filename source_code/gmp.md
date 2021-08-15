# Go语言深度解析之GPM调度器

### 预前补充

先来了解一下进程，线程和Goroutine

在仅支持进程的操作系统中，进程是拥有资源和独立调度的基本单位。在引入线程的操作系统中，**线程是独立调度的基本单位，进程是资源拥有的基本单位**。在同一进程中，线程的切换不会引起进程切换。在不同进程中进行线程切换,如从一个进程内的线程切换到另一个进程中的线程时，会引起进程切换。

线程创建、管理、调度等采用的方式称为**线程模型**。线程模型一般分为以下三种：

- 内核级线程(Kernel Level Thread)模型
- 用户级线程(User Level Thread)模型
- 两级线程模型，也称混合型线程模型

**三大线程模型最大差异就在于用户级线程与内核调度实体KSE（KSE，Kernel Scheduling Entity）之间的对应关系。**KSE是Kernel Scheduling Entity的缩写，其是**可被操作系统内核调度器调度的对象实体**，是操作系统**内核的最小调度单元**，可以简单理解为内核级线程。

**用户级线程即协程**，由应用程序创建与管理，协程必须与内核级线程绑定之后才能执行。**线程由 CPU 调度是抢占式的，协程由用户态调度是协作式的，一个协程让出 CPU 后，才执行下一个协程**。

#### 内核级线程模型

![img](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E5%86%85%E6%A0%B8%E7%BA%A7%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.webp)

**内核级线程模型中用户线程与内核线程是一对一关系（1 : 1）**。**线程的创建、销毁、切换工作都是有内核完成的**。应用程序不参与线程的管理工作，只能调用内核级线程编程接口(应用程序创建一个新线程或撤销一个已有线程时，都会进行一个系统调用）。每个用户线程都会被绑定到一个内核线程。用户线程在其生命期内都会绑定到该内核线程。一旦用户线程终止，两个线程都将离开系统。

操作系统调度器管理、调度并分派这些线程。运行时库为每个用户级线程请求一个内核级线程。操作系统的内存管理和调度子系统必须要考虑到数量巨大的用户级线程。操作系统为每个线程创建上下文。进程的每个线程在资源可用时都可以被指派到处理器内核。

内核级线程模型有如下优点：

- 在多处理器系统中，内核能够并行执行同一进程内的多个线程
- 如果进程中的一个线程被阻塞，不会阻塞其他线程，是能够切换同一进程内的其他线程继续执行
- 当一个线程阻塞时，内核根据选择可以运行另一个进程的线程，而用户空间实现的线程中，运行时系统始终运行自己进程中的线程

缺点：

- 线程的创建与删除都需要CPU参与，成本大

#### 用户级线程模型

![img](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E7%94%A8%E6%88%B7%E7%BA%A7%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.webp)

**用户线程模型中的用户线程与内核线程KSE是多对一关系（N : 1）**。**线程的创建、销毁以及线程之间的协调、同步等工作都是在用户态完成**，具体来说就是由应用程序的线程库来完成。**内核对这些是无感知的，内核此时的调度都是基于进程的**。线程的并发处理从宏观来看，任意时刻每个进程只能够有一个线程在运行，且只有一个处理器内核会被分配给该进程。

从上图中可以看出来：库调度器从进程的多个线程中选择一个线程，然后该线程和该进程允许的一个内核线程关联起来。内核线程将被操作系统调度器指派到处理器内核。用户级线程是一种”多对一”的线程映射

用户级线程有如下优点：

- 创建和销毁线程、线程切换代价等线程管理的代价比内核线程少得多, 因为保存线程状态的过程和调用程序都只是本地过程
- 线程能够利用的表空间和堆栈空间比内核级线程多

缺点：

- 线程发生I/O或页面故障引起的阻塞时，如果调用阻塞系统调用则内核由于不知道有多线程的存在，而会阻塞整个进程从而阻塞所有线程, 因此同一进程中只能同时有一个线程在运行
- 资源调度按照进程进行，多个处理机下，同一个进程中的线程只能在同一个处理机下分时复用

#### 混合线程模型

![img](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E4%B8%A4%E7%BA%A7%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.webp)

**混合线程模型中用户线程与内核线程是一对一关系（N : M）**。两级线程模型充分吸收上面两种模型的优点，尽量规避缺点。其线程创建在用户空间中完成，线程的调度和同步也在应用程序中进行。一个应用程序中的多个用户级线程被绑定到一些（小于或等于用户级线程的数目）内核级线程上。

#### Go的线程模型

**Golang在底层实现了混合型线程模型**。M即系统线程，由系统调用产生，一个M关联一个KSE，即两级线程模型中的系统线程。G为Groutine，即两级线程模型的的应用及线程。M与G的关系是N:M。

![img](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/golang%E7%9A%84%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.webp)

### GMP模型

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/20210512164931.png" alt="image-20210512164931081" style="zoom:50%;" />

​																										GMP图

基于**没有什么是加一个中间层不能解决的**思路，golang在原有的`GM`模型的基础上加入了一个调度器`P`，于是就有了现在的`GMP`模型。

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/20210512164949.png" alt="image-20210512164949073" style="zoom:50%;" />

​																									GMP模型

1. **全局队列**）：存放等待运行的 `G`。

2. **P 的本地队列**：同全局队列类似，存放的也是等待运行的 `G`，存的数量有限，**不超过 256 个**。新建 `G`时，`G`优先加入到 P 的本地队列，如果本地队列满了，则会把**本地队列中一半的 G 移动到全局队列**。

3. **P 列表**：所有的` P` 都在程序启动时创建，并保存在数组中，最多有 **GOMAXPROCS(默认是CPU的核数)** 个。

4. **M**：`M`想运行任务就得获取 `P`，从 P 的本地队列获取 `G`，**访问本地队列不用加锁**。

   如果P 的本地队列为空，`M` 会尝试从全局队列拿一批`G` 放到 P 的本地队列。

   如果全局协程队列为空，`M`会从 其他`P` 的本地队列**偷一半（采用Work Stealing算法）**放到自己 P 的本地队列。

   `M` 运行 `G`，`G` 执行之后，`M` 会从 `P `获取下一个 `G`，不断重复下去。

#### 调度的生命周期

![golang调度器生命周期 ](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E8%B0%83%E5%BA%A6%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.webp)

- `M0` 是启动程序后的编号为 0 的主线程，这个 M 对应的实例会在全局变量 runtime.m0 中，不需要在 heap 上分配，M0 负责执行初始化操作和启动第一个 G， 在之后 M0 就和其他的 M 一样了
- `G0` 是每次启动一个 M 都会第一个创建的 gourtine，G0 仅用于负责调度的 G，G0 不指向任何可执行的函数，每个 M 都会有一个自己的 G0。在调度或系统调用时会使用 G0 的栈空间，全局变量的 G0 是 M0 的 G0

上面生命周期流程说明：

- runtime 创建最初的线程 m0 和 goroutine g0，并把两者进行关联（g0.m = m0)
- 调度器初始化：设置M最大数量，P个数，栈和内存出事，以及创建 GOMAXPROCS个P
- 示例代码中的 main 函数是 main.main，runtime 中也有 1 个 main 函数 ——runtime.main，代码经过编译后，runtime.main 会调用 main.main，程序启动时会为 runtime.main 创建 goroutine，称它为 main goroutine 吧，然后把 main goroutine 加入到 P 的本地队列。
- 启动 m0，m0 已经绑定了 P，会从 P 的本地队列获取 G，获取到 main goroutine。
- G 拥有栈，M 根据 G 中的栈信息和调度信息设置运行环境
- M 运行 G
- G 退出，再次回到 M 获取可运行的 G，这样重复下去，直到 main.main 退出，runtime.main 执行 Defer 和 Panic 处理，或调用 runtime.exit 退出程序。

#### 调度的流程状态

![img](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E8%B0%83%E5%BA%A6%E6%B5%81%E7%A8%8B%E7%9A%84%E7%8A%B6%E6%80%81.webp)

从上图我们可以看出来：

- 每个P有个局部队列，局部队列保存待执行的goroutine(流程2)，当M绑定的P的的局部队列已经满了之后就会把goroutine放到全局队列(流程2-1)
- 每个P和一个M绑定，M是真正的执行P中goroutine的实体(流程3)，M从绑定的P中的局部队列获取G来执行
- 当M绑定的P的局部队列为空时，M会从全局队列获取到本地队列来执行G(流程3.1)，当从全局队列中没有获取到可执行的G时候，M会从其他P的局部队列中偷取G来执行(流程3.2)，这种从其他P偷的方式称为**work stealing**
- 当G因系统调用(syscall)阻塞时会阻塞M，此时P会和M解绑即**hand off**，并寻找新的idle的M，若没有idle的M就会新建一个M(流程5.1)。
- 当G因channel或者network I/O阻塞时，不会阻塞M，M会寻找其他runnable的G；当阻塞的G恢复后会重新进入runnable进入P队列等待执行(流程5.3)

#### 调度过程中阻塞

GMP模型的阻塞可能发生在下面几种情况：

- I/O，select
- block on syscall
- channel
- 等待锁
- runtime.Gosched()

##### 用户态阻塞

当goroutine因为channel操作或者network I/O而阻塞时（实际上golang已经用netpoller实现了goroutine网络I/O阻塞不会导致M被阻塞，仅阻塞G），对应的G会被放置到某个wait队列(如channel的waitq)，该G的状态由_Gruning变为_Gwaitting，而M会跳过该G尝试获取并执行下一个G，如果此时没有runnable的G供M运行，那么M将解绑P，并进入sleep状态；当阻塞的G被另一端的G2唤醒时（比如channel的可读/写通知），G被标记为runnable，尝试加入G2所在P的runnext，然后再是P的Local队列和Global队列。

##### 系统调用阻塞

当G被阻塞在某个系统调用上时，此时G会阻塞在_Gsyscall状态，M也处于 block on syscall 状态，此时的M可被抢占调度：执行该G的M会与P解绑，而P则尝试与其它idle的M绑定，继续执行其它G。如果没有其它idle的M，但P的Local队列中仍然有G需要执行，则创建一个新的M；当系统调用完成后，G会重新尝试获取一个idle的P进入它的Local队列恢复执行，如果没有idle的P，G会被标记为runnable加入到Global队列。

#### GMP内部结构

##### G的内部结构

```go
type g struct {
    stack       stack   // g自己的栈
    m            *m      // 隶属于哪个M
    sched        gobuf   // 保存了g的现场，goroutine切换时通过它来恢复
    atomicstatus uint32  // G的运行状态
    goid         int64
    schedlink    guintptr // 下一个g, g链表
    preempt      bool //抢占标记
    lockedm      muintptr // 锁定的M,g中断恢复指定M执行
    gopc          uintptr  // 创建该goroutine的指令地址
    startpc       uintptr  // goroutine 函数的指令地址
}
```

G的状态有以下9种：

| 状态              | 值   | 含义                                                         |
| ----------------- | ---- | ------------------------------------------------------------ |
| _Gidle            | 0    | 刚刚被分配，还没有进行初始化。                               |
| _Grunnable        | 1    | 已经在运行队列中，还没有执行用户代码。                       |
| _Grunning         | 2    | 不在运行队列里中，已经可以执行用户代码，此时已经分配了 M 和 P。 |
| _Gsyscall         | 3    | 正在执行系统调用，此时分配了 M。                             |
| _Gwaiting         | 4    | 在运行时被阻止，没有执行用户代码，也不在运行队列中，此时它正在某处阻塞等待中。 |
| _Gmoribund_unused | 5    | 尚未使用，但是在 gdb 中进行了硬编码。                        |
| _Gdead            | 6    | 尚未使用，这个状态可能是刚退出或是刚被初始化，此时它并没有执行用户代码，有可能有也有可能没有分配堆栈。 |
| _Genqueue_unused  | 7    | 尚未使用。                                                   |
| _Gcopystack       | 8    | 正在复制堆栈，并没有执行用户代码，也不在运行队列中。         |

##### M的结构

```go
type m struct {
    g0      *g     // g0, 每个M都有自己独有的g0

    curg          *g       // 当前正在运行的g
    p             puintptr // 隶属于哪个P
    nextp         puintptr // 当m被唤醒时，首先拥有这个p
    id            int64
    spinning      bool // 是否处于自旋

    park          note
    alllink       *m // on allm
    schedlink     muintptr // 下一个m, m链表
    mcache        *mcache  // 内存分配
    lockedg       guintptr // 和 G 的lockedm对应
    freelink      *m // on sched.freem
}
复制代码
```

##### P的内部结构

```go
type p struct {
    id          int32
    status      uint32 // P的状态
    link        puintptr // 下一个P, P链表
    m           muintptr // 拥有这个P的M
    mcache      *mcache  

    // P本地runnable状态的G队列，无锁访问
    runqhead uint32
    runqtail uint32
    runq     [256]guintptr
    
    runnext guintptr // 一个比runq优先级更高的runnable G

    // 状态为dead的G链表，在获取G时会从这里面获取
    gFree struct {
        gList
        n int32
    }

    gcBgMarkWorker       guintptr // (atomic)
    gcw gcWork

}
复制代码
```

P有以下5种状态：

| 状态      | 值   | 含义                                                         |
| --------- | ---- | ------------------------------------------------------------ |
| _Pidle    | 0    | 刚刚被分配，还没有进行进行初始化。                           |
| _Prunning | 1    | 当 M 与 P 绑定调用 acquirep 时，P 的状态会改变为 _Prunning。 |
| _Psyscall | 2    | 正在执行系统调用。                                           |
| _Pgcstop  | 3    | 暂停运行，此时系统正在进行 GC，直至 GC 结束后才会转变到下一个状态阶段。 |
| _Pdead    | 4    | 废弃，不再使用。                                             |

##### 调度器的内部结构

```go
type schedt struct {

    lock mutex

    midle        muintptr // 空闲M链表
    nmidle       int32    // 空闲M数量
    nmidlelocked int32    // 被锁住的M的数量
    mnext        int64    // 已创建M的数量，以及下一个M ID
    maxmcount    int32    // 允许创建最大的M数量
    nmsys        int32    // 不计入死锁的M数量
    nmfreed      int64    // 累计释放M的数量

    pidle      puintptr // 空闲的P链表
    npidle     uint32   // 空闲的P数量

    runq     gQueue // 全局runnable的G队列
    runqsize int32  // 全局runnable的G数量

    // Global cache of dead G's.
    gFree struct {
        lock    mutex
        stack   gList // Gs with stacks
        noStack gList // Gs without stacks
        n       int32
    }

    // freem is the list of m's waiting to be freed when their
    // m.exited is set. Linked through m.freelink.
    freem *m
}
```

### 为什么要有P

如果是想实现本地队列、Work Stealing 算法，那为什么不直接在 M 上加呢，M 也照样可以实现类似的功能。为什么又要再多加一个组件P?

结合 `M`的定位来看，若这么做，有以下问题。

- 一般来讲，`M` 的数量都会多于 `P`。像在 golang 中，**M 的数量最大限制是 10000**，**P 的默认数量的 CPU 核数**。另外由于` M` 的属性，也就是如果存在系统阻塞调用，阻塞了`M`，又不够用的情况下，`M` 会不断增加。
- `M `不断增加的话，如果本地队列挂载在 `M` 上，那就意味着本地队列也会随之增加。这显然是不合理的，因为本地队列的管理会变得复杂，且 `Work Stealing` 性能会大幅度下降。
- `M` 被系统调用阻塞后，我们是期望把他既有未执行的任务分配给其他继续运行的，而不是一阻塞就导致全部停止。

因此使用 M 是不合理的，那么引入新的组件 `P`，把本地队列关联到 `P` 上，就能很好的解决这个问题：

- 每个 `P` 有自己的本地队列，大幅度的减轻了对全局队列的直接依赖，所带来的效果就是锁竞争的减少。而 GM 模型的性能开销大头就是锁竞争。
- 每个 `P` 相对的平衡上，在 GMP 模型中也实现了 `Work Stealing `算法，如果 P 的本地队列为空，则会从全局队列或其他 P 的本地队列中窃取可运行的 `G` 来运行，减少空转，提高了资源利用率。

> 参考:
>
> - [动图图解！GMP模型里为什么要有P？背后的原因让人暖心](https://mp.weixin.qq.com/s/O_GPwa71zqcpIkNdlkWYnQ)
> - [Golang并发调度的GMP模型](https://juejin.cn/post/6886321367604527112)
> - [再见 Go 面试官：GMP 模型，为什么要有 P？](https://mp.weixin.qq.com/s/an7dml9NLOhqOZjEGLdEEw)