## Select

`select` 语句用于在多个发送/接收信道操作中进行选择。`select` 语句会一直阻塞，直到发送/接收操作准备就绪。如果有多个信道操作准备完毕，`select` 会随机地选取其中之一执行。该语法与 `switch` 类似，所不同的是，这里的每个 `case` 语句都是信道操作

`select` 允许在一个 goroutine 中管理多个 `channel`。但是，当所有 `channel`同时就绪的时候，go 需要在其中选择一个执行。

### 顺序

`select` 不会按照任何规则或者优先级选择就绪的 `channel`。go 标准库在每次执行的时候，都会将他们顺序打乱，也就是说不能保证任何顺序。

随机选取，当 `select` 由多个 case 准备就绪时，将会随机地选取其中之一去执行。

```go
import (
    "fmt"
    "time"
)

func server1(ch chan string) {
    ch <- "from server1"
}
func server2(ch chan string) {
    ch <- "from server2"

}
func main() {
    output1 := make(chan string)
    output2 := make(chan string)
    go server1(output1)
    go server2(output2)
    time.Sleep(1 * time.Second)
    select {
    case s1 := <-output1:
        fmt.Println(s1)
    case s2 := <-output2:
        fmt.Println(s2)
    }
}
```





### 没有就绪 channels

`select` 运行时，如果没有一个 case channel 就绪，那么他就会运行 `default:`,如果 `select` 中没有写 default，那么他就进入等待状态。没有 case 准备就绪时，可以执行 `select` 语句中的默认情况（Default Case）。这通常用于防止 `select` 语句一直阻塞。

```go
//进入等待状态
a := make(chan bool, 100)
b := make(chan bool, 100)
go func() {
   time.Sleep(time.Minute)
   for i := 0; i < 10; i++ {
      a <- true
      b <- true
   }
}()

for i := 0; i < 10; i++ {
   select {
   case <-a:
      print("< a")
   case <-b:
      print("< b")
   }
}
```

如果 `select` 只含有值为 `nil` 的信道，也同样会执行默认情况。

```go
import "fmt"

func main() {
    var ch chan string
    select {
    case v := <-ch:
        fmt.Println("received value", v)
    default:
        fmt.Println("default case executed")

    }
}
```



### 空

```go
package main

func main() {
    select {}
}
```

`select` 语句没有任何 case，因此它会一直阻塞，导致死锁。