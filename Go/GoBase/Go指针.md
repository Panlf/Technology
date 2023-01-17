## Go指针

### 定义

指针是一种存储变量内存地址（Memory Address）的变量。

> 我们都知道，程序运行时的数据是存放在内存中的，每一个存储在内存中的数据都有一个编号，这个编号就是内存地址。我们可以根据这个内存地址来找到内存中存储的数据，而内存地址可以被赋值给一个指针。我们也可以简单的理解为指针就是内存地址。

### 声明

指针变量的类型为 **`*T`**，该指针指向一个 **T** 类型的变量。

```go
import "fmt"

func main(){

	a := 12
	var b *int = &a

	fmt.Printf("Type of b is %T\n", b)
	fmt.Println("b = ",b)

}
```

在Go语言中，获取一个指针，直接使用取地址符&就可以。

```go
func main() {
  name := "我是谁?"
  nameP := &name //取地址
  fmt.Println("name变量值为：", name)
  fmt.Println("name变量的内存地址为：", nameP)
}
```
普通变量存的是数据，指针变量存的是数据的地址

指针的初始化

```go
import "fmt"

func main(){

	var c *int

	fmt.Println("c = ",c)

}

//打印结果 c =  <nil>
```

指针的初始化值是 `nil`。

指针的取值

```go
func main(){

	a := 12

	var c *int = &a

	fmt.Println("c = ",*c)
}

//打印结果 c = 12
```

### 向函数传递指针参数

```go
import "fmt"


func changeValue(val *int) {
	*val = 100
}

func main(){
	var m = 10

	fmt.Println("before m = ",m)

	changeValue(&m)

	fmt.Println("after m = ",m)
}

// 打印结果
// before m =  10
// after m =  100
```

### 不要向函数传递数组的指针，而应该使用切片

```go
import "fmt"

func modify(arr *[3]int) {
	//a[x] 是 (*a)[x] 的简写形式，因此上面代码中的 (*arr)[0] 可以替换为 arr[0]
	//(*arr)[0] = 90
	arr[0] = 90
}

func main(){
	arr := [3]int{89, 90, 91}
	modify(&arr)
	fmt.Println(arr)
}

//打印结果
//[90 90 91]
```

**这种方式向函数传递一个数组指针参数，并在函数内修改数组。尽管它是有效的，但却不是 Go 语言惯用的实现方式。一般使用切片来处理。**

```go
import "fmt"

func modifySlices(sls []int) {
	sls[0] = 90
}

func main(){
	slices := [3]int{89, 90, 91}
	modifySlices(slices[:])
	fmt.Println(slices)
}
//打印结果
//[90 90 91]
```

### 什么情况下使用指针
- 不要对 map、slice、channel 这类引用类型使用指针；
- 如果需要修改方法接收者内部的数据或者状态时，需要使用指针；
- 如果需要修改参数的值或者内部数据时，也需要使用指针类型的参数；
- 如果是比较大的结构体，每次参数传递或者调用方法都要内存拷贝，内存占用多，这时候可以考虑使用指针；
- 像 int、bool 这样的小数据类型没必要使用指针；
- 如果需要并发安全，则尽可能地不要使用指针，使用指针一定要保证并发安全；
- 指针最好不要嵌套，也就是不要使用一个指向指针的指针，虽然 Go 语言允许这么做，但是这会使你的代码变得异常复杂。

