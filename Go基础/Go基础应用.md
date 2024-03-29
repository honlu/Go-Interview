# 基础应用

### 1.你是如何关闭 HTTP 的响应体的

直接在处理 HTTP 响应错误的代码块中，直接关闭非 nil 的响应体；

手动调用 defer 来关闭响应体。

```go
// 正确示例
func main() {
    resp, err := http.Get("http://www.baidu.com") // 发出请求并返回请求结果

    // 关闭 resp.Body 的正确姿势
    if resp != nil {
      	defer resp.Body.Close()
    }

    checkError(err) // 检查错误，省略写法
    defer resp.Body.Close()	// 手动调用defer来关闭响应体

    body, err := ioutil.ReadAll(resp.Body) // 一次性读写文件的全部数据
    checkError(err)

    fmt.Println(string(body))
}
```

### 2.你是否主动关闭过http连接，为啥要这样做

有关闭，不关闭会程序可能**会消耗完 socket 描述符**。有如下2种关闭方式：

- 直接设置请求变量的 Close 字段值为 true，每次请求结束后就会主动关闭连接。
- 设置 Header 请求头部选项 Connection: close，然后服务器返回的响应头部也会有这个选项，此时 HTTP 标准库会主动断开连接

```go
// 主动关闭连接
func main() {
    req, err := http.NewRequest("GET", "http://golang.org", nil)
    checkError(err)

    req.Close = true // 直接设置请求变量的Close字段值为true,每次请求结束后主动关闭连接
    //req.Header.Add("Connection", "close") // 等效的关闭方式

    resp, err := http.DefaultClient.Do(req)
    if resp != nil {
      	defer resp.Body.Close()
    }
    checkError(err)

    body, err := ioutil.ReadAll(resp.Body)
    checkError(err)

    fmt.Println(string(body))
}
```

你可以创建一个自定义配置的 HTTP transport(传输) 客户端，用来取消 HTTP 全局的**复用连接**。

```go
func main() {
    tr := http.Transport{DisableKeepAlives: true} // 自定义配置传输客户端，用来取消HTTP全部的复用连接。
    client := http.Client{Transport: &tr}

    resp, err := client.Get("https://golang.google.cn/")
    if resp != nil {
      	defer resp.Body.Close()
    }
    checkError(err)

    fmt.Println(resp.StatusCode) // 200

    body, err := ioutil.ReadAll(resp.Body)
    checkError(err)

    fmt.Println(len(string(body)))
}
```

### 3.解析 JSON 数据时，默认将数值当做哪种类型

在 encode/decode JSON 数据时，Go **默认会将数值当做 float64 处理**。

```go
func main() {
    var data = []byte(`{"status": 200}`)
    var result map[string]interface{}

    if err := json.Unmarshal(data, &result); err != nil {
      	log.Fatalln(err)
    }
}
```

解析出来的 200 是 float 类型。

### 4.说说go语言的beego框架

- beego 是一个 golang 实现的**轻量级HTTP框架**
- beego 可以**通过注释路由、正则路由等多种方式完成 url 路由注入**
- 可以使用 bee new 工具生成空工程，然后使用 bee run 命令自动热编译

### 5.说说go语言的goconvey框架

- goconvey 是一个**支持 golang 的单元测试框架**
- goconvey **能够自动监控文件修改并启动测试**，并可以将测试结果实时输出到web界面
- goconvey 提供了丰富的断言简化测试用例的编写

### 6.GoStub的作用是什么

GoStub也是一种测试框架：

- GoStub 可以对全局变量打桩
- GoStub 可以对函数打桩
- GoStub 不可以对类的成员方法打桩
- GoStub 可以打动态桩，比如对一个函数打桩后，多次调用该函数会有不同的行为

### 7. JSON 标准库对 nil slice 和 空 slice 的处理是一致的吗

首先 JSON 标准库对 nil slice 和 空 slice 的处理是不一致。

通常错误的用法，**会报数组越界的错误，因为只是声明了slice，却没有给实例化的对象。**

```go
var slice []int // nil slice
slice[1] = 0
```

此时slice的值是nil，这种情况可以用于需要返回slice的函数，当函数出现异常的时候，保证函数依然会有nil的返回值。

empty slice 是指slice不为nil，但是slice没有值，slice的底层的空间是空的，此时的定义如下：

```go
slice := make([]int,0）// 空slice，没有值，空间也是空的
slice := []int{}
```

当我们查询或者处理一个空的列表的时候，这非常有用，它会告诉我们返回的是一个列表，但是列表内没有任何值。总之，nil slice 和 empty slice是不同的东西,需要我们加以区分的。

