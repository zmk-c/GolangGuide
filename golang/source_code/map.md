# Go语言源码分析之map

## 1.map是什么

`map`在计算机科学里，被称为相关数组、map、符号表或者字典，是由一组 `<key,value>` 对组成的抽象数据结构，并且同一个 key 只会出现一次。

和 map 相关的操作主要是：

1. 增加一个 k-v 对 —— add or insert；
2. 删除一个 k-v 对 —— remove or delete；
3. 修改某个 k 对应的 v —— update；
4. 查询某个 k 对应的 v —— query；

简单说就是最基本的 `增删查改`。

map的底层结构是`hmap`（即hashmap的缩写），**核心元素是一个由若干个桶（`bucket`，结构为`bmap`）组成的数组**，下面是hmap的结构体:

```go
// go 1.14 src/runtime/map.go:114
// A header for a Go map.
type hmap struct {
    count     int // 元素个数，使用len(map)时返回该值
	flags     uint8
	B         uint8  // 说明包含2^B个buckets，bucket中存储了key-value
	noverflow uint16 // 溢出的buckets近似数;
	hash0     uint32 // hash种子

	buckets    unsafe.Pointer // 指向buckets数组，大小为 2^B，如果元素个数为0，就为 nil.
	oldbuckets unsafe.Pointer // 扩容的时候，buckets 长度会是 oldbuckets 的两倍
	nevacuate  uintptr        // 指示扩容进度，小于此地址的 buckets 迁移完成

	extra *mapextra // 用于扩展
}
```

buckets 是一个指针，最终它指向的是一个结构体`bmap`：

```go
// sgo 1.14 src/runtime/map.go
// A bucket for a Go map.
type bmap struct {   
    tophash [bucketCnt]uint8
}
```

但这只是表面的结构，编译期间会给它动态地创建一个新的结构：

```go
type bmap struct {    
    topbits  [8]uint8       // 根据key计算出来的hash值的 高8位 来决定key到底落入桶内的哪个位置
    keys     [8]keytype     // 存储key的数组
    values   [8]valuetype   // 存储vlaue的数组 
    pad      uintptr        
    overflow uintptr        // 指向扩容bucket的指针
}
```

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/20210512165011.png" alt="image-20210512165011571" style="zoom:50%;" />

注意到 key 和 value 是各自放在一起的，并不是 `key/value/key/value/...` 这样的形式。源码里说明这样的好处是在某些情况下可以省略掉 padding 字段，节省内存空间。

**举个例子**，有这样一个类型的 map：

```
map[int64]int8
```

如果按照 `key/value/key/value/...` 这样的模式存储，那在每一个 key/value 对之后都要额外 padding 7 个字节；而将所有的 key，value 分别绑定到一起，这种形式 `key/key/.../value/value/...`，则只需要在最后添加 padding。

每个 bucket 设计成最多只能放 `8个key-value对`，如果有第 9 个 key-value 落入当前的 bucket，那就需要再构建一个 bucket ，通过 `overflow` 指针连接起来。

**下面来用一个整体的图表示map的结构：**

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/map_under.webp" alt="图片" style="zoom:50%;" />

当 map 的 key 和 value 都不是指针，并且 size 都小于 128 字节的情况下，**会把 bmap 标记为不含指针**，这样可以避免 gc 时扫描整个 hmap。但是，我们看 bmap 其实有一个 overflow 的字段，是指针类型的，破坏了 bmap 不含指针的设想，这时会把 overflow 移动到 extra 字段来。

```go
// go 1.14 src/runtime/map.go
type mapextra struct {
	// overflow contains overflow buckets for hmap.buckets.
	// oldoverflow contains overflow buckets for hmap.oldbuckets.
	overflow    *[]*bmap
	oldoverflow *[]*bmap

	// nextOverflow holds a pointer to a free overflow bucket.
	nextOverflow *bmap
}
```

## 2.map的操作

### 2.1创建

```go
ageMp := make(map[string]int)
// 指定 map 长度
ageMp := make(map[string]int, 8)
// 创建并初始化内容
ageMp := map[string]int{“id”:1,"age":22}
// ageMp 为 nil，不能向其添加元素，会直接panic: assign to entry in nil map. 因此需要初始化
var ageMp map[string]int
```

**注意：**map的声明的时候默认值是**nil** ，此时进行取值，返回的是**对应类型的零值**（不存在也是返回零值）。

```go
var m map[int]bool
v, ok := m[1]
fmt.Println(v, ok) // false false
```

### 2.2插入&删除&更新&查询

```go
// 插入
m[1] = "hello world"

// 删除，key不存在则啥也不干
delete(m, 1)

// 更新
m[1] = "Hello World"

// 查询，key不存在返回value类型的零值 有三种查询方式
i := m[1] 
i, ok := m[1]
_, ok := m[1]
```

### 2.3遍历

map本身是**无序的**，在遍历的时候并不会按照你传入的顺序，进行传出。

```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	m := make(map[int]string)
	//插入数据
	for i := 0; i < 50; i++ {
		m[i] = fmt.Sprintf("用户%v", i)
	}

	//正常遍历 map中的内容是无序的
	for k, v := range m {
		fmt.Println(k, v)
	}

	fmt.Println("============== 分割线 ================")

	//若想要获得map中的有序值
	//可以先用一个slice保存key值
	var key []int
	for k := range m {
		key = append(key, k)
	}

	//再用sort包进行排序
	sort.Ints(key)
    
	//最后再遍历，此时就是有序的了
	for k := range key {
		fmt.Println(k, m[k])
	}
}
```

### 2.4函数传参

Golang中是没有引用传递的，均为值传递。这意味着传递的是数据的拷贝。那么map本身是**引用类型**，作为形参或返回参数的时候，传递的是**值的拷贝，而值是地址**，**扩容**时也**不会改变**这个地址。

```go
package main

import "fmt"

func main() {
	var m map[int]int
	m = make(map[int]int, 1)
	fmt.Printf("m 原始的地址是：%p\n", m)
	changeM(m)
	fmt.Printf("m 改变后地址是：%p\n", m)
	fmt.Println("m 长度是", len(m))
	fmt.Println("m 参数是", m)
}

// 改变map的函数
func changeM(m map[int]int) {
	fmt.Printf("m 函数开始时地址是：%p\n", m)
	var max = 5
	for i := 0; i < max; i++ {
		m[i] = 2
	}
	fmt.Printf("m 在函数返回前地址是：%p\n", m)
}
```

结果：

```
m 原始地址是：0xc42007a180
m 函数开始时地址是：0xc42007a180
m 在函数返回前地址是：0xc42007a180
m 改变后地址是：0xc42007a180
m 长度是 5
m 参数是  map[3:2 4:2 0:2 1:2 2:2]
```

## 3.map的hash计算

在第一小节中我们知道bmap中存储的是key-value值，那么具体key是分配到哪个bucket呢？也就是bmap中的tophash是如何计算？

具体实现：

```go
// go 1.14 src/runtime/map.go
func tophash(hash uintptr) uint8 {
	top := uint8(hash >> (sys.PtrSize*8 - 8))
	if top < minTopHash {
		top += minTopHash
	}
	return top
}
```

key 经过哈希计算后得到哈希值，共 64 个 bit 位（64位机）计算它到底要落在哪个桶时，**只会用到最后 B 个 bit 位**。如果 B = 5，那么桶的数量，也就是 buckets 数组的长度是 2<sup>5</sup> = 32。

**例如**，现在有一个 key 经过哈希函数计算后，得到的哈希结果是：

```
 10010111 | 000011110110110010001111001010100010010110010101010 │ 00110
```

用最后的 5 个 bit 位，也就是 `00110`，值为 6，也就是 **6 号桶**。这个操作实际上就是取余操作，但是取余开销太大，所以代码实现上用的**位操作**代替。

再**用哈希值的高 8 位，找到此 key 在 bucket 中的位置**，这是在寻找已有的 key。最开始桶内还没有 key，新加入的 key 会找到第一个空位放入。

buckets 编号就是桶编号，当两个不同的 key 落在同一个桶中，也就是发生了哈希冲突。冲突的解决手段是用链表法。这样，在查找某个 key 时，先找到对应的桶，再去遍历 bucket 中的 key。

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/find_key.jpeg" alt="图片" style="zoom: 50%;" />

上图中，假定 B = 5，所以 bucket 总数就是 2<sup>5</sup> = 32。首先计算出待查找 key 的哈希，使用低 5 位 `00110`，找到对应的 6 号 bucket，使用高 8 位 `10010111`，对应十进制 151，在 6 号 bucket 中寻找 tophash 值（HOB hash）为 151 的 key，找到了 2 号槽位，这样整个查找过程就结束了。

**如果在 bucket 中没找到，并且 overflow 不为空，还要继续去 overflow bucket 中寻找，直到找到或是所有的 key 槽位都找遍了，包括所有的 overflow bucket**。

具体来看源码：

```go
// go 1.14 src/runtime.go
func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {    
    // ……    
    // 如果 h 什么都没有，返回零值    
    if h == nil || h.count == 0 {        
        return unsafe.Pointer(&zeroVal[0])   
    }    
    // 写和读冲突    
    if h.flags&hashWriting != 0 {        
        throw("concurrent map read and map write")    
    }    
    // 不同类型 key 使用的 hash 算法在编译期确定    
    alg := t.key.alg    
    // 计算哈希值，并且加入 hash0 引入随机性    
    hash := alg.hash(key, uintptr(h.hash0))    
    // 比如 B=5，那 m 就是31，二进制是全 1    
    // 求 bucket num 时，将 hash 与 m 相与，    
    // 达到 bucket num 由 hash 的低 8 位决定的效果    
    m := uintptr(1)<<h.B - 1    
    // b 就是 bucket 的地址    
    b := (*bmap)(add(h.buckets, (hash&m)*uintptr(t.bucketsize)))    
    // oldbuckets 不为 nil，说明发生了扩容    
    if c := h.oldbuckets; c != nil {        
        // 如果不是同 size 扩容（看后面扩容的内容）        
        // 对应条件 1 的解决方案        
        if !h.sameSizeGrow() {            
            // 新 bucket 数量是老的 2 倍            
            m >>= 1        
        }        
        // 求出 key 在老的 map 中的 bucket 位置        
        oldb := (*bmap)(add(c, (hash&m)*uintptr(t.bucketsize)))        
        // 如果 oldb 没有搬迁到新的 bucket        
        // 那就在老的 bucket 中寻找        
        if !evacuated(oldb) {            
            b = oldb        
        }    
    }    
    // 计算出高 8 位的 hash    
    // 相当于右移 56 位，只取高8位    
    top := uint8(hash >> (sys.PtrSize*8 - 8))    
    // 增加一个 minTopHash    
    if top < minTopHash {        
        top += minTopHash    
    }   
    for {        
        // 遍历 8 个 bucket        
        for i := uintptr(0); i < bucketCnt; i++ {            
            // tophash 不匹配，继续            
            if b.tophash[i] != top {                
                continue           
            }           
            // tophash 匹配，定位到 key 的位置           
            k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))            
            // key 是指针            
            if t.indirectkey {                
                // 解引用                
                k = *((*unsafe.Pointer)(k))            
            }            
            // 如果 key 相等            
            if alg.equal(key, k) {                
                // 定位到 value 的位置                
                v := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.valuesize))                
                // value 解引用                
                if t.indirectvalue {                   
                    v = *((*unsafe.Pointer)(v))              
                }               
                return v           
            }       
        }        
        // bucgoket 找完（还没找到），继续到 overflow bucket 里找        
        b = b.overflow(t)        
        // overflow bucket 也找完了，说明没有目标 key        
        // 返回零值        
        if b == nil {            
            return unsafe.Pointer(&zeroVal[0])        
        }    
    }
}
```

## 4.map的扩容

使用哈希表的目的就是要快速查找到目标 key，然而，随着向 map 中添加的 key 越来越多，key 发生碰撞的概率也越来越大。bucket 中的 8 个 cell 会被逐渐塞满，查找、插入、删除 key 的效率也会越来越低。最理想的情况是一个 bucket 只装一个 key，这样，就能达到 `O(1)` 的效率，但这样空间消耗太大，用空间换时间的代价太高。

Go 语言采用一个 bucket 里装载 8 个 key，定位到某个 bucket 后，还需要再定位到具体的 key，这实际上又用了时间换空间。

当然，这样做，要有一个度，不然所有的 key 都落在了同一个 bucket 里，直接退化成了链表，各种操作的效率直接降为 O(n)，是不行的。

因此，需要有一个指标来衡量前面描述的情况，这就是 `装载因子`。Go 源码里这样定义 `装载因子`：

```
loadFactor := count / (2^B)
```

count 就是 map 的元素个数，2^B 表示 bucket 数量。

**再来说触发 map 扩容的时机**：在向 map 插入新 key 的时候，会进行条件检测，符合下面这 2 个条件，就会触发扩容：

1. 装载因子超过阈值，源码里定义的阈值是 6.5。

2. overflow 的 bucket 数量过多：

   当 B 小于 15，也就是 bucket 总数 2^B 小于 2^15 时，如果 overflow 的 bucket 数量超过 2^B；

   当 B >= 15，也就是 bucket 总数 2^B 大于等于 2^15，如果 overflow 的 bucket 数量超过 2^15。

对应扩容条件的源码如下：

```go
// src/runtime/hashmap.go/mapassign
// 触发扩容时机
if !h.growing() && (overLoadFactor(int64(h.count), h.B) || tooManyOverflowBuckets(h.noverflow, h.B)) {        
    hashGrow(t, h)   
}
// 装载因子超过 6.5
func overLoadFactor(count int64, B uint8) bool {    
    return count >= bucketCnt && float32(count) >= loadFactor*float32((uint64(1)<<B))
}
// overflow buckets 太多
func tooManyOverflowBuckets(noverflow uint16, B uint8) bool {    
    if B < 16 {        
        return noverflow >= uint16(1)<<B   
    }    
    return noverflow >= 1<<15
}
```

**解释一下：**

第 1 点：我们知道，每个 bucket 有 8 个空位，在没有溢出，且所有的桶都装满了的情况下，装载因子算出来的结果是 8。因此当装载因子超过 6.5 时，表明很多 bucket 都快要装满了，查找效率和插入效率都变低了。在这个时候进行扩容是有必要的。

第 2 点：是对第 1 点的补充。就是说在装载因子比较小的情况下，这时候 map 的查找和插入效率也很低，而第 1 点识别不出来这种情况。表面现象就是计算装载因子的分子比较小，即 map 里元素总数少，但是 bucket 数量多（真实分配的 bucket 数量多，包括大量的 overflow bucket）。

## 5.并发中的map

### 5.1安全性

举例证明并发中的map是不安全的：

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	var m = map[int]int{1: 1}

	go func() {
		for i := 0; i < 1000; i++ {
			m[i] = i
		}
	}()

	go func() {
		for i := 0; i < 1000; i++ {
			m[i] = i
		}
	}()

	time.Sleep(3 * time.Second)
	fmt.Println(m)

}
```

会发现有这样的报错：

```go
fatal error: concurrent map read and map write
```

**根本原因就是：并发的去读写map结构的数据了。**

### 5.2处理方案&优缺点

解决方案就是加锁：

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var m = map[int]int{1: 1}
	var rw sync.RWMutex
	go func() {
		for i := 0; i < 1000; i++ {
			rw.Lock()
			m[i] = i
			rw.Unlock()
		}
	}()

	go func() {
		for i := 0; i < 1000; i++ {
			rw.Lock()
			m[1] = i
			rw.Unlock()
		}
	}()

	time.Sleep(3 * time.Second)
	fmt.Println(m)
}
```

> 优点：实现简单粗暴，好理解
> 缺点：锁的粒度为整个map，存在优化空间
> 适用场景：all

### 5.3官方处理方案 & 优缺点

在程序设计中，想增加运行的速度，那么必然要有另外的牺牲，很容易想到“空间换时间”的方案:

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	m := sync.Map{}
	m.Store(1, 1) //初始值
	go func() {
		for i := 0; i < 1000; i++ {
			m.Store(i, i)
		}
	}()

	go func() {
		for i := 0; i < 1000; i++ {
			m.Store(i, i)
		}
	}()

	time.Sleep(3 * time.Second)
	fmt.Println(m.Load(1)) //根据键取出值
}
```

运行完呢，会发现，其实是不会报错的。因为sync.Map里头已经实现了一套加锁的机制，让你更方便地使用map。

**sync.Map的原理介绍**：sync.Map里头有两个map一个是**专门用于读**的`read map`，另一个是才是**提供读写**的`dirty map`；优先读read map，若不存在则加锁穿透读dirty map，同时记录一个未从read map读到的计数，当计数到达一定值，就将read map用dirty map进行覆盖。

> 优点：是官方推荐的；通过空间换时间的方式；读写分离；
> 缺点：不适用于大量写的场景，这样会导致read map读不到数据而进一步加锁读取，同时dirty map也会一直晋升为read map，整体性能较差。
> 适用场景：大量读，少量写

## 总结

- Go 语言中，通过哈希查找表实现 map，用链表法解决哈希冲突。
- 通过 key 的哈希值将 key 散落到不同的桶中，每个桶中有 8 个 cell。哈希值的低位决定桶序号，高位标识同一个桶中的不同 key。
- 当向桶中添加了很多 key，造成元素过多，或者溢出桶太多，就会触发扩容。扩容分为等量扩容和 2 倍容量扩容。扩容后，原来一个 bucket 中的 key 一分为二，会被重新分配到两个桶中。
- 扩容过程是渐进的，主要是防止一次扩容需要搬迁的 key 数量过多，引发性能问题。触发扩容的时机是增加了新元素，bucket 搬迁的时机则发生在赋值、删除期间，每次最多搬迁两个 bucket。

- 注意并发条件下的map和sync.Map



> 参考：
>
> - [深度解密Go语言之map](https://mp.weixin.qq.com/s/2CDpE5wfoiNXm1agMAq4wA)
> - [golang map源码详解 (juejin.cn)](https://juejin.cn/post/6844903517530882061)
> - [由浅入深聊聊Golang的map_咖啡色的羊驼-CSDN博客_golang map](https://blog.csdn.net/u011957758/article/details/82846609)