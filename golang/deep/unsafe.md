# Go语言深度解析之unsafe

## 什么是unsafe

unsafe 库让 golang 可以像C语言一样操作计算机内存，但这并不是golang推荐使用的，能不用尽量不用，就像它的名字所表达的一样，它绕过了golang的内存安全原则，是不安全的，容易使你的程序出现莫名其妙的问题，不利于程序的扩展与维护。

先简单介绍下Golang指针类型：

1. `*类型`：普通指针，用于传递对象地址，不能进行指针运算。
2. `unsafe.Pointer`：通用指针类型，用于转换不同类型的指针，不能进行指针运算。
3. `uintptr`：用于指针运算，GC 不把 uintptr 当指针，uintptr 无法持有对象，**uintptr 类型的目标会被回收**。

unsafe.Pointer 可以和 普通指针 进行相互转换。

unsafe.Pointer 可以和 uintptr 进行相互转换。

也就是说 **unsafe.Pointer 是桥梁，可以让任意类型的指针实现相互转换，也可以将任意类型的指针转换为 uintptr 进行指针运算**。

![图片](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/transf.png)

unsafe底层源码如下：

两个类型：

```go
// go 1.14 src/unsafe/unsafe.go
type ArbitraryType int
type Pointer *ArbitraryType
```

三个函数：

```go
func Sizeof(x ArbitraryType) uintptr
func Offsetof(x ArbitraryType) uintptr
func Alignof(x ArbitraryType) uintptr
```

通过分析发现，这三个函数的参数均是ArbitraryType类型，就是接受任何类型的变量。

1. `Sizeof` **返回类型 x 所占据的字节数，但不包含 x 所指向的内容的大小**。例如，对于一个指针，函数返回的大小为 8 字节（64位机上），一个 slice 的大小则为 slice header 的大小。
2. `Offsetof`返回变量指定属性的偏移量，这个函数虽然接收的是任何类型的变量，但是有一个前提，就是变量要是一个struct类型，且还不能直接将这个struct类型的变量当作参数，只能将这个struct类型变量的属性当作参数。
3. `Alignof`返回变量对齐字节数量

## unsafe包的操作

### 大小Sizeof

unsafe.Sizeof函数返回的就是uintptr类型的值，表示所占据的字节数（表达式，即值的大小）：

```go
package main

import (
	"fmt"
	"reflect"
	"unsafe"
)

func main() {
	var a int32
	var b = &a
	fmt.Println(reflect.TypeOf(unsafe.Sizeof(a))) // uintptr
	fmt.Println(unsafe.Sizeof(a))                 // 4
	fmt.Println(reflect.TypeOf(b).Kind()) // ptr
	fmt.Println(unsafe.Sizeof(b)) // 8
}
```

对于 `a`来说，它是`int32`类型，在内存中占4个字节，而对于`b`来说，是`*int32`类型，即底层为`ptr`指针类型，在64位机下占8字节。

### 偏移Offsetof

对于一个结构体，通过 Offset 函数可以获取结构体成员的偏移量，进而获取成员的地址，读写该地址的内存，就可以达到改变成员值的目的。

这里有一个内存分配相关的事实：**结构体会被分配一块连续的内存，结构体的地址也代表了第一个字段的地址。**

举个例子：

```go
package main

import (
	"fmt"
	"unsafe"
)

type user struct {
	id   int32
	name string
	age  byte
}

func main() {
	var u = user{
		id:   1,
		name: "xiaobai",
		age:  22,
	}
	fmt.Println(u)
	fmt.Println(unsafe.Offsetof(u.id))   // 0  id在结构体user中的偏移量,也是结构体的地址
	fmt.Println(unsafe.Offsetof(u.name)) // 8
	fmt.Println(unsafe.Offsetof(u.age))  // 24

	// 根据偏移量修改字段的值 比如将id字段改为1001
	// 因为结构体的地址相当于第一个字段id的地址
	// 直接用unsafe包自带的Pointer获取id指针
	id := (*int)(unsafe.Pointer(&u))
	*id = 1001

	// 更加相对于id字段的偏移量获取name字段的地址并修改其内容
	// 需要用到uintptr进行指针运算 然后再利用unsafe.Pointer这个媒介将uintptr类型转换成一般的指针类型*string
	name := (*string)(unsafe.Pointer(uintptr(unsafe.Pointer(&u)) + unsafe.Offsetof(u.name)))
	*name = "花花"

	// 同理更改age字段
	age := (*int)(unsafe.Pointer(uintptr(unsafe.Pointer(&u)) + unsafe.Offsetof(u.age)))
	*age = 33

	fmt.Println(u)
}
```

### 对齐Alignof

要了解这个函数，你需要了解`数据对齐`。简单的说，它让数据结构在内存中以某种的布局存放，使该数据的读取性能能够更加的快速。

CPU 读取内存是一块一块读取的，块的大小可以为 2、4、6、8、16 字节等大小。块大小我们称其为内存访问粒度。

#### 普通字段的对齐值

```go
fmt.Printf("bool align: %d\n", unsafe.Alignof(bool(true)))
fmt.Printf("int32 align: %d\n", unsafe.Alignof(int32(0)))
fmt.Printf("int8 align: %d\n", unsafe.Alignof(int8(0)))
fmt.Printf("int64 align: %d\n", unsafe.Alignof(int64(0)))
fmt.Printf("byte align: %d\n", unsafe.Alignof(byte(0)))
fmt.Printf("string align: %d\n", unsafe.Alignof("EDDYCJY"))
fmt.Printf("map align: %d\n", unsafe.Alignof(map[string]string{}))
```

输出结果：

```text
bool align: 1
int32 align: 4
int8 align: 1
int64 align: 8
byte align: 1
string align: 8
map align: 8
```

在 Go 中可以调用 `unsafe.Alignof` 来返回相应类型的对齐系数。通过观察输出结果，可得知基本都是 2<sup>n</sup>，最大也`不会超过 8`。这是因为我们的**64位编译器默认对齐系数是 8**，因此最大值不会超过这个数。

#### 对齐规则

1. **结构体的成员变量**，第一个成员变量的偏移量为 0。往后的每个成员变量的`对齐值=min(编译器默认对齐长度,当前成员变量类型的长度)`。其`偏移量=对齐值xN`
2. **结构体本身**，`对齐值=最小整数倍{max(编译器默认对齐长度,结构体的所有成员变量类型中的最大长度)}`，

结合以上两点，可得知若编译器默认对齐长度超过结构体内成员变量的类型最大长度时，默认对齐长度是没有任何意义的

#### 结构体的对齐值

下面来看一下结构体的对齐：

```go
type part struct {
	a bool  // 1
	b int32 // 4
	c int8  // 1
	d int64 // 8
	e byte  // 1
}

func main() {
	var p part
	fmt.Println(unsafe.Sizeof(p)) // 32
}
```

按照普通字段（结构体内成员变量）的对齐方式，我们可以计算得出，这个结构体的大小占`1+4+1+8+1=15`个字节，但是用`unsafe.Sizeof`计算发现`part结构体`占`32`字节，是不是有点惊讶😮

这里面就涉及到了内存对齐，下面我们来分析一下：

| 成员变量   | 类型  | 偏移量 | 自身占用 |
| ---------- | ----- | ------ | -------- |
| a          | bool  | 0      | 1        |
| 数据对齐   | -     | 1      | 3        |
| b          | int32 | 4      | 4        |
| c          | int8  | 8      | 1        |
| 数据对齐   | -     | 9      | 7        |
| d          | in64  | 16     | 8        |
| e          | byte  | 24     | 1        |
| 数据对齐   | -     | 25     | 7        |
| 总占用大小 | -     | -      | 32       |

- 对于变量a而言

  类型是bool；大小/对齐值本身为1字节；偏移量为0，**占用了第0位**；此时内存中表示为`a`

- 对于变量b而言

  类型是int32；大小/对齐值本身为4字节；根据对齐规则一，偏移量必须为对齐值4的整数倍，故这里的偏移量为4，**占用了第4 ~ 7位**，则**第1 ~ 3位用padding字节填充**；此时内存中表示为`a---|bbbb`，(`|`只起到分隔作用，表示方便一些)

- 对于变量c而言

  类型是int8；大小/对齐值本身为1字节；当前偏移量为8，无需扩充，**占用了第8位**；此时内存中表示为`a---|bbbb|c`

- 对于变量d而言

  类型是int64；大小/对齐值本身为8字节；根据对齐规则一，偏移量必须为对齐值8的整数倍，故当前偏移量为16，**占用了第16 ~ 23位**，则**第9 ~ 15为用padding字节填充**；此时内存中表示为`a---|bbbb|c---|----|dddd|dddd`

- 对于变量e而言

  类型是byte；大小/对齐值本身为1字节；当前偏移量为24，无需扩充，**占用了第24位**；此时内存中表示为`a---|bbbb|c---|----|dddd|dddd|e`

这里计算后，发现总共占用25字节，哪里又来的32字节呢？😳 

再让我们回顾一下对齐原则的第二点，**结构体本身**，`对齐值=最小整数倍{max(编译器默认对齐长度,结构体的所有成员变量类型中的最大长度)}`

1. 这里编译器默认对齐长度为8字节(64位机)

2. 结构体中所有成员变量类型的最大长度为int64，8字节
3. 取二者最大数的最小整数倍作为对齐值，我们算的part结构体大小为25字节，不是8字节的整数倍，故还需要填充到32字节。

综上，part结构体在内存中表示为`a---|bbbb|c---|----|dddd|dddd|e----|----`

#### 扩展

让我们改变一下part结构体中字段的顺序看看（part结构体完全相同）

```go
type part struct {
	a bool  // 1
	c int8  // 1
	e byte  // 1
	b int32 //4
	d int64 // 8
}

func main() {
	var p part
	fmt.Println(unsafe.Sizeof(p)) // 16
}
```

这时候再用`unsafe.Sizeof`查看会发现，part结构体的内存占用只有`16`字节，瞬间减少了一半的内存空间，大家可以按照前面的步骤分析一下~

这里建议在构建结构体时，**按照字段大小的升序进行排序，会减少一点的内存空间**。

#### 反射包的对齐方法

反射包也有某些方法可用于计算对齐值：

```go
unsafe.Alignof(w)等价于reflect.TypeOf(w).Align
unsafe.Alignof(w.i)等价于reflect.Typeof(w.i).FieldAlign()
```

## 总结

- unsafe 包绕过了 Go 的类型系统，达到直接操作内存的目的，使用它有一定的风险性。但是在某些场景下，使用 unsafe 包提供的函数会提升代码的效率，Go 源码中也是大量使用 unsafe 包。

- unsafe 包定义了 Pointer 和三个函数：

  ```go
  type ArbitraryType int
  type Pointer *ArbitraryType
  
  func Sizeof(x ArbitraryType) uintptr
  func Offsetof(x ArbitraryType) uintptr
  func Alignof(x ArbitraryType) uintptr
  ```

  通过三个函数可以获取变量的大小、偏移、对齐等信息。

- uintptr 可以和 unsafe.Pointer 进行相互转换，uintptr 可以进行数学运算。这样，通过 uintptr 和 unsafe.Pointer 的结合就解决了 Go 指针不能进行数学运算的限制。

- 通过 unsafe 相关函数，可以获取结构体私有成员的地址，进而对其做进一步的读写操作，突破 Go 的类型安全限制。

  

> 参考：
>
> - [深度解密Go语言之unsafe](https://mp.weixin.qq.com/s/OO-kwB4Fp_FnCaNXwGJoEw)
> - [unsafe包的学习和使用 - 离地最远的星 - 博客园 (cnblogs.com)](https://www.cnblogs.com/hualou/p/12070155.html)
> - [在 Go 中恰到好处的内存对齐 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/53413177)