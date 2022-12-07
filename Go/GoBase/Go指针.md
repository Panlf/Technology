## Go指针

### 定义

指针是一种存储变量内存地址（Memory Address）的变量。

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

