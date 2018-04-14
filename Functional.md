# 函数式编程

go语言将函数作为头等公民，支持函数式编程等特性。

## 闭包
所谓闭包就是一种函数，其引用了外部作用域的变量，导致了在外部作用域执行完毕后仍然保留对其的引用。
首先我们看下面的例子：
```golang
func add() func() int  {
	i := 0
	return func() int {
		return i+1
	}
}
```
`add`函数返回了一个匿名函数，这个函数的功能是把`i`的值加1，我们看看如何使用这个函数：
```golang
func main(){
	a := add()
	a() //1
	a() //2
}
```
在上面的例子中我们用`a`变量去接收`add`返回的匿名函数，然后执行了两次，令人惊讶的是，第二次执行竟然是在第一次的“基础上”执行的。这就是golang的函数式编程中闭包的概念。

## 斐波那契数列
下面我们用闭包来实现一个斐波那契数列：
```golang
type intGen func() int

func Fibonacci() intGen {
	a, b := 0, 1
	return func() int {
		a, b = b, a + b
		return a
	}
}
```
上面我们可以看到，我们用类型声明为`func() int`创建了一个别名，`Fibonacci`函数中的返回值和这样写是等价的`func Fibonacci() func() int`，接着我们看看是如何调用的：
```golang
func main(){
	f := Fibonacci()
	f() //1
	f() //1
	f() //2
}
```
可以看到这个例子和上一个是类似的，每次调用`f`就会返回基于上一次结果的值，因为匿名函数也就是我们`return`语句后面的匿名函数始终保留着对`a`，`b`变量的引用，这就是闭包的特性。

## 函数实现接口
函数作为一等公民，不仅能作为变量保存，也能实现接口。为此我们先介绍一下我们要实现的接口：
- Read接口
```golang
type Reader interface {
	Read(p []byte) (n int, err error)
}
```
`Read`接口用于将`n`个字节读取到`p`的切片中并返回`n`和任何错误。
在下面的例子中我们要让上文中提到的`intGen`函数实现`Read`接口，为了能让其输出的内容被读取（实际上就是为了能让斐波那契数列被读取）：
```golang
func (g intGen) Read(p []byte) (n int, err error)  {
	next := g() //next表示下一个斐波那契数
	//限制读取的次数
	if next > 1000 {
		return 0, io.EOF
	}
	s := fmt.Sprintf("%d\n", next)
	//借用strings实现Read接口
	return strings.NewReader(s).Read(p)
}
```
上面的代码中，我们像为传统结构体实现接口一样为函数`intGen`实现了`Read`接口，下面我们看看如何使用：
```golang
scanner := bufio.NewScanner(reader)
if scanner.Scan() {
		fmt.Println(scanner.Text()) //1
	}
```
至此，我们就完成了函数实现接口的能力了。
