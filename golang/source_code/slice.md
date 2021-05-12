# Go语言源码分析之slice

## 1.slice是什么

`slice` 翻译成中文就是`切片`，它和`数组（array）`很类似，可以用下标的方式进行访问，如果越界，就会产生 panic。但是它比数组更灵活，可以自动地进行扩容。

了解 slice 的本质，最简单的方法就是看它的源代码：

```go
// go 1.14 src/runtime/slice.go
type slice struct {
    array unsafe.Pointer // 元素指针 指向底层数组
    len   int // 长度 表示切片可用元素的个数
    cap   int // 容量 底层数组的元素个数，一般>=长度
}
```

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/slice_under.png" alt="图片" style="zoom:50%;" />

可以看到slice共有三个属性，每个属性的解释如上所示，其中在底层数组不进行扩容的情况下，容量也是 slice 可以扩张的最大限度。需要注意的是，**底层数组是可以被多个 slice 同时指向的**，因此对一个 slice 的元素进行操作是有可能影响到其他 slice 的。

## 2.slice的创建

创建 slice 的方式有以下几种：

| 序号 | 方式               | 代码示例                                             |
| ---- | ------------------ | ---------------------------------------------------- |
| 1    | 直接声明           | `var slice []int`                                    |
| 2    | new                | `slice := *new([]int)`                               |
| 3    | 字面量             | `slice := []int{1,2,3,4,5}`                          |
| 4    | make               | `slice := make([]int, 5, 10)`                        |
| 5    | 从切片或数组“截取” | `slice := array[1:5]` 或 `slice := sourceSlice[1:5]` |

其中第一种方式和第二种方式创建出来的切片其实是一种`nil slice`，与之对应的还有一种`empty slice`。

区别如下（官方建议使用`nil slice`）：

| 创建方式      | nil切片              | 空切片                  |
| ------------- | -------------------- | ----------------------- |
| 方式一        | var s1 []int         | var s2 = []int{}        |
| 方式二        | var s3 = *new([]int) | var s4 = make([]int, 0) |
| 长度          | 0                    | 0                       |
| 容量          | 0                    | 0                       |
| 和 `nil` 比较 | `true`               | `false`                 |

```go
package main

import "fmt"

func main() {
	var s1 []int
	var s2 = *new([]int)
	var s3 = make([]int, 0)
	var s4 = []int{}
	fmt.Printf("s1's address is %p\n", s1)
	fmt.Printf("s2's address is %p\n", s2)
	fmt.Printf("s3's address is %p\n", s3)
	fmt.Printf("s4's address is %p\n", s4)
	fmt.Printf("s1 equal nil? %v\n", s1 == nil)
	fmt.Printf("s2 equal nil? %v\n", s2 == nil)
	fmt.Printf("s3 equal nil? %v\n", s3 == nil)
	fmt.Printf("s4 equal nil? %v\n", s4 == nil)
}
```

```
结果如下：
s1's address is 0x0
s2's address is 0x0
s3's address is 0x5a6d48
s4's address is 0x5a6d48
s1 equal nil? true
s2 equal nil? true
s3 equal nil? false
s4 equal nil? false
```

所有的空切片`empty slice`的数据指针都指向**同一个地址`0x5a6d48`**。

下面来看看“截取”方式：

第一种：

```go
data := [...]int{0,1,2,3,4,5,6,7,8,9}
slice := data[2:4] //data[low,high]   
```

对 `data` 使用2个索引值，截取出新的`slice`。这里 `data` 可以是数组或者 slice。`low` 是最低索引值，为闭区间，也就是说第一个元素是 data 位于 `low` 索引处的元素（这里为元素`2`）；而 `high` 则是开区间，表示最后一个元素只能是索引 `high-1` 处的元素（这里为元素`3`），新的slice的长度计算方式为`high-low`（这里长度为4-2=2），而容量则是从当前low索引往后的元素个数（这里容量为8）。

```go
package main

import "fmt"

func main() {
	data1 := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	slice := data1[2:4]
	fmt.Printf("slice is %v\n", slice)
	fmt.Printf("slice' length = %v\n", len(slice))
	fmt.Printf("slice' cap = %v\n", cap(slice))
}
```

```
结果：
slice is [2 3]
slice' length = 2
slice' cap = 8
```

第二种：

```go
 data := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9} 
 slice := data[2:4:6] // data[low, high, max]  其中max >= high >= low
```

对 `data` 使用3个索引值，多出了一个`max`索引，其中`low`和`high`的作用相同，可以计算出slice的长度为`high-low`(这里长度同样为4-2=2)，而`max`也是开区间，而最大容量则只能是索引 `max-1` 处的元素（这里为`5`），slice容量的计算方式为`max-low`(这里容量为6-2=4)。

```go
package main

import "fmt"

func main() {
	data := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
    slice := data[2:4:6]
	fmt.Printf("slice is %v\n", slice)
	fmt.Printf("slice' length = %v\n", len(slice))
	fmt.Printf("slice' cap = %v\n", cap(slice))
}
```

```
结果：
slice is [2 3]
slice' length = 2
slice' cap = 4
```

注意：

- 当 `high == low` 时，新 `slice` 为空。

- 还有一点，`high` 和 `max` 必须在老数组或者老 `slice` 的容量（`cap`）范围内。

**这里引入一道题，对“截取”做一个好好的回顾：**

```go
package main
import "fmt"
func main() {   	
    slice := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

	s1 := slice[2:5]
	s2 := s1[2:6:7]

	s2 = append(s2, 100)
	s2 = append(s2, 200)

	s1[2] = 20

	fmt.Println(s1)
	fmt.Println(s2)
	fmt.Println(slice)
}
```

```
结果：
[2 3 20]
[4 5 6 7 100 200]
[0 1 2 3 20 5 6 7 100 9]
```

来分析一遍代码，初始状态如下：

```go
slice := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
s1 := slice[2:5]
s2 := s1[2:6:7]
```

`s1` 从 `slice`的索引 [2,5)，长度为3，容量默认到数组结尾，为8。 `s2` 从 `s1`的索引 [2,6)，容量大小从[2,7)，为5。

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/exp1.jpeg" alt="图片" style="zoom:50%;" />



接着，向 `s2` 尾部追加一个元素 100：

```go
s2 = append(s2, 100)
```

`s2` 容量刚好够，直接追加。不过，这会修改原始数组对应位置的元素。这一改动，数组和 `s1` 都可以看得到。

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/exp2.jpeg" alt="图片" style="zoom:50%;" />



再次向 `s2` 追加元素200：

```go
s2 = append(s2, 100)
```

这时，`s2` 的容量不够用，该扩容了。于是，`s2` 另起炉灶，将原来的元素复制新的位置，扩大自己的容量。并且为了应对未来可能的 `append` 带来的再一次扩容，`s2` 会在此次扩容的时候多留一些 `buffer`，将新的容量将扩大为原始容量的`2倍`（具体见下小节分析），也就是10了。

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/exp3.webp" alt="图片" style="zoom:50%;" />



最后，修改 `s1` 索引为2位置的元素：

```go
s1[2] = 20
```

这次只会影响原始数组相应位置的元素。它影响不到 `s2` ，因为不属于同一个底层数组了。

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/20210512212327.webp" alt="图片" style="zoom:50%;" />



最后打印 `s1` 的时候，只会打印出 `s1` 长度以内的元素。所以，只会打印出3个元素，虽然它的底层数组不止3个元素。

## 3.slice的append操作

在2节最后一段提到了append操作，**下面让我们来看看slice的append操作到底做了什么**：

先来看看 `append` 函数的原型：

```go
func append(slice []Type, elems ...Type) []Type
```

append 函数的参数长度可变，因此可以追加多个值到 slice 中，还可以用 `...` 传入 slice，直接追加一个切片。

```go
slice = append(slice, elem1, elem2)
slice = append(slice, anotherSlice...)
```

使用 append 可以向 slice 追加元素，实际上是往底层数组添加元素。但是底层数组的长度是固定的，如果索引 `len-1` 所指向的元素已经是底层数组的最后一个元素，就没法再添加了。

这时，slice 会迁移到新的内存位置，新底层数组的长度也会增加，这样就可以放置新增的元素。同时，为了应对未来可能再次发生的 append 操作，新的底层数组的长度，也就是新 `slice` 的容量是留了一定的 `buffer` 的。否则，每次添加元素的时候，都会发生迁移，成本太高。

新 slice 预留的 `buffer` 大小是有一定规律的。网上大多数的文章都是这样描述的：

> 当原 slice 容量小于 `1024` 的时候，新 slice 容量变成原来的 `2` 倍；原 slice 容量超过 `1024`，新 slice 容量变成原来的`1.25`倍。

==这句话是错误的！==

**举个例子：**

```go
package main

import "fmt"

func main() {
    s := []int{1,2}
    s = append(s,4,5,6)
    fmt.Printf("len=%d, cap=%d",len(s),cap(s))
}
```

```
结果：
len=5,cap=6
```

如果按网上各种文章中总结的那样：**小于原 slice 长度小于 1024 的时候，容量每次增加 1 倍。添加元素 4 的时候，容量变为4；添加元素 5 的时候不变；添加元素 6 的时候容量增加 1 倍，变成 8。**

那上面代码的运行结果就是：

```
len=5, cap=8
```

这是错误的，下面来看看源码：

```go
// go 1.14 src/runtime/slice.go:76
//growslice()：它被传递给slice元素类型，旧的slice和所需的新的最小容量，并返回一个至少具有该容量的新slice，并将旧数据复制到其中。
func growslice(et *_type, old slice, cap int) slice {
	//...
	newcap := old.cap
	doublecap := newcap + newcap
	//如果新的容量大于旧的两倍，则直接扩容到新的容量
	if cap > doublecap {
		newcap = cap
	} else {
		// 当新的容量不大于旧的两倍
		// 如果旧长度小于1024，那扩容到旧的两倍
		if old.len < 1024 {
			newcap = doublecap
		} else {
			//否则扩容到旧的1.25倍
			for 0 < newcap && newcap < cap {
				newcap += newcap / 4
			}
			//...
	}

	//...
    //内存对齐
    capmem = roundupsize(uintptr(newcap) * sys.PtrSize)
    newcap = int(capmem / sys.PtrSize)
}
```

如果只看前半部分，现在网上各种文章里说的 `newcap` 的规律是对的。现实是，后半部分还对 `newcap` 作了一个`内存对齐roundupsize`，这个**和内存分配策略相关**。

>  进行内存对齐之后，新 slice 的容量是要 `大于等于` 老 slice 容量的 `2倍`或者`1.25倍`。

**例子分析：**

`growslice`函数的参数依次是 `元素的类型，旧slice，新slice最小求的容量`。

例子中 `s` 原来只有 2 个元素，`len` 和 `cap` 都为 2，`append` 了三个元素后，长度变为 3，容量最小要变成 5，即调用 `growslice` 函数时，传入的第三个参数应该为 5。即 `cap=5`。而一方面，`doublecap` 是原 `slice`容量的 2 倍，等于 4。满足第一个 `if` 条件，所以 `newcap` 变成了 5。

接着调用了 `roundupsize` 函数，传入` size=40`。（代码中`sys.PtrSize`是指一个指针的大小，在64位机上是8，即传入5*8）

```go
// go 1.14 src/runtime/internal/sys/stubs.go: 8
//拿 64 系统来说，0 取反之后右移 63 位的结果是 1，然后 4 的二进制表示是 0100，左移 1 位的结果是 1000，结果为 8。
const PtrSize = 4 << (^uintptr(0) >> 63) 
```

下面来看看`roundupsize`源码：

```go
// go 1.14 src/runtime/msize.go:13
func roundupsize(size uintptr) uintptr {
	if size < _MaxSmallSize {
		if size <= smallSizeMax-8 {
			return uintptr(class_to_size[size_to_class8[(size+smallSizeDiv-1)/smallSizeDiv]])
		} else {
			return uintptr(class_to_size[size_to_class128[(size-smallSizeMax+largeSizeDiv-1)/largeSizeDiv]])
		}
	}
	//...
}

其中：
const _MaxSmallSize = 32768
const smallSizeMax = 1024
const smallSizeDiv = 8
```

传入size=40后，很明显，最后返回

```go
class_to_size[size_to_class8[(size+smallSizeDiv-1)/smallSizeDiv]]
```

这是 `Go` 源码中有关内存分配的两个 `slice`。`class_to_size`通过 `spanClass`获取 `span`划分的 `object`大小。而 `size_to_class8` 表示通过 `size` 获取它的 `spanClass`。

```go
// go 1.14 src/runtime/sizeclasses.go
var class_to_size = [_NumSizeClasses]uint16{0, 8, 16, 32, 48, 64, 80, 96, 112, 128, 144, 160, 176, 192, 208, 224, 240, 256, 288, 320, 352, 384, 416, 448, 480, 512, 576, 640, 704, 768, 896, 1024, 1152, 1280, 1408, 1536, 1792, 2048, 2304, 2688, 3072, 3200, 3456, 4096, 4864, 5376, 6144, 6528, 6784, 6912, 8192, 9472, 9728, 10240, 10880, 12288, 13568, 14336, 16384, 18432, 19072, 20480, 21760, 24576, 27264, 28672, 32768}

var size_to_class8 = [smallSizeMax/smallSizeDiv + 1]uint8{0, 1, 2, 3, 3, 4, 4, 5, 5, 6, 6, 7, 7, 8, 8, 9, 9, 10, 10, 11, 11, 12, 12, 13, 13, 14, 14, 15, 15, 16, 16, 17, 17, 18, 18, 18, 18, 19, 19, 19, 19, 20, 20, 20, 20, 21, 21, 21, 21, 22, 22, 22, 22, 23, 23, 23, 23, 24, 24, 24, 24, 25, 25, 25, 25, 26, 26, 26, 26, 26, 26, 26, 26, 27, 27, 27, 27, 27, 27, 27, 27, 28, 28, 28, 28, 28, 28, 28, 28, 29, 29, 29, 29, 29, 29, 29, 29, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 30, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31}
```

我们传进去的 `size` 等于 40。所以 `(size+smallSizeDiv-1)/smallSizeDiv = 5`；获取 `size_to_class8` 数组中索引为 `5` 的元素为 `4`；获取 `class_to_size` 中索引为 `4` 的元素为 `48`。

最终，新的 slice 的容量为 `6`：

```
newcap = int(capmem / sys.PtrSize)  // 48 / 8 = 6
```

## 4.为什么 nil slice 可以直接 append

其实 `nil slice` 或者 `empty slice` 都是可以通过调用 append 函数来获得底层数组的扩容。最终都是调用 `mallocgc` 来向 Go 的内存管理器申请到一块内存，然后再赋给原来的`nil slice` 或 `empty slice`，然后摇身一变，成为“真正”的 `slice` 了。

## 5.传 slice 和 slice 指针有什么区别

当 slice 作为函数参数时，就是一个普通的结构体。其实很好理解：若直接传 slice，在调用者看来，实参 slice 并不会被函数中的操作改变；若传的是 slice 的指针，在调用者看来，是会被改变原 slice 的。

值的注意的是，不管传的是 slice 还是 slice 指针，如果改变了 slice 底层数组的数据，会反应到实参 slice 的底层数据。为什么能改变底层数组的数据？很好理解：**底层数据在 slice 结构体里是一个指针，仅管 slice 结构体自身不会被改变，也就是说底层数据地址不会被改变。 但是通过指向底层数据的指针，可以改变切片的底层数据**。

通过 slice 的 array 字段就可以拿到数组的地址。在代码里，是直接通过类似 `s[i]=10` 这种操作改变 slice 底层数组元素值。

将切片通过参数传递给函数，其实质是复制了slice结构体对象，两个slice结构体的字段值均相等。正常情况下，由于函数内slice结构体的array和函数外slice结构体的array指向的是同一底层数组，所以当对底层数组中的数据做修改时，两者均会受到影响。

但是存在这样的问题：如果指向底层数组的指针被覆盖或者修改（copy、重分配、append触发扩容），此时函数内部对数据的修改将不再影响到外部的切片，代表长度的len和容量cap也均不会被修改。

## 总结

- 切片是对底层数组的一个抽象，描述了它的一个片段。
- 切片实际上是一个结构体，它有三个字段：长度，容量，底层数据的地址。
- 多个切片可能共享同一个底层数组，这种情况下，对其中一个切片或者底层数组的更改，会影响到其他切片。
- `append` 函数会在切片容量不够的情况下，调用 `growslice` 函数获取所需要的内存，这称为扩容，扩容会改变元素原来的位置。
- 扩容策略并不是简单的扩为原切片容量的 `2` 倍或 `1.25` 倍，还有**内存对齐的操作**。扩容后的容量 >= 原容量的 `2` 倍或 `1.25` 倍。
- 当直接用切片作为函数参数时，可以改变切片的元素，不能改变切片本身；想要改变切片本身，可以将改变后的切片返回，函数调用者接收改变后的切片或者将切片指针作为函数参数。



> 参考：
>
> [深度解密Go语言之Slice](https://mp.weixin.qq.com/s/MTZ0C9zYsNrb8wyIm2D8BA)
>
> [切片传递与指针传递的区别_zuiyijiangnan的博客-CSDN博客_切片传递](https://blog.csdn.net/zuiyijiangnan/article/details/112673446)