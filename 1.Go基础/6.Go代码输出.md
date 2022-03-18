## 常量与变量

1. 下列代码的输出是：

```
func main() {
	const (
		a, b = "golang", 100
		d, e
		f bool = true
		g
	)
	fmt.Println(d, e, g)
}
```

<details open="" style="box-sizing: border-box; margin-top: 10px; margin-bottom: 10px; padding: 5px 10px; border-width: 1px; border-style: solid; border-color: rgb(227, 227, 227) rgb(236, 236, 236) rgb(224, 224, 224) rgb(227, 227, 227); border-image: initial; background-color: rgb(240, 248, 255); box-shadow: rgba(0, 0, 0, 0.07) 1px 2px 1px;"><summary style="box-sizing: border-box; cursor: pointer; font-weight: bold; user-select: none;">答案</summary><div style="box-sizing: border-box;"><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;">golang 100 true</p><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;">在同一个 const group 中，如果常量定义与前一行的定义一致，则可以省略类型和值。编译时，会按照前一行的定义自动补全。即等价于</p><figure class="highlight go" style="box-sizing: border-box; margin: 10px 0px 20px; padding: 15px; overflow: auto; font-size: 13px; color: rgb(36, 41, 46); background: rgb(246, 248, 250); line-height: 1.8;"><table style="box-sizing: border-box; border-collapse: collapse; border-spacing: 0px; margin: 0px; display: block; width: auto; overflow: auto; border: none;"><tbody style="box-sizing: border-box;"><tr style="box-sizing: border-box; background-color: transparent; border-top: none;"><td class="code" style="box-sizing: border-box; padding: 0px; text-align: left; border: none !important;"><pre style="box-sizing: border-box; margin: 0px; padding: 1px; font-family: SFMono-Regular, Consolas, &quot;Liberation Mono&quot;, Menlo, Courier, monospace; overflow-wrap: normal; overflow: auto; line-height: 1.8; background: rgb(246, 248, 250); border-radius: 3px; font-size: 13px; color: rgb(36, 41, 46); border: none;"><span class="line" style="box-sizing: border-box; height: 20px;"><span class="function" style="box-sizing: border-box; color: rgb(66, 113, 174);"><span class="keyword" style="box-sizing: border-box; color: rgb(215, 58, 73);">func</span> <span class="title" style="box-sizing: border-box; color: rgb(111, 66, 193);">main</span><span class="params" style="box-sizing: border-box; color: rgb(227, 98, 9);">()</span></span> {</span><br style="box-sizing: border-box;"><span class="line" style="box-sizing: border-box; height: 20px;">	<span class="keyword" style="box-sizing: border-box; color: rgb(215, 58, 73);">const</span> (</span><br style="box-sizing: border-box;"><span class="line" style="box-sizing: border-box; height: 20px;">		a, b = <span class="string" style="box-sizing: border-box; color: rgb(113, 140, 0);">"golang"</span>, <span class="number" style="box-sizing: border-box; color: rgb(113, 140, 0);">100</span></span><br style="box-sizing: border-box;"><span class="line" style="box-sizing: border-box; height: 20px;">		d, e = <span class="string" style="box-sizing: border-box; color: rgb(113, 140, 0);">"golang"</span>, <span class="number" style="box-sizing: border-box; color: rgb(113, 140, 0);">100</span></span><br style="box-sizing: border-box;"><span class="line" style="box-sizing: border-box; height: 20px;">		f <span class="keyword" style="box-sizing: border-box; color: rgb(215, 58, 73);">bool</span> = <span class="literal" style="box-sizing: border-box; color: rgb(227, 98, 9);">true</span></span><br style="box-sizing: border-box;"><span class="line" style="box-sizing: border-box; height: 20px;">		g <span class="keyword" style="box-sizing: border-box; color: rgb(215, 58, 73);">bool</span> = <span class="literal" style="box-sizing: border-box; color: rgb(227, 98, 9);">true</span></span><br style="box-sizing: border-box;"><span class="line" style="box-sizing: border-box; height: 20px;">	)</span><br style="box-sizing: border-box;"><span class="line" style="box-sizing: border-box; height: 20px;">	fmt.Println(d, e, g)</span><br style="box-sizing: border-box;"><span class="line" style="box-sizing: border-box; height: 20px;">}</span><br style="box-sizing: border-box;"></pre></td></tr></tbody></table></figure></div></details>

1. 下列代码的输出是：

```
func main() {
	const N = 100
	var x int = N

	const M int32 = 100
	var y int = M
	fmt.Println(x, y)
}
```

<details open="" style="box-sizing: border-box; margin-top: 10px; margin-bottom: 10px; padding: 5px 10px; border-width: 1px; border-style: solid; border-color: rgb(227, 227, 227) rgb(236, 236, 236) rgb(224, 224, 224) rgb(227, 227, 227); border-image: initial; background-color: rgb(240, 248, 255); box-shadow: rgba(0, 0, 0, 0.07) 1px 2px 1px;"><summary style="box-sizing: border-box; cursor: pointer; font-weight: bold; user-select: none;">答案</summary><div style="box-sizing: border-box;"><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;">编译失败：cannot use M (type int32) as type int in assignment</p><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;">Go 语言中，常量分为无类型常量和有类型常量两种，<code style="box-sizing: border-box; font-family: SFMono-Regular, Consolas, &quot;Liberation Mono&quot;, Menlo, Courier, monospace; padding: 0.2em 0px; margin: 0px; background-color: rgba(27, 31, 35, 0.05); border-radius: 3px; color: rgb(36, 41, 46); font-size: 0.9em;">const N = 100</code>，属于无类型常量，赋值给其他变量时，如果字面量能够转换为对应类型的变量，则赋值成功，例如，<code style="box-sizing: border-box; font-family: SFMono-Regular, Consolas, &quot;Liberation Mono&quot;, Menlo, Courier, monospace; padding: 0.2em 0px; margin: 0px; background-color: rgba(27, 31, 35, 0.05); border-radius: 3px; color: rgb(36, 41, 46); font-size: 0.9em;">var x int = N</code>。但是对于有类型的常量<span>&nbsp;</span><code style="box-sizing: border-box; font-family: SFMono-Regular, Consolas, &quot;Liberation Mono&quot;, Menlo, Courier, monospace; padding: 0.2em 0px; margin: 0px; background-color: rgba(27, 31, 35, 0.05); border-radius: 3px; color: rgb(36, 41, 46); font-size: 0.9em;">const M int32 = 100</code>，赋值给其他变量时，需要类型匹配才能成功，所以显示地类型转换：</p><figure class="highlight go" style="box-sizing: border-box; margin: 10px 0px 20px; padding: 15px; overflow: auto; font-size: 13px; color: rgb(36, 41, 46); background: rgb(246, 248, 250); line-height: 1.8;"><table style="box-sizing: border-box; border-collapse: collapse; border-spacing: 0px; margin: 0px; display: block; width: auto; overflow: auto; border: none;"><tbody style="box-sizing: border-box;"><tr style="box-sizing: border-box; background-color: transparent; border-top: none;"><td class="code" style="box-sizing: border-box; padding: 0px; text-align: left; border: none !important;"><pre style="box-sizing: border-box; margin: 0px; padding: 1px; font-family: SFMono-Regular, Consolas, &quot;Liberation Mono&quot;, Menlo, Courier, monospace; overflow-wrap: normal; overflow: auto; line-height: 1.8; background: rgb(246, 248, 250); border-radius: 3px; font-size: 13px; color: rgb(36, 41, 46); border: none;"><span class="line" style="box-sizing: border-box; height: 20px;"><span class="keyword" style="box-sizing: border-box; color: rgb(215, 58, 73);">var</span> y <span class="keyword" style="box-sizing: border-box; color: rgb(215, 58, 73);">int</span> = <span class="keyword" style="box-sizing: border-box; color: rgb(215, 58, 73);">int</span>(M)</span><br style="box-sizing: border-box;"></pre></td></tr></tbody></table></figure></div></details>

1. 下列代码的输出是：

```
func main() {
	var a int8 = -1
	var b int8 = -128 / a
	fmt.Println(b)
}
```

<details open="" style="box-sizing: border-box; margin-top: 10px; margin-bottom: 10px; padding: 5px 10px; border-width: 1px; border-style: solid; border-color: rgb(227, 227, 227) rgb(236, 236, 236) rgb(224, 224, 224) rgb(227, 227, 227); border-image: initial; background-color: rgb(240, 248, 255); box-shadow: rgba(0, 0, 0, 0.07) 1px 2px 1px;"><summary style="box-sizing: border-box; cursor: pointer; font-weight: bold; user-select: none;">答案</summary><div style="box-sizing: border-box;"><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;">-128</p><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;">int8 能表示的数字的范围是 [-2^7, 2^7-1]，即 [-128, 127]。-128 是无类型常量，转换为 int8，再除以变量 -1，结果为 128，常量除以变量，结果是一个变量。变量转换时允许溢出，符号位变为1，转为补码后恰好等于 -128。</p><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;">对于有符号整型，最高位是是符号位，计算机用补码表示负数。补码 = 原码取反加一。</p><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;">例如：</p><figure class="highlight bash" style="box-sizing: border-box; margin: 10px 0px 20px; padding: 15px; overflow: auto; font-size: 13px; color: rgb(36, 41, 46); background: rgb(246, 248, 250); line-height: 1.8;"><table style="box-sizing: border-box; border-collapse: collapse; border-spacing: 0px; margin: 0px; display: block; width: auto; overflow: auto; border: none;"><tbody style="box-sizing: border-box;"><tr style="box-sizing: border-box; background-color: transparent; border-top: none;"><td class="code" style="box-sizing: border-box; padding: 0px; text-align: left; border: none !important;"><pre style="box-sizing: border-box; margin: 0px; padding: 1px; font-family: SFMono-Regular, Consolas, &quot;Liberation Mono&quot;, Menlo, Courier, monospace; overflow-wrap: normal; overflow: auto; line-height: 1.8; background: rgb(246, 248, 250); border-radius: 3px; font-size: 13px; color: rgb(36, 41, 46); border: none;"><span class="line" style="box-sizing: border-box; height: 20px;">-1 :  11111111</span><br style="box-sizing: border-box;"><span class="line" style="box-sizing: border-box; height: 20px;">00000001(原码)    11111110(取反)    11111111(加一)</span><br style="box-sizing: border-box;"><span class="line" style="box-sizing: border-box; height: 20px;">-128：    </span><br style="box-sizing: border-box;"><span class="line" style="box-sizing: border-box; height: 20px;">10000000(原码)    01111111(取反)    10000000(加一)</span><br style="box-sizing: border-box;"><span class="line" style="box-sizing: border-box; height: 20px;"></span><br style="box-sizing: border-box;"><span class="line" style="box-sizing: border-box; height: 20px;">-1 + 1 = 0</span><br style="box-sizing: border-box;"><span class="line" style="box-sizing: border-box; height: 20px;">11111111 + 00000001 = 00000000(最高位溢出省略)</span><br style="box-sizing: border-box;"><span class="line" style="box-sizing: border-box; height: 20px;">-128 + 127 = -1</span><br style="box-sizing: border-box;"><span class="line" style="box-sizing: border-box; height: 20px;">10000000 + 01111111 = 11111111</span><br style="box-sizing: border-box;"></pre></td></tr></tbody></table></figure></div></details>

1. 下列代码的输出是：

```
func main() {
	const a int8 = -1
	var b int8 = -128 / a
	fmt.Println(b)
}
```

<details open="" style="box-sizing: border-box; margin-top: 10px; margin-bottom: 10px; padding: 5px 10px; border-width: 1px; border-style: solid; border-color: rgb(227, 227, 227) rgb(236, 236, 236) rgb(224, 224, 224) rgb(227, 227, 227); border-image: initial; background-color: rgb(240, 248, 255); box-shadow: rgba(0, 0, 0, 0.07) 1px 2px 1px;"><summary style="box-sizing: border-box; cursor: pointer; font-weight: bold; user-select: none;">答案</summary><div style="box-sizing: border-box;"><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;">编译失败：constant 128 overflows int8</p><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;">-128 和 a 都是常量，在编译时求值，-128 / a = 128，两个常量相除，结果也是一个常量，常量类型转换时不允许溢出，因而编译失败。</p></div></details>

## 作用域

1. 下列代码的输出是：

```
func main() {
	var err error
	if err == nil {
		err := fmt.Errorf("err")
		fmt.Println(1, err)
	}
	if err != nil {
		fmt.Println(2, err)
	}
}
```

<details open="" style="box-sizing: border-box; margin-top: 10px; margin-bottom: 10px; padding: 5px 10px; border-width: 1px; border-style: solid; border-color: rgb(227, 227, 227) rgb(236, 236, 236) rgb(224, 224, 224) rgb(227, 227, 227); border-image: initial; background-color: rgb(240, 248, 255); box-shadow: rgba(0, 0, 0, 0.07) 1px 2px 1px;"><summary style="box-sizing: border-box; cursor: pointer; font-weight: bold; user-select: none;">答案</summary><div style="box-sizing: border-box;"><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;">1 err</p><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;"><code style="box-sizing: border-box; font-family: SFMono-Regular, Consolas, &quot;Liberation Mono&quot;, Menlo, Courier, monospace; padding: 0.2em 0px; margin: 0px; background-color: rgba(27, 31, 35, 0.05); border-radius: 3px; color: rgb(36, 41, 46); font-size: 0.9em;">:=</code><span>&nbsp;</span>表示声明并赋值，<code style="box-sizing: border-box; font-family: SFMono-Regular, Consolas, &quot;Liberation Mono&quot;, Menlo, Courier, monospace; padding: 0.2em 0px; margin: 0px; background-color: rgba(27, 31, 35, 0.05); border-radius: 3px; color: rgb(36, 41, 46); font-size: 0.9em;">=</code><span>&nbsp;</span>表示仅赋值。</p><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;">变量的作用域是大括号，因此在第一个 if 语句<span>&nbsp;</span><code style="box-sizing: border-box; font-family: SFMono-Regular, Consolas, &quot;Liberation Mono&quot;, Menlo, Courier, monospace; padding: 0.2em 0px; margin: 0px; background-color: rgba(27, 31, 35, 0.05); border-radius: 3px; color: rgb(36, 41, 46); font-size: 0.9em;">if err == nil</code><span>&nbsp;</span>内部重新声明且赋值了与外部变量同名的局部变量 err。对该局部变量的赋值不会影响到外部的 err。因此第二个 if 语句<span>&nbsp;</span><code style="box-sizing: border-box; font-family: SFMono-Regular, Consolas, &quot;Liberation Mono&quot;, Menlo, Courier, monospace; padding: 0.2em 0px; margin: 0px; background-color: rgba(27, 31, 35, 0.05); border-radius: 3px; color: rgb(36, 41, 46); font-size: 0.9em;">if err != nil</code><span>&nbsp;</span>不成立。所以只打印了<span>&nbsp;</span><code style="box-sizing: border-box; font-family: SFMono-Regular, Consolas, &quot;Liberation Mono&quot;, Menlo, Courier, monospace; padding: 0.2em 0px; margin: 0px; background-color: rgba(27, 31, 35, 0.05); border-radius: 3px; color: rgb(36, 41, 46); font-size: 0.9em;">1 err</code>。</p></div></details>

## defer 延迟调用

1. 下列代码的输出是：

```
type T struct{}

func (t T) f(n int) T {
	fmt.Print(n)
	return t
}

func main() {
	var t T
	defer t.f(1).f(2)
	fmt.Print(3)
}
```

<details open="" style="box-sizing: border-box; margin-top: 10px; margin-bottom: 10px; padding: 5px 10px; border-width: 1px; border-style: solid; border-color: rgb(227, 227, 227) rgb(236, 236, 236) rgb(224, 224, 224) rgb(227, 227, 227); border-image: initial; background-color: rgb(240, 248, 255); box-shadow: rgba(0, 0, 0, 0.07) 1px 2px 1px;"><summary style="box-sizing: border-box; cursor: pointer; font-weight: bold; user-select: none;">答案</summary><div style="box-sizing: border-box;"><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;">132</p><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;">defer 延迟调用时，需要保存函数指针和参数，因此链式调用的情况下，除了最后一个函数/方法外的函数/方法都会在调用时直接执行。也就是说<span>&nbsp;</span><code style="box-sizing: border-box; font-family: SFMono-Regular, Consolas, &quot;Liberation Mono&quot;, Menlo, Courier, monospace; padding: 0.2em 0px; margin: 0px; background-color: rgba(27, 31, 35, 0.05); border-radius: 3px; color: rgb(36, 41, 46); font-size: 0.9em;">t.f(1)</code><span>&nbsp;</span>直接执行，然后执行<span>&nbsp;</span><code style="box-sizing: border-box; font-family: SFMono-Regular, Consolas, &quot;Liberation Mono&quot;, Menlo, Courier, monospace; padding: 0.2em 0px; margin: 0px; background-color: rgba(27, 31, 35, 0.05); border-radius: 3px; color: rgb(36, 41, 46); font-size: 0.9em;">fmt.Print(3)</code>，最后函数返回时再执行<span>&nbsp;</span><code style="box-sizing: border-box; font-family: SFMono-Regular, Consolas, &quot;Liberation Mono&quot;, Menlo, Courier, monospace; padding: 0.2em 0px; margin: 0px; background-color: rgba(27, 31, 35, 0.05); border-radius: 3px; color: rgb(36, 41, 46); font-size: 0.9em;">.f(2)</code>，因此输出是 132。</p></div></details>

1. 下列代码的输出是：

```
func f(n int) {
	defer fmt.Println(n)
	n += 100
}

func main() {
	f(1)
}
```

<details open="" style="box-sizing: border-box; margin-top: 10px; margin-bottom: 10px; padding: 5px 10px; border-width: 1px; border-style: solid; border-color: rgb(227, 227, 227) rgb(236, 236, 236) rgb(224, 224, 224) rgb(227, 227, 227); border-image: initial; background-color: rgb(240, 248, 255); box-shadow: rgba(0, 0, 0, 0.07) 1px 2px 1px;"><summary style="box-sizing: border-box; cursor: pointer; font-weight: bold; user-select: none;">答案</summary><div style="box-sizing: border-box;"><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;">1</p><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;">打印 1 而不是 101。defer 语句执行时，会将需要延迟调用的函数和参数保存起来，也就是说，执行到 defer 时，参数 n(此时等于1) 已经被保存了。因此后面对 n 的改动并不会影响延迟函数调用的结果。</p></div></details>

1. 下列代码的输出是：

```
func main() {
	n := 1
	defer func() {
		fmt.Println(n)
	}()
	n += 100
}
```

<details open="" style="box-sizing: border-box; margin-top: 10px; margin-bottom: 10px; padding: 5px 10px; border-width: 1px; border-style: solid; border-color: rgb(227, 227, 227) rgb(236, 236, 236) rgb(224, 224, 224) rgb(227, 227, 227); border-image: initial; background-color: rgb(240, 248, 255); box-shadow: rgba(0, 0, 0, 0.07) 1px 2px 1px;"><summary style="box-sizing: border-box; cursor: pointer; font-weight: bold; user-select: none;">答案</summary><div style="box-sizing: border-box;"><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;">101</p><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;">匿名函数没有通过传参的方式将 n 传入，因此匿名函数内的 n 和函数外部的 n 是同一个，延迟执行时，已经被改变为 101。</p></div></details>

1. 下列代码的输出是：

```
func main() {
	n := 1
	if n == 1 {
		defer fmt.Println(n)
		n += 100
	}
	fmt.Println(n)
}
```

<details open="" style="box-sizing: border-box; margin-top: 10px; margin-bottom: 10px; padding: 5px 10px; border-width: 1px; border-style: solid; border-color: rgb(227, 227, 227) rgb(236, 236, 236) rgb(224, 224, 224) rgb(227, 227, 227); border-image: initial; background-color: rgb(240, 248, 255); box-shadow: rgba(0, 0, 0, 0.07) 1px 2px 1px;"><summary style="box-sizing: border-box; cursor: pointer; font-weight: bold; user-select: none;">答案</summary><div style="box-sizing: border-box;"><figure class="highlight plain" style="box-sizing: border-box; margin: 10px 0px 20px; padding: 15px; overflow: auto; font-size: 13px; color: rgb(36, 41, 46); background: rgb(246, 248, 250); line-height: 1.8;"><table style="box-sizing: border-box; border-collapse: collapse; border-spacing: 0px; margin: 0px; display: block; width: auto; overflow: auto; border: none;"><tbody style="box-sizing: border-box;"><tr style="box-sizing: border-box; background-color: transparent; border-top: none;"><td class="code" style="box-sizing: border-box; padding: 0px; text-align: left; border: none !important;"><pre style="box-sizing: border-box; margin: 0px; padding: 1px; font-family: SFMono-Regular, Consolas, &quot;Liberation Mono&quot;, Menlo, Courier, monospace; overflow-wrap: normal; overflow: auto; line-height: 1.8; background: rgb(246, 248, 250); border-radius: 3px; font-size: 13px; color: rgb(36, 41, 46); border: none;"><span class="line" style="box-sizing: border-box; height: 20px;">101</span><br style="box-sizing: border-box;"><span class="line" style="box-sizing: border-box; height: 20px;">1</span><br style="box-sizing: border-box;"></pre></td></tr></tbody></table></figure><p style="box-sizing: border-box; margin: 10px 0px 0px; padding: 0px;">先打印 101，再打印 1。defer 的作用域是函数，而不是代码块，因此 if 语句退出时，defer 不会执行，而是等 101 打印后，整个函数返回时，才会执行。</p></div></details>

------

参考：[Go 语言笔试面试题(代码输出) | 极客面试 | 极客兔兔 (geektutu.com)](https://geektutu.com/post/qa-golang-c1.html)