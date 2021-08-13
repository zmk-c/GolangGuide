# Go语言源码分析之interface

### Go语言中的接口类型

GO语言有接口类型（interface{}），但它与面向对象的接口含义不同，GO语言的接口类型与数组（array）、切片（slice）、集合（map）、结构体（struct）是同等地位的。怎么理解这句话呢？

我们知道：

```golang
var num int // 定义了一个int型变量num 
```

同理：go

```go
var any interface{} // 定义了一个接口类型变量
```

从这个角度上看，GO的interface{}与面向对象的接口是不一样的。interface{}是一个**任意类型**，或者说是万能类型。

也就是说定义一个变量为interface{}类型，可以把任意的值赋给这个变量，例如：

```go
var v1 interface{} = 250 // 把int值赋给interface{}

var v2 interface{} = "eagle" // 把string值赋给interface{}

var v3 interface{} = &v1 // 把v1的地址赋给interface{}
```

当然函数的入参类型也可以是interface{}，这样函数就可以接受任意类型的参数，例如GO语言标准库fmt中的函数Println()。

### 值接收者和指针接收者的区别

在调用方法的时候，值类型既可以调用`值接收者`的方法，也可以调用`指针接收者`的方法；指针类型既可以调用`指针接收者`的方法，也可以调用`值接收者`的方法。

也就是说，不管方法的接收者是什么类型，该类型的值和指针都可以调用，不必严格符合接收者的类型。

实现了接收者是值类型的方法，相当于自动实现了接收者是指针类型的方法；而实现了接收者是指针类型的方法，不会自动生成对应接收者是值类型的方法。**即：**实现了接收者是值类型的方法，会隐含地也实现了接收者是指针类型的方法。

如果**方法的接收者是值类型**，无论调用者是对象还是对象指针，修改的都是对象的`副本`，不影响调用者；如果**方法的接收者是指针类型**，则调用者修改的是指针指向的对象本身。

### interface的底层

`iface`描述的接口是包含方法的

```go
// go1.16.2 src/runtime/runtime2.go
type iface struct {
	tab  *itab				// 表示接口的类型以及赋给这个接口的实体类型
	data unsafe.Pointer		// 指向实际类型的实际值
}
```

```go
// go1.16.2 src/runtime/runtime2.go
type itab struct {
	inter *interfacetype 	// 实际类型实现的接口类型
	_type *_type 			// 描述实体的实际类型（包括内存堆区方式，大小等）
	hash  uint32 
	_     [4]byte
	fun   [1]uintptr 		// 放置和接口方法对应的实际类型的方法地址
}
```

```go
// go1.16.2 src/runtime/type.go
type interfacetype struct {
	typ     _type			// _type实际上是描述Go语言中各种数据类型的结构体
	pkgpath name			// 记录定义了接口的包名
	mhdr    []imethod 		// 表示接口所定义的函数列表
}
```

```go
// go1.16.2 src/runtime/type.go
type _type struct {
	size       uintptr		// 类型大小
	ptrdata    uintptr 		
	hash       uint32		// 类型的hash
	tflag      tflag		// 类型的flag,和反射相关
	align      uint8		// 内存对齐相关
	fieldAlign uint8
	kind       uint8		// 类型的编号，有bool,slice,struct等
	equal func(unsafe.Pointer, unsafe.Pointer) bool
	gcdata    *byte			// 和gc相关
	str       nameOff
	ptrToThis typeOff
}
```

iface类型的整体图：

![图片](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/iface.webp)



`eface`描述的则是不包含任何方法的空口**interface{}**

```go
// go1.16.2 src/runtime/runtime2.go
type eface struct {
	_type *_type // 实际类型
	data  unsafe.Pointer // 实际类型的实际值
}
```

接口值的零值是指`动态类型`和`动态值`都为 `nil`。当仅且当这两部分的值都为 `nil` 的情况下，这个接口值就才会被认为 `接口值 == nil`





>Reference:
>
>1. [深度解密Go语言之关于 interface 的 10 个问题](https://mp.weixin.qq.com/s/EbxkBokYBajkCR-MazL0ZA)
