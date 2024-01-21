# Go 语言中， defer的使用时有哪些陷阱？

## 01 介绍

大家好，今天我们来聊聊在Go 语言中， defer的使用时，会有哪些陷阱？


defer 的使用方式是在其后紧跟一个函数调用或方法调用，确保在其所在的函数体返回之前执行其调用的函数或方法。

在 Go 语言中，defer 一般用于资源释放，或使用 defer 调用一个匿名函数，在匿名函数中使用 recover() 处理异常 panic。

在使用 defer 时，也很容易遇到陷阱，本文我们介绍使用 defer 时有哪些陷阱。

## 02 defer 陷阱

defer 语句不可以在 return 语句之后。

示例代码：

```
func main() {
	name := GetUserName("phper")
	fmt.Printf("name:%s\n", name)
	if name != "gopher" {
		return
	}
	defer fmt.Println("this is a defer call")
}

func GetUserName(name string) string {
	return name
}
```
输出结果：
```
name:phper
```
阅读上面这段代码，我们在 return 语句之后执行 defer 语句，通过输出结果可以发现 defer 语句调用未执行。

虽然 defer 可以在函数体中的任意位置，我们也是需要特别注意使用 defer 的位置是否可以执行。

defer 语句执行匿名函数，参数预处理。

示例代码：
```
func main() {
	var count int64
	defer func(data int64) {
		fmt.Println("defer:", data)
	}(count + 1)
	count = 100
	fmt.Println("main:", count)
}
```
输出结果：
```
main: 100
defer: 1
```
阅读上面这段代码，首先我们定义一个类型为 int64 的变量 count，然后使用 defer 语句执行一个匿名函数，匿名函数传递参数为 count + 1，最终 main 函数输出 100，defer 执行的匿名函数输出 1。

因为在执行 defer 语句时，执行了 count + 1，并先将其存储，等到 defer 所在的函数体 main 执行完，再执行 defer 语句调用的匿名函数的函数体中的代码。

## 03 总结

本文主要介绍在使用 defer 语句时可能会遇到的陷阱。分别是 defer 语句不可以在 return 语句之后；defer 语句执行的匿名函数，匿名函数的参数会被预先处理。

