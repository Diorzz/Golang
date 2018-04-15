# Channel (管道)
`channel`是用来在两个协程之间通信用的。`channel`是一个类型安全的管道，用于连接两个协程。
```golang
func main(){
  var ch chan int // c == nil
  c := make(chan int) //创建并初始化一个channel c
  c <- 1 //向channel中写数据
  n := <- c //从channel中读取数据
}
```
