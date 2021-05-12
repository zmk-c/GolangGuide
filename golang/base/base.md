# Go语言基础知识细节

1. 使用表达式`new(Type)`将创建一个Type类型的匿名变量，初始化为`Type类型的零值`，然后返回`变量的地址，返回的指针类型为*Type`

   **面试题：new与make的区别**

   > new 的作用是初始化一个指向类型的指针(*T)
   >
   > new函数是内建函数，函数定义：`func new(Type) *Type`
   >
   > 使用new函数来分配空间。传递给`new` 函数的是一个**类型**，不是一个值。返回值是指向这个新分配的零值的**指针**。
   >
   > make 的作用是为 slice，map 或 chan 初始化并返回引用(T)。
   >
   > make函数是内建函数，函数定义：`func make(Type, size IntegerType) Type`
   >
   > · 第一个参数是一个类型，第二个参数是长度
   >
   > · 返回值是一个类型
   >
   > `make(T, args)`函数的目的与`new(T)`不同。它仅仅用于创建 Slice, Map 和 Channel，并且返回类型是 T（不是*T）的一个初始化的（不是零值）的实例

2. `byte`和`uint8`没有区别，`rune`和`uint32`没有区别。因为uint8和uint32直观上让人以为这是一个数值，但是实际上，它也可以表示一个`字符`，所以为了消除这种直观上的错觉，就诞生了byte和rune这两个类型的别名。且对于`rune`关键字来说，其表示范围(-2<sup>31</sup>到2<sup>31-1</sup>)，比byte(-128到127)可以表示更多的字符，特别是中文字符。

   **面试题：翻转含有中文，数字，英文字母的字符串**

   ```go
   package main
   import "fmt"
   
   func reverse(s []rune) string {
   	for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
   		s[i], s[j] = s[j], s[i]
   	}
   	return string(s)
   }
   
   func main() {
   	var src = "aa张111"
   	dst := reverse([]rune(src))
   	fmt.Println(dst)
   }
   ```

3. 对于map来说，读取元素，直接使用`map[key]`即可，如果key不存在，系统不会报错，会返回其`value-type的零值`

4. 使用`delete`函数删除map元素，如果key不存在，`delete函数会静默处理`，不会报错

5. 使用`defer只是延时调用函数`，此时传递给函数里的变量，不应该受到后续程序的影响，相当于在defer处对变量做了`快照`，多个defer的调用顺序类似`栈`，且`defer在return之后调用`

   **面试题：defer和return的调用顺序**

   ```go
   package main
   
   import "fmt"
   
   var name = "go"
   
   func getName() string {
   	defer func() {
   		name = "js"
   	}()
   
   	fmt.Printf("getName函数中的name值为%v \n", name) // go
   
   	return name
   }
   
   func main() {
   	myname := getName()
   	fmt.Printf("main函数中的name为%v \n", name)     // js
   	fmt.Printf("main函数中的myname为%v \n", myname) // go
   }
   ```

6. `recover`的使用必须在defer函数中才能生效

7. `panic`会导致整个程序退出，但是在退出前，若有defer延迟函数，还得执行完defer。且`defer在多个协程之间没有效果`，在子协程里触发panic，只能触发自己协程内的defer，而不能调用main协程里的defer函数

   ```go
   package main
   
   import (
   	"fmt"
   	"time"
   )
   
   func main() {
   	//这个defer不会执行
   	defer fmt.Println("in main")
   
   	//子协程
   	go func() {
   		//这个defer还得执行完
   		defer fmt.Println("in goroutine")
   		panic("出现错误")
   	}()
   	
       //给子协程留出能执行的时间
   	time.Sleep(2 * time.Second)
   }
   
   ```

8. 在golang中，`接口就是方法签名的集合`，当一个类型定义了接口中的`所有方法`，我们称它实现了该接口，接口指定了一个类型应该具有的方法，并由该类型决定如何实现这些方法。一个接口（老师）下，在不同对象（人）上的不同表现，就是多态。

9. 根据`变量声明的位置不同`，作用域可以分为以下四个类型：

   - `内置作用域`：不需要自己声明，所有的关键字和内置类型、函数都拥有全局作用域
   - `包级作用域`：必须函数外声明，在该包所有文件内都可以访问
   - `文件级作用域`：不需要声明，导入即可。一个文件中通过import导入的包名，只能在该文件内使用
   - `局部作用域`：在自己的语句块内声明，包括函数、for、if等语句块，或自定义的{}语句块形成的作用域

   以上四种作用域，从上往下，范围从大到小，为了表述方便，这里将范围大的作用域称为高层作用域，范围小的作用域称为低层作用域。总结几点：

   - 低层作用域可以访问高级作用域
   - 同一层级的作用域是相互隔离的
   - 低层作用域里声明的变量会覆盖高层作用域里面声明的变量

10. `缓冲信道`允许信道里存储一个或多个数据，这意味着，设置了缓冲区后，发送端和接收端可以处于`异步`状态

11. `无缓冲信道`在信道里无法存储数据，这意味着，`接收端必须先于发送端准备好`，以确保你发送完数据后立马有人接收，否则会造成发送端堵塞，原因就是信道中无法存储数据，即发送端和接收端是`同步`进行的

12. 遍历信道，可以使用for搭配range关键字，在range时，要确保信道是否处于关闭状态，否则循环会阻塞

13. `不要通过共享内存来通信，要通过通信来共享内存`

14. `sync.WaitGroup类型`中的方法：

    - `Add`：初始值为0，传入的值会往计数器上加，这里一般传入子协程的数量
    - `Done`：当某个子协程完成后，可调用此方法，会从计数器上减一，通常可以使用defer调用
    - `Wait`：阻塞当前协程，直到实例里的计数器归零，一般放在main方法中阻塞，直到所有的子协程全部调用完毕

    ```go
    package main
    
    import (
    	"fmt"
    	"sync"
    )
    
    var wg sync.WaitGroup
    
    func worker(x int) {
    	//计数器减一
    	defer wg.Done()
    
    	for i := 0; i < 5; i++ {
    		fmt.Printf("worker %d 在做第%d份工作 \n", x, i)
    	}
    }
    
    func main() {
    	//添加子协程数量 为2
    	wg.Add(2)
    
    	go worker(1)
    	go worker(2)
    
    	//阻塞main协程  直到所有的子协程全部执行完毕
    	wg.Wait()
    }
    ```

15. 面对并发问题，我们始终应该优先考虑使用信道，如果通过信道解决不了，不得不使用共享内存来实现并发编程的，那么就需要使用golang中的`锁机制`了。golang中的锁分为两种：一个叫`Mutex`，利用它可以实现`互斥锁`，另一个叫`RWMutex`，利用它可以实现`读写锁`。其中RWMutex将程序对资源的访问分为`读操作`和`写操作`，为了保证数据的安全，它规定了当有人还在读取数据（即读锁占用）时，不允许有人更新这个数据（即写锁会阻塞）为了保证程序的效率，多个人（线程）读取数据（拥有读锁）时，互不影响，不会造成阻塞，它不会像Mutex那样只允许有一个人（线程）读取同一个数据
