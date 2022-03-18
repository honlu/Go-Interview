# 基础语法

### 1.使用值为 nil 的 slice、map会发生啥

允许对值为 nil 的 slice 添加元素，但对值为 nil 的 map 添加元素，则会造成运行时 panic。

```GO
// map 错误示例
func main() {
    var m map[string]int
    m["one"] = 1  // error: panic: assignment to entry in nil map
    // m := make(map[string]int)// map 的正确声明，分配了实际的内存,这样添加元素就不会错
}    

// slice 正确示例
func main() {
    var s []int
    s = append(s, 1)
}
```

### 2.访问 map 中的 key，需要注意啥

当访问 map 中不存在的 key 时，Go 则会返回元素对应数据类型的零值，比如 nil、’’ 、false 和 0，取值操作总有值返回，故**不能通过取出来的值，来判断 key 是不是在 map 中**。

检查 key 是否存在可以**用 map 直接访问，检查返回的第二个参数**即可。

```go
// 错误的 key 检测方式
func main() {
    x := map[string]string{"one": "2", "two": "", "three": "3"}
    if v := x["two"]; v == "" {
      	fmt.Println("key two is no entry") // 键 two 存不存在都会返回的空字符串
    }
}

// 正确示例
func main() {
    x := map[string]string{"one": "2", "two": "", "three": "3"}
    if _, ok := x["two"]; !ok {
      	fmt.Println("key two is no entry")
    }
}
```

### 3.string 类型的值可以修改吗

不能，**尝试使用索引遍历字符串，来更新字符串中的个别字符，是不允许的**。

string 类型的值是只读的二进制 byte slice，如果**真要修改字符串中的字符，将 string 转为 []byte 修改后，再转为 string 即可**。

```go
// 修改字符串的错误示例
func main() {
   x := "text"
   x[0] = "T"  // error: cannot assign to x[0]
   fmt.Println(x)
}


// 修改示例
func main() {
   x := "text"
   xBytes := []byte(x)
   xBytes[0] = 'T' // 注意此时的 T 是 rune 类型
   x = string(xBytes)
   fmt.Println(x) // Text
}
```

### 4.switch 中如何强制执行下一个 case 代码块

switch 语句中的 case 代码块会默认带上 break，但可以**使用 fallthrough 来强制执行下一个 case 代码**块。

```go
func main() {
   isSpace := func(char byte) bool {
    switch char {
    case ' ': // 空格符会直接 break，返回 false // 和其他语言不一样
      // fallthrough // 返回 true
    case '\t':
       return true
    }
    return false
	 }
   fmt.Println(isSpace('\t')) // true
   fmt.Println(isSpace(' ')) // false
}
```

### 5.如何从 panic 中恢复

在一个 **defer 延迟执行的函数中调用 recover** ，它便能捕捉/中断 panic。这是因为即使panic，也会继续执行完goroutine，而defer延迟执行的函数中含有recover，所以会恢复。

```go
// 错误的 recover 调用示例
func main() {
    recover() // 什么都不会捕捉
    panic("not good") // 发生 panic，主程序退出
    recover() // 不会被执行
    println("ok")
}

// 正确的 recover 调用示例
func main() {
    defer func() {
      fmt.Println("recovered: ", recover())
    }()
    panic("not good")
}
```

### 6.简短声明(:=)的变量需要注意啥

- 简短声明的变量**只能在函数内部使用**
- **struct 的变量字段不能使用 := 来赋值**
- 不能用简短声明方式来单独为一个变量重复声明， **:= 左侧至少有一个新变量**，才允许多变量的重复声明

### 7.range 迭代 map是有序的吗

无序的。Go 的运行时是有意打乱迭代顺序的，所以你得到的迭代结果可能不一致。但也并不总会打乱，得到连续相同的 5 个迭代结果也是可能的。

若**想有序遍历map,将 `Map` 中的 key 拿出来，放入 `slice` 中做排序.**

### 8.recover的执行时机

无，recover 必须在 defer 函数中运行。recover 捕获的是祖父级调用时的异常，直接调用时无效。

```go
func main() { // 无效
    recover()
    panic(1)
}
```

直接 defer 调用也是无效。

```go
func main() {
    defer recover() // 直接调用，无效
    panic(1)
}
```

defer 调用时多层嵌套依然无效。

```go
func main() {
    defer func() {  // 多层嵌套无效
        func() { recover() }()
    }()
    panic(1)
}
```

必须在 defer 函数中直接调用才有效。

```go
func main() {
    defer func() { // 执行中的函数调用，有效
        recover()
    }()
    panic(1)
}
```

### 9.闭包错误引用同一个变量问题怎么处理

在**每轮迭代中生成一个局部变量 i** 。如果没有 i := i 这行，将会打印同一个变量。

```go
func main() {
    for i := 0; i < 5; i++ {
        i := i // 再次新的局部变量
        defer func() {
            println(i)
        }()
    }
}
```

或者是**通过函数参数传入 i** 。

```go
func main() {
    for i := 0; i < 5; i++ {
        defer func(i int) { // 参数传递
            println(i)
        }(i)
    }
}
```

### 10.在循环内部执行defer语句会发生啥

defer 在函数退出时才能执行，在 for 执行 defer 会**导致资源延迟释放**。

```go
func main() {
    for i := 0; i < 5; i++ {
        func() {
            f, err := os.Open("/path/to/file")
            if err != nil {
                log.Fatal(err)
            }
            defer f.Close()
        }()
    }
}
```

**func 是一个局部函数，在局部函数里面执行 defer 将不会有问题**。因为当**函数执行结束后，defer 语句出栈，遵循先入后出**。

### 11.如何跳出for select 循环

通常在for循环中，使用break可以跳出循环，但是注意在go语言中，**for select配合时，break 并不能跳出循环**。因为**默认在select中break是只跳脱了select体**，而不是结束for循环。需要设置标签，**break标签或goto便签即可跳出循环**！但要注意标签和便签位置不一样！还可以return，适合退出goroutine的场景。

```go
func testSelectFor2(chExit chan bool){
    EXIT:// 标签
    for  {
        select {
          case v, ok := <-chExit:
          if !ok {
              fmt.Println("close channel 2", v)
              break EXIT//goto EXIT2
          }

          fmt.Println("ch2 val =", v)
        }
    }

    //EXIT2://goto的便签，注意位置不同，如果在标签位置，goto仍然不能退出for循环！
    fmt.Println("exit testSelectFor2")
}
```

### 12.如何在切片中查找

go中使用 sort.searchXXX 方法，**在排序好的切片**中查找指定的方法，但是其**返回是对应的查找元素不存在时，待插入的位置下标(元素插入在返回下标前)**。

可以通过封装如下函数，达到目的。

```go
func IsExist(s []string, t string) (int, bool) {
    iIndex := sort.SearchStrings(s, t)
    bExist := iIndex!=len(s) && s[iIndex]==t // 待插入索引不等于长度且待插入位置正好存在要查找的元素，则返回存在。

    return iIndex, bExist
}
```

### 13.如何初始化带嵌套结构的结构体

go 的哲学是**组合优于继承，使用 struct 嵌套即可完成组合**，内嵌的结构体属性就像外层结构的属性即可，可以直接调用。

注意初始化外层结构体时，**必须指定内嵌结构体名称的结构体初始化**，如下看到 s1方式报错，s2 方式正确。

```go
type stPeople struct {
    Gender bool
    Name string
}

type stStudent struct {
    stPeople
    Class int
}

//尝试4 嵌套结构的初始化表达式
//var s1 = stStudent{false, "JimWen", 3}
var s2 = stStudent{stPeople{false, "JimWen"}, 3} // 指定内嵌结构体的名称
fmt.Println(s2.Gender, s2.Name, s2.Class)
```

### 14.切片和数组的区别

**数组是具有固定长度，且拥有零个或者多个，相同数据类型元素的序列。**数组的长度是数组类型的一部分，所以[3]int 和 [4]int 是两种不同的数组类型。数组**需要指定大小**，不指定也会根据初始化的自动推算出大小，不可改变；**数组是值传递**。数组是内置类型，是一组同类型数据的集合，它是值类型，通过从0开始的下标索引访问元素值。在初始化后长度是固定的，无法修改其长度。

当**作为方法的参数传入时将复制一份数组而不是引用同一指针。**数组的长度也是其类型的一部分，通过内置函数len(array)获取其长度。数组定义：

```go
var array [10]int

var array =[5]int{1,2,3,4,5}
```

**切片表示一个拥有相同类型元素的可变长度的序列**。切片是一种轻量级的数据结构，它有三个属性：指针、长度和容量。**切片不需要指定大小；切片是地址传递；切片可以通过数组来初始化，也可以通过内置函数make()初始化 。**初始化时len=cap,在追加元素时如果容量cap不足时将按len的2倍扩容。切片定义：

```go
var slice []type = make([]type, len)
```

### 15.new和make的区别

**new 的作用是初始化一个指向类型的指针 (*T)** 。new 函数是内建函数，函数定义：func new(Type) *Type。使用 new 函数来分配空间。传递给 new 函数的是一个类型，不是一个值。**返回值是指向这个新分配的零值的指针**。

**make 的作用是为 slice，map 或 chan 初始化并返回引用 (T)**。make 函数是内建函数，函数定义：func make(Type, size IntegerType) Type；第一个参数是一个类型，第二个参数是长度；**返回值是一个类型**。

make(T, args) 函数的目的与 new(T) 不同。它仅仅用于创建 Slice, Map 和 Channel，并且返回类型是 T（不是T*）的一个初始化的（不是零值）的实例。

### 16.Printf()、Sprintf()、Fprintf()函数的区别用法是什么

都是把格式好的字符串输出，只是输出的目标不一样。

Printf()，是**把格式字符串输出到标准输出**（一般是屏幕，可以重定向）。Printf() 是和标准输出文件 (stdout) 关联的，Fprintf 则没有这个限制。
Sprintf()，是**把格式字符串输出到指定字符串中**，所以参数比printf多一个char*。那就是目标字符串地址。

Fprintf()，是**把格式字符串输出到指定文件设备中**，所以参数比 printf 多一个文件指针 FILE*。主要用于文件操作。Fprintf() 是格式化输出到一个stream，通常是到文件。

### 17.说说go语言中的for循环

for 循环**支持 continue 和 break 来控制循环**，但是它提供了一个更高级的break，可以选择中断哪一个循环。[配合标签]

for 循环**不支持以逗号为间隔的多个赋值语句，必须使用平行赋值的方式来初始化多个变量**。

### 18.Array 类型的值作为函数参数

在 C/C++ 中，数组（名）是指针。将数组作为参数传进函数时，相当于传递了数组内存地址的引用，在函数内部会改变该数组的值。

在 Go 中，数组是值。作为参数传进函数时，传递的是数组的原始值拷贝，此时在函数内部是无法更新该数组的。

```go
// 数组使用值拷贝传参
func main() {
    x := [3]int{1,2,3}

    func(arr [3]int) {
        arr[0] = 7
        fmt.Println(arr) // [7 2 3]
    }(x)
    fmt.Println(x)   // [1 2 3] // 并不是你以为的 [7 2 3]
}
```

想改变数组，直接传递指向这个数组的指针类型。

```go
// 传址会修改原数据
func main() {
  x := [3]int{1,2,3}

  func(arr *[3]int) { // 匿名函数
      (*arr)[0] = 7 
    	fmt.Println(arr) // &[7 2 3]
  }(&x)
  fmt.Println(x) // [7 2 3]
}
```

直接使用 slice：即使函数内部得到的是 slice 的值拷贝，但依旧会更新 slice 的原始数据（底层 array）

```go
// 错误示例
func main() {
    x := []string{"a", "b", "c"}
    for v := range x { // range返回类型，索引遗漏
      	fmt.Println(v) // 1 2 3
    }
}


// 正确示例
func main() {
    x := []string{"a", "b", "c"}
    for _, v := range x { // 使用 _ 丢弃索引
      	fmt.Println(v)
    }
}
```

### 19.说说go语言中的switch语句

单个 case 中，可以出现多个变量或结果选项，但最终结果要为相同类型的表达式。多个可能复合条件的值，要使用逗号分隔。case默认带有break,自动退出，只有在 case 中明确添加 fallthrough关键字，才会继续执行紧跟的下一个 case。

### 20.说说go语言中有没有隐藏的this指针

Go语言中没有隐藏的this指针。

方法施加的对象**显式传递**，没有被隐藏起来。（指的是接收器。需要给结构体增加方法时，需要使用  func (a 结构体名) 方法名(参数列表) (返回值列表) {函数体}  这种形式，在函数体里面，调用结构体成员的时候使用的就是 a.xxx，用 c 语言的方式来解释，就是将对象作为参数传入了函数，函数调用这个参数从而访问对象的成员，当然这个函数是友联函数，可以访问任意访问权限的成员）

golang 的面向对象表达更直观，对于面向过程只是换了一种语法形式来表达。

方法施加的对象不需要非得是指针，也不用非得叫 this。（可以穿对象，不一定要传对象指针，至于名字随意！）

### 21.go语言中的引用类型包含哪些

数组切片(slice)、字典(map)、通道（channel）、接口（interface）。

### 22.go语言中指针运算有哪些

可以通过“&”取指针的地址；可以通过“*”取指针指向的数据。

### 23.说说go语言的main函数

main 函数**不能带参数**；main 函数**不能定义返回值**。main 函数**所在的包必须为 main 包**；main 函数中**可以使用 flag 包来获取和解析命令行参数**。

### 24.go语言触发异常的场景有哪些

- 空指针解析
- 下标越界
- 除数为0
- 调用 panic 函数

### 25.go语言编程的好处是什么

- 编译和运行都很快。
- 在语言层级支持并行操作。
- 有垃圾处理器GC。
- 内置字符串和 maps。
- 函数是 go 语言的最基本编程单位。

### 26.说说go语言的select机制

- select 机制用来处理异步 IO 问题

- select 机制最大的一条限制就是每个 case 语句里必须是一个 IO 操作

- golang 在语言级别支持 select 关键字

- 1.select+case是用于阻塞监听goroutine的，如果没有case，就单单一个select{}，则为监听当前程序中的goroutine，此时注意，需要有真实的goroutine在跑，否则select{}会报panic

  2.select底下有多个可执行的case，则随机执行一个。

  3.select常配合for循环来监听channel有没有故事发生。需要注意的是在这个场景下，break只是退出当前select而不会退出for，需要用break TIP / goto的方式。

  4.无缓冲的通道，则传值后立马close，则会在close之前阻塞，有缓冲的通道则即使close了也会继续让接收后面的值【！】

  5.同个通道多个goroutine进行关闭，可用recover panic的方式来判断通道关闭问题【！】

### 27.解释一下go语言中的静态类型声明

静态类型声明是**告诉编译器不需要太多的关注这个变量的细节**。

静态变量的声明，**只是针对于编译的时候, 在连接程序的时候，编译器还要对这个变量进行实际的声明。**

### 28.go的接口是什么

- 在 go 语言中，**interface 也就是接口**，被**用来指定一个对象**。接口具有下面的要素:
- 一系列的方法
- 具体应用中用来表示某个数据类型
- 在 go 中使用 interface 来**实现多态**

### 29.Go语言里面的类型断言是怎么回事

类型断言是**用来从一个接口里面读取数值给一个具体的类型变量**。

类型断言的语法格式如下：

value, ok := x.(T)

其中，x 表示一个接口的类型，T 表示一个具体的类型（也可为接口类型）。

该断言表达式会返回 x 的值（也就是 value）和一个布尔值（也就是 ok），可根据该布尔值判断 x 是否为 T 类型：

- 如果 T 是具体某个类型，类型断言会检查 x 的动态类型是否等于具体类型 T。如果检查成功，类型断言返回的结果是 x 的动态值，其类型是 T。
- 如果 T 是接口类型，类型断言会检查 x 的动态类型是否满足 T。如果检查成功，x 的动态值不会被提取，返回值是一个类型为 T 的接口值。
- 无论 T 是什么类型，如果 x 是 nil 接口值，类型断言都会失败。

示例代码如下：

```go
package main
import ("fmt")
func main() { 
  var x interface{}  
  x = 10   
  value, ok := x.(int)  
  fmt.Print(value, ",", ok)
}
```

运行结果如下：

10,true

需要注意如果不接收第二个参数也就是上面代码中的 ok，断言失败时会直接造成一个 panic。如果 x 为 nil 同样也会 panic。

**类型转换**是指转换两个不相同的数据类型。

### 30.go语言中局部变量和全局变量的缺省值是什么

全局变量的缺省值是与这个类型相关的零值。

### 31.模块化编程是怎么回事

模块化编程是指**把一个大的程序分解成几个小的程序。这么做的目的是为了减少程序的复杂度，易于维护，并且达到最高的效率。**

### 32.Golang的方法有什么特别之处

**函数的定义声明没有接收者。**
方法的声明和函数类似，他们的区别是：**方法在定义的时候，会在func和方法名之间增加一个参数，这个参数就是接收者，这样我们定义的这个方法就和接收者绑定在了一起，称之为这个接收者的方法。**
Go语言里有两种类型的接收者：值接收者和指针接收者。使用值类型接收者定义的方法，在调用的时候，使用的其实是值接收者的一个副本，所以对该值的任何操作，不会影响原来的类型变量。——-相当于形式参数。

如果我们使用一个指针作为接收者，那么就会其作用了，因为指针接收者传递的是一个指向原值指针的副本，指针的副本，指向的还是原来类型的值，所以修改时，同时也会影响原来类型变量的值。

### 33.Golang可变参数

**函数方法的参数，可以是任意多个，这种我们称之为可以变参数**。比如我们常用的fmt.Println()这类函数，可以接收一个可变的参数。可以变参数，可以是任意多个。我们自己也可以定义可以变参数，**可变参数的定义，在类型前加上省略号…**即可。

```go
func main() {
   print("1","2","3")
}


func print (a ...interface{}){
    for _,v := range a{
      	fmt.Print(v)
    }
    fmt.Println()
}
```

例子中我们自己定义了一个接受可变参数的函数，效果和fmt.Println()一样。可变参数本质上是一个数组，所以我们向使用数组一样使用它，比如例子中的 for range 循环。

### 34.Golang Slice的底层实现

**切片是基于数组实现的，它的底层是数组**，它自己本身非常小，可以理解为对底层数组的抽象。**因为基于数组实现，所以它的底层的内存是连续分配的，效率非常高，还可以通过索引获得数据，可以迭代以及垃圾回收优化。**
切片本身并不是动态数组或者数组指针。它内部实现的数据结构通过指针引用底层数组，设定相关属性将数据读写操作限定在指定的区域内。切片本身是一个只读对象，其工作机制类似数组指针的一种封装。
切片对象非常小，是因为它是只有3个字段的数据结构：

- 指向底层数组的指针
- 切片的长度
- 切片的容量

这3个字段，就是Go语言操作底层数组的元数据。

<img src="https://topgoer.cn/uploads/blog/202104/attach_16778ba69a10ebd8.png" alt="null" style="zoom: 50%;" />

### 35.Golang Slice的扩容机制，有什么注意点

Go 中切片扩容的策略是这样的：

**首先判断，如果新申请容量大于 2 倍的旧容量，最终容量就是新申请的容量。否则判断，如果旧切片的长度小于 1024，则最终容量就是旧容量的两倍。**

**否则判断，如果旧切片长度大于等于 1024，则最终容量从旧容量开始循环增加原来的 1/4 , 直到最终容量大于等于新申请的容量。如果最终容量计算值溢出，则最终容量就是新申请容量。**

情况一：原数组还有容量可以扩容（实际容量没有填充完），这种情况下，扩容以后的数组还是指向原来的数组，对一个切片的操作可能影响多个指针指向相同地址的Slice。

情况二：原来数组的容量已经达到了最大值，再想扩容， Go 默认会先开一片内存区域，把原来的值拷贝过来，然后再执行 append() 操作。这种情况丝毫不影响原数组。

要复制一个Slice，最好使用Copy函数。

### 36.Golang Map底层实现

Golang 中 map 的底层实现是一个散列表，因此实现 map 的过程实际上就是实现散表的过程。
在这个散列表中，主要出现的结构体有两个，**一个叫hmap(a header for a go map)，一个叫bmap(a bucket for a Go map，通常叫其bucket)**。

hmap如下所示：

<img src="https://topgoer.cn/uploads/blog/202104/attach_16778bb1faac1ea3.png" alt="null" style="zoom: 50%;" />

图中有很多字段，但是便于理解 map 的架构，你只需要关心的只有一个，就是标红的字段：buckets 数组。Golang 的 map 中用于存储的结构是 bucket数组。而 bucket(即bmap)的结构是怎样的呢？
bucket：

<img src="https://topgoer.cn/uploads/blog/202104/attach_16778bb53eb6d643.png" alt="null" style="zoom: 50%;" />

相比于 hmap，bucket 的结构显得简单一些，标橙的字段依然是“核心”，我们使用的 map 中的 key 和 value 就存储在这里。

“高位哈希值”数组记录的是当前 bucket 中 key 相关的”索引”，稍后会详细叙述。还有一个字段是一个指向扩容后的 bucket 的指针，使得 bucket 会形成一个链表结构。
整体的结构应该是这样的：

<img src="https://topgoer.cn/uploads/blog/202104/attach_16778bbcf3d8904e.png" alt="null" style="zoom: 50%;" />

Golang 把求得的哈希值按照用途一分为二：高位和低位。低位用于寻找当前 key属于 hmap 中的哪个 bucket，而高位用于寻找 bucket 中的哪个 key。
需要特别指出的一点是：map中的key/value值都是存到同一个数组中的。这样做的好处是：在key和value的长度不同的时候，可以消除padding带来的空间浪费。

![null](https://topgoer.cn/uploads/blog/202104/attach_16778bc2077dd9d7.png)

Map 的扩容：当 Go 的 map 长度增长到大于加载因子所需的 map 长度时，Go 语言就会将产生一个新的 bucket 数组，然后把旧的 bucket 数组移到一个属性字段 oldbucket中。

注意：并不是立刻把旧的数组中的元素转义到新的 bucket 当中，而是，只有当访问到具体的某个 bucket 的时候，会把 bucket 中的数据转移到新的 bucket 中。

### 37.Golang的内存模型，为什么小对象多了会造成gc压力

通常**小对象过多会导致 GC 三色法消耗过多的GPU**。优化思路是，减少对象分配。

### 38.Data Race问题怎么解决？能不能不加锁解决这个问题

data race 译作数据竞争，比如不同的goroutine并发读写同一个变量，可能会发生数据竞争。

**同步访问共享数据**是处理**数据竞争**的一种有效的方法。

golang在 1.1 之后引入了竞争检测机制，可以使用 go run -race 或者 go build -race来进行静态检测。其在内部的实现是,开启多个协程执行同一个命令， 并且记录下每个变量的状态。

竞争检测器基于C/C++的ThreadSanitizer 运行时库，该库在Google内部代码基地和Chromium找到许多错误。这个技术在2012年九月集成到Go中，从那时开始，它已经在标准库中检测到42个竞争条件。现在，它已经是我们持续构建过程的一部分，当竞争条件出现时，它会继续捕捉到这些错误。

竞争检测器已经完全集成到Go工具链中，仅仅添加-race标志到命令行就使用了检测器。

```
$ go test -race mypkg    // 测试包
$ go run -race mysrc.go  // 编译和运行程序 $ go build -race mycmd 
// 构建程序 $ go install -race mypkg // 安装程序
```

要想解决数据竞争的问题**可以使用互斥锁sync.Mutex,解决数据竞争(Data race)**,也**可以使用管道解决,使用管道的效率要比互斥锁高**。

### 39.在 range 迭代 slice 时，你怎么修改值的

在 range 迭代中，得到的值其实是元素的一份值拷贝，更新拷贝并不会更改原来的元素，即是拷贝的地址并不是原有元素的地址。

```go
func main() {
    data := []int{1, 2, 3}
    for _, v := range data {
      	v *= 10  // data 中原有元素是不会被修改的
    }
    fmt.Println("data: ", data) // data:  [1 2 3]
}
```

**如果要修改原有元素的值，应该**使用索引直接访问。

```go
func main() {
    data := []int{1, 2, 3}
    for i, v := range data {
      	data[i] = v * 10 
    }
    fmt.Println("data: ", data) // data:  [10 20 30]
}
```

**如果你的集合保存的是指向值的指针，需稍作修改。依旧需要使用索引访问元素，不过可以使用 range 出来的元素直接更新原有值。**

```go
func main() {
    data := []*struct{ num int }{{1}, {2}, {3},}
    for _, v := range data {
      	v.num *= 10 // 直接使用指针更新
    }
    fmt.Println(data[0], data[1], data[2]) // &{10} &{20} &{30}
}
```

### 40.nil  和 nil interface 的区别

虽然 interface 看起来像指针类型，但它不是。interface 类型的变量只有在类型和值均为 nil 时才为 nil.如果你的 interface 变量的值是跟随其他变量变化的，与 nil 比较相等时小心。如果你的函数返回值类型是 interface，更要小心这个坑：

```go
func main() {
   var data *byte
   var in interface{}

   fmt.Println(data, data == nil) // <nil> true
   fmt.Println(in, in == nil) // <nil> true

   in = data
   fmt.Println(in, in == nil) // <nil> false // data 值为 nil，但 in 值不为 nil
}

// 正确示例
func main() {
    doIt := func(arg int) interface{} {
        var result *struct{} = nil

        if arg > 0 {
          	result = &struct{}{}
        } else {
          	return nil // 明确指明返回 nil
        }

        return result
    }


    if res := doIt(-1); res != nil {
      	fmt.Println("Good result: ", res)
    } else {
      	fmt.Println("Bad result: ", res) // Bad result: <nil>
    }
}
```

### 41.select可以用于什么【可看26】

常用语gorotine的完美退出。

golang 的 select 就是监听 IO 操作，当 IO 操作发生时，触发相应的动作每个case语句里必须是一个IO操作，确切的说，应该是一个面向channel的IO操作。

### 42. 指针数据坑

range到底有什么坑呢，我们先来运行一个例子吧。

```go
package main

import (
  "fmt"
)

type user struct {
  name string
  age uint64
}

func main()  {
  u := []user{
    {"asong",23},
    {"song",19},
    {"asong2020",18},
  }
  n := make([]*user,0,len(u))
  for _,v := range u{
    n = append(n, &v) 
  }
  fmt.Println(n)
  for _,v := range n{
    fmt.Println(v)
  }
}
```

这个例子的目的是，通过u这个slice构造成新的slice。我们预期应该是显示uslice的内容，但是运行结果如下：

```
[0xc0000a6040 0xc0000a6040 0xc0000a6040]
&{asong2020 18}
&{asong2020 18}
&{asong2020 18}
```

这里我们看到n这个slice打印出来的三个同样的数据，并且他们的内存地址相同。这是什么原因呢？先别着急，再来看这一段代码，我给他改正确他，对比之后我们再来分析，你们才会恍然大悟。

```go
package main

import (
  "fmt"
)

type user struct {
  name string
  age uint64
}

func main()  {
  u := []user{
    {"asong",23},
    {"song",19},
    {"asong2020",18},
  }
  n := make([]*user,0,len(u))
  for _,v := range u{
    o := v // 多了这一步！
    n = append(n, &o)
  }
  fmt.Println(n)
  for _,v := range n{
    fmt.Println(v)
  }
}
```

细心的你们看到，我改动了哪一部分代码了嘛？对，没错，我就加了一句话，他就成功了，我在for range里面引入了一个中间变量，每次迭代都重新声明一个变量o，赋值后再将v的地址添加n切片中，这样成功解决了刚才的问题。

现在来解释一下原因：在**for range中，变量v是用来保存迭代切片所得的值，因为v只被声明了一次，每次迭代的值都是赋值给v，该变量的内存地址始终未变，这样讲他的地址追加到新的切片中，该切片保存的都是同一个地址**，这肯定无法达到预期效果的。这里还需要注意一点，变量v的地址也并不是指向原来切片u[2]的，因我在使用range迭代的时候，变量v的数据是切片的拷贝数据，所以直接copy了结构体数据。

上面的问题**还有一种解决方法，直接引用数据的内存**，这个方法比较好，不需要开辟新的内存空间，看代码：

```go
......略
for k,_ := range u{
  n = append(n, &u[k])
}
......略
```

### 43. 是否会造成死循环

来看一段代码：

```go
func main() {
  v := []int{1, 2, 3}
  for i := range v {  // i 为索引。range会对最初的v拷贝，所以后面v变化和range的无关！
    v = append(v, i)
  }
}
```

这一段代码会造成死循环吗？答案：当然不会，**前面都说了range会对切片做拷贝，新增的数据并不在拷贝内容中，并不会发生死循环。**这种题一般会在面试中问，可以留意下的。

### 你不知道的range用法

#### delete

没看错，删除，在range迭代时，可以删除map中的数据，第一次见到这么使用的，我刚听到确实不太相信，所以我就去查了一下官方文档，确实有这个写法：

```go
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}
```

看看官方的解释：

```
The iteration order over maps is not specified and is not guaranteed to be the same from one iteration to the next. If map entries that have not yet been reached are removed during iteration, the corresponding iteration values will not be produced. If map entries are created during iteration, that entry may be produced during the iteration or may be skipped. The choice may vary for each entry created and from one iteration to the next. If the map is nil, the number of iterations is 0.

翻译：
未指定`map`的迭代顺序，并且不能保证每次迭代之间都相同。 如果在迭代过程中删除了尚未到达的映射条目，则不会生成相应的迭代值。 如果映射条目是在迭代过程中创建的，则该条目可能在迭代过程中产生或可以被跳过。 对于创建的每个条目以及从一个迭代到下一个迭代，选择可能有所不同。 如果映射为nil，则迭代次数为0。
```

看这个代码：

```go
func main()  {
  d := map[string]string{
    "asong": "帅",
    "song": "太帅了",
  }
  for k := range d{
    if k == "asong"{
      delete(d,k)
    }
  }
  fmt.Println(d)
}

# 运行结果
map[song:太帅了]
```

从运行结果我们可以看出，key为asong的这位帅哥被从帅哥map中删掉了，哇哦，可气呀。这个方法，相信很多小伙伴都不知道，今天教给你们了，以后可以用起来了。

#### add

上面是删除，那肯定会有新增呀，直接看代码吧。

```go
func main()  {
  d := map[string]string{
    "asong": "帅",
    "song": "太帅了",
  }
  for k,v := range d{
    d[v] = k
    fmt.Println(d)
  }
}
```

这里我把打印放到了range里，你们思考一下，新增的元素，在遍历时能够遍历到呢。我们来验证一下。

```go
func main()  {
  var addTomap = func() {
    var t = map[string]string{
      "asong": "太帅",
      "song": "好帅",
      "asong1": "非常帅",
    }
    for k := range t {
      t["song2020"] = "真帅"
      fmt.Printf("%s%s ", k, t[k])
    }
  }
  for i := 0; i < 10; i++ {
    addTomap()
    fmt.Println()
  }
}
```

运行结果：

```
asong太帅 song好帅 asong1非常帅 song2020真帅 
asong太帅 song好帅 asong1非常帅 
asong太帅 song好帅 asong1非常帅 song2020真帅 
asong1非常帅 song2020真帅 asong太帅 song好帅 
asong太帅 song好帅 asong1非常帅 song2020真帅 
asong太帅 song好帅 asong1非常帅 song2020真帅 
asong太帅 song好帅 asong1非常帅 
asong1非常帅 song2020真帅 asong太帅 song好帅 
asong太帅 song好帅 asong1非常帅 song2020真帅 
asong太帅 song好帅 asong1非常帅 song2020真帅
```

从运行结果，我们可以看出来，**每一次的结果并不是确定的**。这是为什么呢？这就来揭秘，**map内部实现是一个链式hash表，为了保证无顺序，初始化时会随机一个遍历开始的位置，所以新增的元素被遍历到就变的不确定了**，同样删除也是一个道理，但是删除元素后边就不会出现，所以一定不会被遍历到。

### 44. 拷贝大切片一定比拷贝小切片代价大吗？

这道题比较有意思，原文地址：Are large slices more expensive than smaller ones?

这道题本质是考察对切片本质的理解，Go语言中只有值传递，所以我们以传递切片为例子：

```go
func main()  {
  param1 := make([]int, 100)
  param2 := make([]int, 100000000)
  smallSlice(param1)
  largeSlice(param2)
}

func smallSlice(params []int)  {
  // ....
}

func largeSlice(params []int)  {
  // ....
}
```

切片param2要比param1大1000000个数量级，在进行值拷贝的时候，是否需要更昂贵的操作呢？

**实际上不会**，因为切片本质内部结构如下：

```go
type SliceHeader struct {
  Data uintptr
  Len  int
  Cap  int
}
```

切片中的第一个字是指向切片底层数组的指针，这是切片的存储空间，第二个字段是切片的长度，第三个字段是容量。**将一个切片变量分配给另一个变量只会复制三个机器字，大切片跟小切片的区别无非就是 Len 和 Cap的值比小切片的这两个值大一些，如果发生拷贝，本质上就是拷贝上面的三个字段。**

### 44. 切片的深浅拷贝

深浅拷贝都是进行复制，区别在于复制出来的新对象与原来的对象在它们发生改变时，是否会相互影响，**本质区别就是复制出来的对象与原对象是否会指向同一个地址。**在Go语言，切片拷贝有三种方式：

- 使用=操作符拷贝切片，这种就是浅拷贝
- 使用[:]下标的方式复制切片，这种也是浅拷贝
- 使用**Go语言的内置函数copy()进行切片拷贝，这种就是深拷贝**

### 45. 零切片、空切片、nil切片是什么

为什么问题中这么多种切片呢？因为在Go语言中切片的创建方式有五种，不同方式创建出来的切片也不一样；

- 零切片

我们**把切片内部数组的元素都是零值或者底层数组的内容就全是 nil的切片叫做零切片，使用make创建的、长度、容量都不为0的切片就是零值切片**：

```
slice := make([]int,5) // 0 0 0 0 0
slice := make([]*int,5) // nil nil nil nil nil
```

- nil切片

**nil切片的长度和容量都为0，并且和nil比较的结果为true**，**采用直接创建切片的方式、new创建切片的方式都可以创建nil切片**：

```
var slice []int
var slice = *new([]int)
```

- 空切片

**空切片的长度和容量也都为0，但是和nil的比较结果为false**，因为所有的空切片的数据指针都指向同一个地址 0xc42003bda0；**使用字面量、make可以创建空切片**：

```
var slice = []int{}
var slice = make([]int, 0)
```

空切片指向的 zerobase 内存地址是一个神奇的地址，从 Go 语言的源代码中可以看到它的定义：

```
// base address for all 0-byte allocations
var zerobase uintptr

// 分配对象内存
func mallocgc(size uintptr, typ *_type, needzero bool) unsafe.Pointer {
 ...
 if size == 0 {
  return unsafe.Pointer(&zerobase)
 }
  ...
}
```

### 46. 切片的扩容策略

这个问题是一个高频考点，我们通过源码来解析一下切片的扩容策略，切片的扩容都是调用growslice方法，截取部分重要源代码：

```go
// runtime/slice.go
// et：表示slice的一个元素；old：表示旧的slice；cap：表示新切片需要的容量；
func growslice(et *_type, old slice, cap int) slice {
  if cap < old.cap {
    panic(errorString("growslice: cap out of range"))
  }

  if et.size == 0 {
    // append should not create a slice with nil pointer but non-zero len.
    // We assume that append doesn't need to preserve old.array in this case.
    return slice{unsafe.Pointer(&zerobase), old.len, cap}
  }

  newcap := old.cap
  // 两倍扩容
  doublecap := newcap + newcap
  // 新切片需要的容量大于两倍扩容的容量，则直接按照新切片需要的容量扩容
  if cap > doublecap {
    newcap = cap
  } else {
    // 原 slice 容量小于 1024 的时候，新 slice 容量按2倍扩容
    if old.cap < 1024 {
      newcap = doublecap
    } else { // 原 slice 容量超过 1024，新 slice 容量变成原来的1.25倍。
      // Check 0 < newcap to detect overflow
      // and prevent an infinite loop.
      for 0 < newcap && newcap < cap {
        newcap += newcap / 4
      }
      // Set newcap to the requested cap when
      // the newcap calculation overflowed.
      if newcap <= 0 {
        newcap = cap
      }
    }
  }

  // 后半部分还对 newcap 作了一个内存对齐，这个和内存分配策略相关。进行内存对齐之后，新 slice 的容量是要 大于等于 老 slice 容量的 2倍或者1.25倍。
  var overflow bool
  var lenmem, newlenmem, capmem uintptr
  // Specialize for common values of et.size.
  // For 1 we don't need any division/multiplication.
  // For sys.PtrSize, compiler will optimize division/multiplication into a shift by a constant.
  // For powers of 2, use a variable shift.
  switch {
    case et.size == 1:
    lenmem = uintptr(old.len)
    newlenmem = uintptr(cap)
    capmem = roundupsize(uintptr(newcap))
    overflow = uintptr(newcap) > maxAlloc
    newcap = int(capmem)
    case et.size == sys.PtrSize:
    lenmem = uintptr(old.len) * sys.PtrSize
    newlenmem = uintptr(cap) * sys.PtrSize
    capmem = roundupsize(uintptr(newcap) * sys.PtrSize)
    overflow = uintptr(newcap) > maxAlloc/sys.PtrSize
    newcap = int(capmem / sys.PtrSize)
    case isPowerOfTwo(et.size):
    var shift uintptr
    if sys.PtrSize == 8 {
      // Mask shift for better code generation.
      shift = uintptr(sys.Ctz64(uint64(et.size))) & 63
    } else {
      shift = uintptr(sys.Ctz32(uint32(et.size))) & 31
    }
    lenmem = uintptr(old.len) << shift
    newlenmem = uintptr(cap) << shift
    capmem = roundupsize(uintptr(newcap) << shift)
    overflow = uintptr(newcap) > (maxAlloc >> shift)
    newcap = int(capmem >> shift)
    default:
    lenmem = uintptr(old.len) * et.size
    newlenmem = uintptr(cap) * et.size
    capmem, overflow = math.MulUintptr(et.size, uintptr(newcap))
    capmem = roundupsize(capmem)
    newcap = int(capmem / et.size)
  }
}
```

通过源代码可以总结切片扩容策略：

> 切片**在扩容时会进行内存对齐**，这个和内存分配策略相关。**进行内存对齐之后，新 slice 的容量是要 大于等于老 slice 容量的 2倍或者1.25倍，当原 slice 容量小于 1024 的时候，新 slice 容量变成原来的 2 倍；原 slice 容量超过 1024，新 slice 容量变成原来的1.25倍。**

### 48. 参数传递切片和切片指针有什么区别？

我们都知道切片底层就是一个结构体，里面有三个元素：

```go
type SliceHeader struct {
  Data uintptr
  Len  int
  Cap  int
}
```

分别表示切片底层数据的地址，切片长度，切片容量。

**当切片作为参数传递时，其实就是一个结构体的传递，因为Go语言参数传递只有值传递，传递一个切片就会浅拷贝原切片，但因为底层数据的地址没有变**，所以在函数内对切片的修改，也将会影响到函数外的切片，举例：

```go
func modifySlice(s []string)  {
  s[0] = "song"
  s[1] = "Golang"
  fmt.Println("out slice: ", s)
}

func main()  {
  s := []string{"asong", "Golang梦工厂"}
  modifySlice(s)
  fmt.Println("inner slice: ", s)
}
// 运行结果
out slice:  [song Golang]
inner slice:  [song Golang]
```

不过这也有一个特例，先看一个例子：

```go
func appendSlice(s []string)  {
  s = append(s, "快关注！！")
  fmt.Println("out slice: ", s)
}

func main()  {
  s := []string{"asong", "Golang梦工厂"}
  appendSlice(s)
  fmt.Println("inner slice: ", s)
}
// 运行结果
out slice:  [asong Golang梦工厂 快关注！！]
inner slice:  [asong Golang梦工厂]
```

因为切片发生了扩容，函数外的切片指向了一个新的底层数组，所以函数内外不会相互影响，因此可以得出一个结论，**当参数直接传递切片时，如果指向底层数组的指针被覆盖或者修改（copy、重分配、append触发扩容），此时函数内部对数据的修改将不再影响到外部的切片，代表长度的len和容量cap也均不会被修改。**

参数传递切片指针就很容易理解了，**如果你想修改切片中元素的值，并且更改切片的容量和底层数组，则应该按指针传递。**

### 49. range遍历切片有什么要注意的？

Go语言提供了range关键字用于for 循环中迭代数组(array)、切片(slice)、通道(channel)或集合(map)的元素，有两种使用方式：

```
for k,v := range _ { }
for k := range _ { }
```

**第一种是遍历下标和对应值，第二种是只遍历下标，使用range遍历切片时会先拷贝一份，然后在遍历拷贝数据**：

```go
s := []int{1, 2}
for k, v := range s {

}
会被编译器认为是
for_temp := s
len_temp := len(for_temp)
for index_temp := 0; index_temp < len_temp; index_temp++ {
  value_temp := for_temp[index_temp]
  _ = index_temp
  value := value_temp

}
```

不知道这个知识点的情况下很容易踩坑，例如下面这个例子：

```go
package main

import (
  "fmt"
)

type user struct {
  name string
  age uint64
}

func main()  {
  u := []user{
    {"asong",23},
    {"song",19},
    {"asong2020",18},
  }
  for _,v := range u{
    if v.age != 18{
      v.age = 20
    }
  }
  fmt.Println(u)
}
// 运行结果
[{asong 23} {song 19} {asong2020 18}]
```

因为使用range遍历切片u，变量v是拷贝切片中的数据，修改拷贝数据不会对原切片有影响。

##### 参考：[Golang 50题 笔记 - 格罗玛什·地狱咆哮 - 博客园 (cnblogs.com)](https://www.cnblogs.com/arvin-an/p/14666978.html)

