# Go语言深度解析之垃圾回收机制

## 常见的垃圾回收方法

1. 引用计数（reference counting）

   对每个对象维护一个引用计数，当引用该对象的对象被销毁时，引用计数减1，当引用计数器为0是回收该对象。

   - 优点：对象可以很快的被回收，不会出现内存耗尽或达到某个阀值时才回收。
   - 缺点：不能很好的处理循环引用，而且实时维护引用计数，有也一定的代价。
   - 代表语言：Python、PHP、Swift

2. 标记清除（mark and sweep）

   该方法分为两步：1.从根变量来遍历所有被引用的对象进行标记，2.对未标记对象进行回收。优点：解决了引用计数的缺点。但每次垃圾回收的时候都会暂停所有正常运行的代码`STW`(stop the world)导致**卡顿**，所以后面有mark and sweep的变种方法——**三色标记法**（golang采用的垃圾回收算法），用来缓解性能问题。	

3. 分代收集（generation）

   jvm使用的垃圾回收算法。在面向对象编程语言中，绝大多数对象的生命周期非常短，分代收集的基本思想是，将堆划分为两个或多个称为代(generation)的空间，新创建的对象存放在称为**新生代**，随着垃圾回收的重复执行，生命周期较长的对象会被提升到老年代中。然后分成针对新生代和老年代的垃圾回收方式。

## Golang中的垃圾回收

>  记住：当前Golang使用的垃圾回收机制是**三色标记法**配合**写屏障**和**辅助GC**，三色标记法是**标记-清除法**的一种增强版本。

简单的说，垃圾回收的核心就是标记出哪些内存还在使用中（即被引用到），哪些内存不再使用了（即未被引用），把未被引用的内存回收掉，以供后续内存分配时使用。

前面介绍内存分配时，介绍过`span`数据结构，`span`中维护了一个个内存块，并由一个位图`allocBits`表示每个内存块的分配情况。在span数据结构中还有另一个位图`gcmarkBits`用于标记内存块被引用情况。

![img](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E5%86%85%E5%AD%98%E6%A0%87%E8%AE%B0.jpeg)

如上图所示，`allocBits`记录了每块内存分配情况，而`gcmarkBits`记录了每块内存标记情况。标记阶段对每块内存进行标记，有对象引用的的内存标记为1(如图中灰色所示)，没有引用到的保持默认为0。

allocBits和gcmarkBits数据结构是完全一样的，标记结束就是内存回收，回收时将allocBits指向gcmarkBits，则代表标记过的才是存活的，gcmarkBits则会在下次标记时重新分配内存，非常的巧妙。

### 三色标记

三色标记法将对象的颜色分为了灰、黑、白，三种：

- 灰色：对象已被标记，但这个对象包含的子对象未标记
- 黑色：对象已被标记，且这个对象包含的子对象也已标记（gcmarkBits对应的位为1，该对象不会在本次GC中被清理）
- 白色：对象未被标记（gcmarkBits对应的位为0，该对象将会在本次GC中被清理）

例如，当前内存中有A~F一共6个对象，根对象a,b本身为栈上分配的局部变量，根对象a、b分别引用了对象A、B, 而B对象又引用了对象D，则GC开始前各对象的状态如下图所示:

![img](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/20210811144258.jpeg)

初始状态下所有对象都是白色的。

接着开始扫描根对象a、b:

![img](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/20210811145454.jpeg)

由于根对象引用了对象A、B,那么A、B变为灰色对象，接下来就开始分析灰色对象，分析A时，A没有引用其他对象很快就转入黑色，B引用了D，则B转入黑色的同时还需要将D转为灰色，进行接下来的分析。如下图所示：

![img](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/20210811145508.jpeg)

上图中灰色对象只有D，由于D没有引用其他对象，所以D转入黑色。标记过程结束：

![img](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/20210811145520.jpeg)

最终，黑色的对象会被保留下来，白色对象会被回收掉。

**下面总结一下三色标记法的流程：**

1. GC的根对象会被标记为灰色（当前运行的进程所需的对象）；
2. 从当前灰色集合中获取对象，将该对象引用到的对象标记为灰色，自身则标记为黑色；
3. 重复步骤2，直到没有灰色集合可以标记为止；
4. 对剩下没有标记的白色对象进行回收（表示GC 根对象不可达）；

### 三色标记存在的问题

因为go支持并行GC，GC的扫描和go代码可以同时运行，这样带来的问题是GC扫描的过程中go代码有可能改变了对象的依赖树。因此三色标记也会存在一些问题：

1. **多标 - 浮动垃圾问题**

   假设 E 已经被标记过了（变成灰色了），此时 D 和 E 断开了引用，按理来说对象 E/F/G 应该被回收的，但是因为 E 已经变为灰色了，其仍会被当作存活对象继续遍历下去。最终的结果是：这部分对象仍会被标记为存活，即**本轮 GC 不会回收这部分内存**。

   这部分本应该回收 但是没有回收到的内存，被称之为“浮动垃圾”。过程如下图所示：

   ![image-20210811131733133](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E5%A4%9A%E6%A0%87-%E6%B5%AE%E5%8A%A8%E5%9E%83%E5%9C%BE%E9%97%AE%E9%A2%98.png)

2. **漏标 - 悬挂指针问题**

   当 GC 线程已经遍历到 E 变成灰色，D变成黑色时，灰色 E 断开引用白色 G ，黑色 D 引用了白色 G。此时切回 GC 线程继续跑，因为 E 已经没有对 G 的引用了，所以不会将 G 放到灰色集合。尽管因为 D 重新引用了 G，但因为 D 已经是黑色了，不会再重新做遍历处理。

   最终导致的结果是：G 会一直停留在白色集合中，最后被当作垃圾进行清除。这直接影响到了应用程序的正确性，是不可接受的，这也是 Go 需要在 GC 时解决的问题。过程如下图所示：

   ![image-20210811132134505](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E6%BC%8F%E6%A0%87-%E6%82%AC%E6%8C%82%E6%8C%87%E9%92%88%E9%97%AE%E9%A2%98.png)

为了避免这个问题，go在GC的标记阶段会启用**写屏障**（Write Barrier），写屏障类似一种开关，在GC的特定时机开启，开启后指针传递时会把指针标记，即本轮不回收，下次GC时再确定。

同时为了防止内存分配过快，在GC执行过程中，如果goroutine需要分配内存，那么这个goroutine会参与一部分GC的工作，这个机制叫做**辅助GC（Mutator Assist）**

### 垃圾回收触发时机

1. 内存分配量达到阀值

   每次内存分配时都会检查当前内存分配量是否已达到阀值，如果达到阀值则立即启动GC。

   ```
   阀值 = 上次GC内存分配量 * 内存增长率
   ```

   内存增长率由环境变量`GOGC`控制，默认为100，即每当内存扩大一倍时启动GC。

2. 定期触发GC

   默认情况下，最长2分钟触发一次GC，这个间隔在`src/runtime/proc.go:forcegcperiod`变量中被声明：

   ```go
   // forcegcperiod is the maximum time in nanoseconds between garbage
   // collections. If we go this long without a garbage collection, one
   // is forced to run.
   //
   // This is a variable for testing purposes. It normally doesn't change.
   var forcegcperiod int64 = 2 * 60 * 1e9
   ```

3. 手动触发

   程序代码中也可以使用`runtime.GC()`来手动触发GC。这主要用于GC性能测试和统计。

## 总结golang GC的垃圾回收阶段

GC 相关的代码在`runtime/mgc.go`文件下。通过注释介绍我们可以知道 GC 一共分为4个阶段：

1. 准备阶段：STW，初始化标记任务，启用写屏障
2. 标记阶段 GCMark：标记存活对象，并发与用户代码执行，保持只占用25%CPU
3. 标记终止阶段 GCMarkTermination：STW，关闭写屏障
4. 清扫阶段 GCOff：回收白色对象，并发与用户代码执行

> Reference：
>
> 1. [Go语言——垃圾回收GC](https://www.jianshu.com/p/8b0c0f7772da)
> 2. [Go语言GC实现原理及源码分析](https://juejin.cn/post/6941768640265977886)
> 3. [《Go专家编程》Go 垃圾回收原理](https://my.oschina.net/renhc/blog/2244717)
> 4. [**golang gc**](http://yangxikun.github.io/golang/2019/12/22/golang-gc.html)