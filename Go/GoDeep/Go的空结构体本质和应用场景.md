# Go的空结构体本质和应用场景


## 空结构体本质
```go
type EST struct {
}

var b EST
var c struct{}

func TestStructs(t *testing.T) {
	var b EST
	var c struct{}

	fmt.Printf("b address %p size %d\n", &b, unsafe.Sizeof(b))
	fmt.Printf("b address %p size %d\n", &c, unsafe.Sizeof(c))
}
```

上述结果，空间占用为0
```
b address 0x528480 size 0
b address 0x528480 size 0
```

## 应用场景

### 替代Set数据结构
Go语言没有Set这种数据结构，用`map`来替代，值用空结构体替代
```go
students := make(map[string]struct{}, 10)
students["张三"] = EST{}
students["李四"] = struct{}{}
fmt.Println(len(students))
```
### 用于协程之间的协调同步

```go
teachers := make(chan struct{}, 0)
go func() {
    time.Sleep(3 * time.Second)
    fmt.Println("子协程工作完毕")
    teachers <- struct{}{}
}()
<-teachers
```
让main去等子协程执行完毕




