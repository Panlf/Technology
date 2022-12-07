## Go并发

### channel

#### 信道

信道可以想像成 Go 协程之间通信的管道。如同管道中的水会从一端流到另一端，通过使用信道，数据也可以从一端发送，在另一端接收。

#### 声明

的。

`chan T` 表示 `T` 类型的信道。

信道的零值为 `nil`。信道的零值没有什么用，应该像对 map 和切片所做的那样，用 `make` 来定义信道。

```go
package main

import "fmt"

func main() {
 //var a chan int
 a := make(chan int)
 if a == nil {
  	fmt.Println("channel a is nil, going to define it")
 	fmt.Printf("Type of a is %T", a)
 }
}
```

#### 信道发送和接收

```
data := <- a // 读取信道 a
a <- data // 写入信道 a
```

信道旁的箭头方向指定了是发送数据还是接收数据。

在第一行，箭头对于 `a` 来说是向外指的，因此我们读取了信道 `a` 的值，并把该值存储到变量 `data`。

在第二行，箭头指向了 `a`，因此我们在把数据写入信道 `a`。



*发送与接收默认是阻塞的*

发送与接收默认是阻塞的。这是什么意思？当把数据发送到信道时，程序控制会在发送数据的语句处发生阻塞，直到有其它 Go 协程从信道读取到数据，才会解除阻塞。与此类似，当读取信道的数据时，如果没有其它的协程把数据写入到这个信道，那么读取过程就会一直阻塞着。

```go
package main

import (
	"fmt"
)

func numAdd(number int, add chan int){
	if number > 0 {
		number += 10
	}
    //将结果写入到add管道
	add <- number
}

func numCalc(number int,calc chan int){
	if number != 0 {
		number *= 2
	}
     //将结果写入到calc管道
	calc <- number
}


func main() {
	add := make(chan int)
	calc := make(chan int)

	number := 15

	go numAdd(number,add)
	go numCalc(number,calc)

    //从管道中读取数据
	addResult,calcResult := <-add,<-calc

	fmt.Println("final result ", addResult+calcResult)


}
```

#### 死锁

使用信道需要考虑的一个重点是死锁。当 Go 协程给一个信道发送数据时，照理说会有其他 Go 协程来接收数据。如果没有的话，程序就会在运行时触发 panic，形成死锁。

同理，当有 Go 协程等着从一个信道接收数据时，我们期望其他的 Go 协程会向该信道写入数据，要不然程序就会触发 panic。

```
package main

func main() {
 ch := make(chan int)
 ch <- 5
}
```

#### 单向信道

单向信道只能发送或者接收数据。

```go
package main

import "fmt"

func sendData(sendch chan<- int) {
 sendch <- 10
}

func main() {
 sendch := make(chan<- int)
 go sendData(sendch)
 fmt.Println(<-sendch)
}
```

#### 关闭信道

数据发送方可以关闭信道，通知接收方这个信道不再有数据发送过来。

当从信道接收数据时，接收方可以多用一个变量来检查信道是否已经关闭。

```
v, ok := <- ch
```

如果成功接收信道所发送的数据，那么 `ok` 等于 true。而如果 `ok` 等于 false，说明我们试图读取一个关闭的通道。从关闭的信道读取到的值会是该信道类型的零值。



```go
package main

import (
 "fmt"
)

func producer(chnl chan int) {
 for i := 0; i < 10; i++ {
  chnl <- i
 }
 close(chnl)
}
func main() {
 ch := make(chan int)
 go producer(ch)
 for {
  v, ok := <-ch
  if ok == false {
   break
  }
  fmt.Println("Received ", v, ok)
 }
}
```

使用`for range`循环重写上述代码

```go
package main

import (
 "fmt"
)

func producer(chnl chan int) {
 for i := 0; i < 10; i++ {
  chnl <- i
 }
 close(chnl)
}
func main() {
 ch := make(chan int)
 go producer(ch)
 for v := range ch {
  fmt.Println("Received ",v)
 }
}
```

 