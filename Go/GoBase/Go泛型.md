# Go泛型

泛型（Generics）是指在定义函数、接口或类的时候，不预先指定具体的类型，而在使用的时候再指定类型的一种特性。

golang在1.18实现了泛型的语言特性
- 函数和类型的类型参数
- 将接口定义为类型集，不包括任何的方法类型
- 类型推断，允许调用函数的时候省略类型参数

## 基本语法

使用[T constraint]的语法来声明使用泛型
- 可以声明在接口上
- 可以声明在结构体上
- 可以声明在方法上

```go
type Set[T any] interface {
}

type HashSet[T any] struct {
}


func Print[T any](t T){
    fmt.Printf(“%v”,t)
}
```
