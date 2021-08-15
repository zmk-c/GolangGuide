# Go语言深度解析之channel

## 1.什么是channel

`Goroutine` 和 `channel` 是 Go 语言并发编程的两大基石。**Goroutine 用于执行并发任务**，**channel 用于 goroutine 之间的同步、通信**。

Channel 在 goroutine 间架起了一条管道，在管道里传输数据，实现 gouroutine 间的通信；由于它是线程安全的，所以用起来非常方便；channel 还提供“先进先出”的特性；它还能影响 goroutine 的阻塞和唤醒。

相信大家都见过这一句话

> 不要通过共享内存来通信，而要通过通信来实现内存共享。

前面半句说的是通过 `sync` 包里的一些组件进行并发编程；而后面半句则是说 Go 推荐使用 channel 进行并发编程。两者其实都是必要且有效的。实际上看完本文后面对 channel 的源码分析，你会发现，channel 的底层就是通过 mutex 来控制并发的。只是 channel 是更高一层次的并发编程原语，封装了更多的功能。

channel在golang中的关键字为`chan`，对 chan 的发送和接收操作都会在编译期间转换成为底层的**发送接收函数**。

Channel 分为两种：**带缓冲**、**不带缓冲**。对不带缓冲的 channel 进行的操作实际上可以看作“同步模式”，带缓冲的则称为“异步模式”。

同步模式下，发送方和接收方要同步就绪，只有在两者都 ready 的情况下，数据才能在两者间传输（后面会看到，实际上就是内存拷贝）。否则，任意一方先行进行发送或接收操作，都会被挂起，等待另一方的出现才能被唤醒。

异步模式下，在缓冲槽可用的情况下（有剩余容量），发送和接收操作都可以顺利进行。否则，操作的一方（如写入）同样会被挂起，直到出现相反操作（如接收）才会被唤醒。

channel的底层结构为`hchan`（ `※`表示比较重要的字段），如下所示：

```go
// go 1.14 src/runtime/chan.go
type hchan struct {
	qcount   uint           // chan队列中的元素个数
	dataqsiz uint           // 底层循环队列的长度
	buf      unsafe.Pointer //※ 指向底层循环队列的长度  只针对有缓存的chan
	elemsize uint16 // chan中元素大小
	closed   uint32 // chan是否被关闭
	elemtype *_type // chan中元素类型
	sendx    uint   //※ 已发送在循环队列中的索引，指向底层循环队列
	recvx    uint   //※ 已接收在循环队列中的索引，指向底层循环队列
	recvq    waitq  //※等待接收的goroutine队列，阻塞的goroutine
	sendq    waitq  //※等待发送的goroutine队列，阻塞的goroutine

	// 保护chan中所有字段  保证每个读 channel 或写 channel 的操作都是原子的
	lock mutex
}
```

**例如**，创建一个容量为 6 的，元素为 int 型的 channel 数据结构如下 ：

![image-20210512164842504](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/20210512164842.png)



## 2.channel的操作

### 2.1创建channel

一般而言，使用 `make` 创建一个能收能发的通道有两种方式：

```go
// 无缓冲通道
ch1 := make(chan int)
// 有缓冲通道
ch2 := make(chan int, 10)
```

来看一下底层实现创建channel的源码，通过`go tool compile -S`命令用于调用Go语言提供的底层命令工具，其中`-S`参数表示输出汇编格式。发现底层是`makechain`函数创建：

```go
func makechan(t *chantype, size int64) *hchan
```

从函数原型来看，创建的 chan 是一个指针。所以我们能在函数间直接传递 channel，而不用传递 channel 的指针。

具体来看下代码：

```go
// go 1.14 src/runtime/chan.go
func makechan(t *chantype, size int) *hchan {
	elem := t.elem
	...
    // 计算所需的内存空间
	mem, overflow := math.MulUintptr(elem.size, uintptr(size))
	if overflow || mem > maxAlloc-hchanSize || size < 0 {
		panic(plainError("makechan: size out of range"))
	}

	var c *hchan
	switch {
	case mem == 0:
		// Queue or element size is zero.
		c = (*hchan)(mallocgc(hchanSize, nil, true))
		// Race detector uses this location for synchronization.
		c.buf = c.raceaddr()
	case elem.ptrdata == 0:
		// Elements do not contain pointers.
		// Allocate hchan and buf in one call.
		c = (*hchan)(mallocgc(hchanSize+mem, nil, true))
		c.buf = add(unsafe.Pointer(c), hchanSize)
	default:
		// Elements contain pointers.
		c = new(hchan)
		c.buf = mallocgc(mem, elem, true)
	}

	c.elemsize = uint16(elem.size)
	c.elemtype = elem
	c.dataqsiz = uint(size)

	if debugChan {
		print("makechan: chan=", c, "; elemsize=", elem.size, "; dataqsiz=", size, "\n")
	}
	return c
}
```









## 总结



> 参考：
>
> - [深度解密Go语言之channel](https://mp.weixin.qq.com/s/90Evbi5F5sA1IM5Ya5Tp8w)