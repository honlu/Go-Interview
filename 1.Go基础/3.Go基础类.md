# go基础类

### 1. go优势

```markdown
* 天生支持并发，性能高
* 单一的标准代码格式，比其它语言更具可读性
* 自动垃圾收集比java和python更有效，因为它与程序同时执行
```

### 2. go数据类型

```go
int string float bool array slice map channel pointer struct interface method
```

### 3. go程序中的包是什么

```go
* 项目中包含go源文件以及其它包的目录，源文件中的函数、变量、类型都存储在该包中
* 每个源文件都属于一个包，该包在文件顶部使用 package packageName 声明
* 我们在源文件中需要导入第三方包时需要使用 import packageName
```

### 4. go支持什么形式的类型转换？将整数转换为浮点数

```css
* go支持显示类型转换，以满足严格的类型要
* a := 15
* b := float64(a)
* fmt.Println(b, reflect.TypeOf(b))
```

### 5. 什么是 goroutine，你如何停止它？

~~~csharp
* goroutine是协程/轻量级线程/用户态线程，不同于传统的内核态线程
* 占用资源特别少，创建和销毁只在用户态执行不会到内核态，节省时间
* 创建goroutine需要使用go关键字
* 可以向goroutine发送一个信号通道来停止它，goroutine内部需要检查信号通道
例子：
```
func main() {
    var wg sync.WaitGroup // 等待组进行多个任务的同步，可以保证并发环境中完成指定数量的任务，每个sync.WaitGroup值在内部维护着一个计数，此计数的初始默认值为0
    var exit = make(chan bool)
    wg.Add(1) // 等待组的计数器+1
    go func() {
        for {
            select {
            case <-exit:  // 接收到信号后return退出当前goroutine
                fmt.Println("goroutine接收到信号退出了！")
                wg.Done() // 等待组的计数器-1
                return
            default:
                fmt.Println("还没有接收到信号")
            }
        }
    }()
    exit <- true
    wg.Wait() // 当等待组计数器不等于0时阻塞，直到变为0
}
```
~~~

### 6. 如何在运行时检查变量类型

```dart
* 类型开关(Type Switch)是在运行时检查变量类型的最佳方式。
* 类型开关按类型而不是值来评估变量。每个 Switch 至少包含一个 case 用作条件语句
* 如果没有一个 case 为真，则执行 default。
```

### 7. go两个接口之间可以存在什么关系

```css
* 如果两个接口有相同的方法列表，那么他俩就是等价的，可以相互赋值
* 接口A可以嵌套到接口B里面，那么接口B就有了自己的方法列表+接口A的方法列表
```

### 8. go中同步锁（也叫互斥锁）有什么特点，作用是什么？何时使用互斥锁，何时使用读写锁？

~~~scss
* 当一个goroutine获得了Mutex（互斥锁）后，其它goroutine就只能乖乖等待，除非该goroutine释放Mutex
* RWMutext（读写互斥锁）在读锁占用的情况下会阻止写，但不会阻止读，在写锁占用的情况下，会阻止任何其它goroutine进来
* 无论是读还是写，整个锁相当于由该goroutine独占
* 作用：保证资源在使用时的独有性，不会因为并发导致数据错乱，保证系统稳定性
* 案例：
``` 
package main
import (
    "fmt"
    "sync"
    "time"
)
var (
    num = 0
    lock = sync.RWMutex{}  // 耗时：100+毫秒
    //lock = sync.Mutex{}  // 耗时：50+毫秒
)
func main() {
    start := time.Now()
    go func() {
        for i := 0; i < 100000; i++{
            lock.Lock()
            //fmt.Println(num)
            num++
            lock.Unlock()
        }
    }()
    for i := 0; i < 100000; i++{
        lock.Lock()
        //fmt.Println(num)
        num++
        lock.Unlock()
    }
    fmt.Println(num)
    fmt.Println(time.Now().Sub(start))
}
```
// 结论：
// 1. 如果对数据写的比较多，使用Mutex同步锁/互斥锁性能更高
// 2. 如果对数据读的比较多，使用RWMutex读写锁性能更高
~~~

### 9. goroutine案例（两个goroutine，一个负责输出数字，另一个负责输出26个英文字母，格式如下：12ab34cd56ef78gh ... yz）

```go
package main
import (
	"fmt"
	"sync"
	"unicode/utf8"
)
// 案例：两个goroutine，一个负责输出数字，另一个负责输出26个英文字母，格式如下：12ab34cd56ef78gh ... yz
var (
	wg = sync.WaitGroup{}  // 和第五题很相关。申明等待组
	chNum = make(chan bool)
	chAlpha = make(chan bool)
)
func main() {
	go func() {
		i := 1
		for {
			<-chNum // 接到信号，运行该goroutine
			fmt.Printf("%v%v", i, i + 1)
			i += 2
			chAlpha <- true // 发送信号
		}
	}()
	wg.Add(1) // 等待组的计数器+1
	go func() {
		str := "abcdefghigklmnopqrstuvwxyz"
		i := 0
		for {
			<-chAlpha // 接到信号，运行该goroutine
			fmt.Printf("%v", str[i:i+2])
			i += 2
			if i >= utf8.RuneCountInString(str){
				wg.Done() // 等待组的计数器-1
				return
			}
			chNum <- true // 发送信号
		}
	}()
	chNum <- true // 发送信号
	wg.Wait() // 等待组的计数器不为0时，阻塞main进程，直到等待组的计数器为0
}
```

### 10. go语言中，channel通道有什么特点，需要注意什么？

- 结论：
  1. 给一个nil channel发送数据时会一直堵塞
  2. 从一个nil channel接收数据时会一直阻塞
  3. 给一个已关闭的channel发送数据时会panic
  4. 从一个已关闭的channel中读取数据时，如果channel为空，则返回通道中类型的零值
- 案例

```go
package main
import (
	"fmt"
	"sync"
)
func main() {
	var wg sync.WaitGroup // 等待组
	var ch chan int // nil channel
	var ch1 = make(chan int) // 创建channel
	fmt.Println(ch, ch1)  // <nil> 0xc000086060
	wg.Add(1) // 等待组的计数器+1
	go func() {
		//ch <- 15  // 如果给一个nil的channel发送数据会造成永久阻塞
		//<-ch  // 如果从一个nil的channel中接收数据也会造成永久阻塞
		ret := <-ch1
		fmt.Println(ret)
		ret = <-ch1  // 从一个已关闭的通道中接收数据，如果缓冲区中为空，则返回该类型的零值
		fmt.Println(ret)
		wg.Done() // 等待组的计数器-1
	}()
	go func() {
		//close(ch1)
		ch1 <- 15  // 给一个已关闭通道发送数据就会包panic错误
		close(ch1)
	}()
	wg.Wait() // 等待组的计数器不为0时阻塞
}
```

### 11. go中channel缓冲有什么特点？

- 无缓冲的通道是同步的，有缓冲的通道是异步的

### 12. go中的cap函数可以作用于哪些内容？

- 可作用于的类型有：
  1. 数组（array）
  2. 切片（slice）
  3. 通道（channel）
- 查看他们的容量大小，而不是装的数据大小

### 13. go convey是什么，一般用来做什么？

1. go convey是一个**支持golang的单元测试框架**
2. 能够自动监控文件修改并启动测试，并可以将测试结果实时输出到web界面
3. 提供了丰富的断言简化测试用例的编写

### 14. go语言中new的作用是什么？

1. 使用new函数来**分配内存空间**
2. **传递给new函数的是一个类型，而不是一个值**
3. **返回值是指向这个新分配的地址的指针**

### 15. go语言中的make作用是什么？

- 分配**内存空间并进行初始化**, 返回值是**该类型的实例而不是指针**
- make**只能接收三种类型当做参数：slice、map、channel**

### 16. 总结new和make的区别？

1. new可以**接收任意内置类型当做参数**，返回的是**对应类型的指针**
2. make**只能接收slice、map、channel当做参数**，返回值是**对应类型的实例**

### 17. Printf、Sprintf、FprintF都是格式化输出，有什么不同？

- 虽然这三个函数都是格式化输出，但是输出的目标不一样

  1. Printf输出到控制台
  2. Sprintf结果赋值给返回值
  3. FprintF输出到指定的io.Writer接口中
     例如：

  ```go
  func main() {
      var a int = 15
      file, _ := os.OpenFile("test.log", os.O_CREATE|os.O_APPEND, 0644)
      // 格式化字符串并输出到文件
      n, _ := fmt.Fprintf(file, "%T:%v:%p", a, a, &a)
      fmt.Println(n)
  }
  ```

### 18. go语言中的数组和切片的区别是什么？

- 数组：
  1. 数组**固定长度**，数组长度是数组类型的一部分，所以[3]int和[4]int是两种不同的数组类型
  2. 数组类型**需要指定大小**，不指定也会根据初始化，自动推算出大小，大小不可改变，数组是通过值传递的
- 切片：
  1. 切片的**长度可改变**，切片是轻量级的数据结构，三个属性：指针、长度、容量
  2. **不要指定切片的大小**，切片也是值传递只不过切片的一个属性指针指向的数据不变，所以看起来像引用传递
  3. 切片**可以通过数组来初始化也可以通过make函数来初始化**，**初始化时的len和cap相等，然后进行扩容**
  4. 切片**扩容的时候会导致底层的数组复制**，也就是切片中的指针属性会发生变化
  5. 切片也是拷贝，在不发生扩容时，底层使用的是同一个数组，当对其中一个切片append的时候, 该切片长度会增加
     但是不会影响另外一个切片的长度
  6. copy函数将原切片拷贝到目标切片，会导致底层数组复制，因为目标切片需要通过make函数来声明初始化内存，然后
     将原切片指向的数组元素拷贝到新切片指向的数组元素
- 重点：**数组保存真正的数据**，**切片值保存数组的指针和该切片的长度和容量**
- append函数如果切片容量足够的话，只会影响当前切片的长度，数组底层不会复制，不会影响与数组关联的其它切片的长度
- copy直接会导致数组底层复制

### 19. go语言中值传递和地址传递（引用传递）如何运行？有什么区别？举例说明

1. 值传递会把参数的值复制一份放到对应的函数里，两个变量的地址不同，不可互相修改
2. 地址传递会把参数的地址复制一份放到对应的函数里，两个变量的地址相同，可以互相修改
3. 例如：**数组传递就是值传递**，而**切片传递就是数组的地址传递**（本质上切片值传递，只不过是保存的数据地址相同）

### 20. go中数组和切片在传递时有什么区别？

1. 数组是值传递
2. 切片地址传递（引用传递）

### 21. go中是如何实现切片扩容的？

1. 当容量小于1024时，每次扩容容量翻倍，当容量大于1024时，每次扩容加25%

```go
func main() {
  s1 := make([]int, 0)
  for i := 0; i < 3000; i++{
  	fmt.Println("len =", len(s1), "cap = ", cap(s1))
  	s1 = append(s1, i)
  }
  }
```

### 22. 看下面代码defer的执行顺序是什么？defer的作用和特点是什么？

1. 在普通函数或方法前加上defer关键字，就完成了defer所需要的语法，**当defer语句被执行时，跟在defer语句后的函数会被延迟执行**
2. 知道**包含该defer语句的函数执行完毕，defer语句后的函数才会执行**，无论包含defer语句的函数是通过return正常结束，还是通过panic导致的异常结束
3. 可以在一个函数中执行多条defer语句，由于在栈中存储，所以它的**执行顺序和声明顺序相反**

### 23. defer语句中通过recover捕获panic例子

注意要在defer后函数里的recover()

```go
func main() {
	defer func() {
		err := recover()
		fmt.Println(err)
	}()
	defer fmt.Println("first defer")
	defer fmt.Println("second defer")
	defer fmt.Println("third defer")
	fmt.Println("哈哈哈哈")
	panic("abc is an error")
}
```

### 24. go中的25个关键字

- 程序声明2个：
  package import
- 程序实体声明和定义8个：
  var const type func struct map chan interface
- 程序流程控制15个：
  for range continue break select switch case default if else fallthrough defer go goto return

### 25. 写一个定时任务，每秒执行一次

```go
func main() {
  t1 := time.NewTicker(time.Second * 1) // 创建一个周期定时器
  var i = 1
  for {
  	if i == 10{
  		break
  	}
  	select {
  	case <-t1.C:  // 一秒执行一次的定时任务
  		task1(i)
  		i++
  	}
  }
}
func task1(i int) {
    fmt.Println("task1执行了---", i)
}
```

### 26. switch case fallthrough default使用场景

```go
func main() {
	var a int
	for i := 0; i < 10; i++{
		a = rand.Intn(100)
		switch {
		case a >= 80:
			fmt.Println("优秀", a)
			fallthrough // 强制执行下一个case
		case a >= 60:
			fmt.Println("及格", a)
			fallthrough
		default:
			fmt.Println("不及格", a)
		}
	}
}
```

### 27. defer的常用场景

- defer语句经常被**用于处理成对的操作打开/关闭，链接/断开连接，加锁/释放锁**
- 通过defer机制，不论函数逻辑多复杂，都**能保证在任何执行路径下，资源被释放**
- 释放资源的defer语句应该直接跟在请求资源处理错误之后
- 注意：defer一定要放在请求资源处理错误之后

### 28. go中slice的底层实现

1. 切片是基于数组实现的，它的底层是数组，它本身非常小，它可以理解为对底层数组的抽闲
2. 因为基于数组实现，所以它的底层内存是连续分配的，效率非常高，还可以通过索引获取数据
3. 切片本身并不是动态数组或数组指针，它内部实现的数据结构体通过指针引用底层数组
4. 设定相关属性将读写操作限定在指定的区域内，切片本身是一个只读对象，其工作机制类似于数组指针的一种封装
5. 切片对象非常小，因为它只有三个字段的数据结构：指向底层数组的指针、切片的长度、切片的容量

### 29. go中slice的扩容机制，有什么注意点？

1. 首先判断，如果新申请的容量大于2倍的旧容量，最终容量就是新申请的容量
2. 否则判断，如果旧切片的长度小于1024，最终容量就是旧容量的两倍
3. 否则判断，如果旧切片的长度大于等于1024，则最终容量从旧容量开始循环增加原来的1/4,直到最终容量大于新申请的容量
4. 如果最终容量计算值溢出，则最终容量就是新申请的容量

### 30. 扩容前后的slice是否相同？

- 情况一：
  1. **原来数组还有容量可以扩容（实际容量没有填充完）**，这种情况下，扩容之后的切片还是指向原来的数组
  2. 对一个切片的操作可能影响多个指针指向相同地址的切片
- 情况二：
  1. 原来数组的容量已经达到了最大值，在扩容，go默认会先开辟一块内存区域，把原来的值拷贝过来
  2. 然后再执行append操作，这种情况丝毫不影响原数组
- 注意：要复制一个slice最好使用copy函数

### 31. go中的参数传递、引用传递

1. go语言中的**所有的传参都是值传递（传值），都是一个副本，一个拷贝**，
2. 因为拷贝的内容有时候是非引用类型（int, string, struct）等，这样在函数中就无法修改原内容数据
3. 有的是引用类型（指针、slice、map、chan），这样就可以修改原内容数据

- go中的引用类型包含slice、map、chan,它们有复杂的内部结构，除了申请内存外，还需要初始化相关属性
- 内置函数new计算类型大小，为其分配零值内存，返回指针。
- 而make会被编译器翻译成具体的创建函数，由其分配内存并初始化成员结构，返回对象而非指针

### 32. 哈希概念讲解

1. 哈希表又称为**散列表，由一个直接寻址表和一个哈希函数组成**

2. 由于哈希表的大小是有限的而要存储的数值是无限的，因此对于任何哈希函数,都会出现两个不同元素映射到相同位置的情况，这种情况叫做哈希冲突

4. 通过**拉链法解决哈希冲突**：
   \* 哈希表每个位置都连接一个链表，当冲突发生是，冲突的元素将会被加到该位置链表的最后
   
4. 哈希表的查找速度起决定性作用的就是**哈希函数**: **除法哈希发、乘法哈希法、全域哈希法**

5. 哈希表的应用？

   字典与集合都是通过哈希表来实现的

   md5曾经是密码学中常用的哈希函数，可以把任意长度的数据映射为128位的哈希值

### 33. go中的map底层实现

1. go中map的底层实现就是一个散列表，因此实现map的过程实际上就是实现散列表的过程
2. 在这个散列表中，**主要出现的结构体由两个，一个是hmap、一个是bmap**
3. go中也有一个哈希函数，用来对map中的键生成哈希值
4. hash结果的低位用于把k/v放到bmap数组中的哪个bmap中
5. 高位用于key的快速预览，快速试错

### 34. go中的map如何扩容

1. 翻倍扩容：如果map中的键值对个数/桶的个数>6.5，就会引发翻倍扩容
2. 等量扩容：当B<=15时，如果溢出桶的个数>=2的B次方就会引发等量扩容
3. 当B>15时，如果溢出桶的个数>=2的15次方时就会引发等量扩容

### 35. go中map的查找

1. go中的map采用的是哈希查找表，由哈希函数通过key和哈希因此计算出哈希值，
2. 根据hamp中的B来确定放到哪个桶中，如果B=5，那么就根据哈希值的后5位确定放到哪个桶中
3. 在用哈希值的高8位确定桶中的位置，如果当前的bmap中未找到，则去对应的overflow bucket中查找
4. 如果当前map处于数据搬迁状态，则优先从oldbuckets中查找

### 36. 介绍一下channel

1. go中不要通过共享内存来通信，而要**通过通信实现共享内存**
2. go中的**csp并发模型，中文名通信顺序进程，就是通过goroutine和channel实现的**
3. **channel收发遵循先进先出，分为有缓冲通道（异步通道），无缓冲通道（同步通道）**

### 37. go中channel的特性

1. 给一个nil的channel发送数据，会造成永久阻塞
2. 从一个nil的channel接收数据，会造成永久阻塞
3. 给一个已经关闭的channel发送数据，会造成panic
4. 从一个已经关闭的channel接收数据，如果缓冲区为空，会返回零值
5. 无缓冲的channel是同步的，有缓冲的channel是异步的
6. 关闭一个nil channel会造成panic

### channel中ring buffer的实现

1. channel中使用了ring buffer（环形缓冲区）来缓存写入数据，
2. ring buffer有很多好处，而且非常适合实现FiFo的固定长度队列
3. channel中包含buffer、sendx、recvx
4. recvx指向最早被读取的位置，sendx指向再次写入时插入的位置

来源：[go面试题-基础类 - 专职 - 博客园 (cnblogs.com)](https://www.cnblogs.com/mayanan/p/15836710.html)