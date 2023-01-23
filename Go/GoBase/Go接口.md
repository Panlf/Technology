# Go接口

在Go中，接口是一组方法签名(声明的是一组方法的集合)。当一个类型为接口中的所有方法提供定义时，它被称为实现该接口。它与oop非常相似。接口指定类型应具有的方法，类型决定如何实现这些方法。

## interface 

```go
type Animal interface {
    Speak() string
}
```

非常简单：我们定义 Animal 为任何具有 Speak 方法的类型。Speak 方法没有参数，返回一个字符串。所有定义了该方法的类型我们称它实现了 Animal 接口。Go 中没有 implements 关键字，判断一个类型是否实现了一个接口是完全是自动地。让我们创建几个实现这个接口的类型：

```go
type Dog struct {
}
 
func (d Dog) Speak() string {
    return "Woof!"
}
 
type Cat struct {
}
 
func (c Cat) Speak() string {
    return "Meow!"
}
 
type Pig struct {
}
 
func (p Pig) Speak() string {
    return "Hoho!"
}
 
```

我们现在有四种不同类型的动物：`Dog`、`Cat`和`Pig`。在我们的`main`函数中，我们创建了一个`[]Animal{Dog{}, Cat{}, Llama{}, JavaProgrammer{}}`，看看每只动物都说了些什么：

```go
func main() {
    animals := []Animal{Dog{}, Cat{}, Pig{}}
    for _, animal := range animals {
        fmt.Println(animal.Speak())
    }
}
```

## interface{}
`interface{}`类型，空接口，是导致很多混淆的根源。`interface{}`类型是没有方法的接口。由于没有`implements`关键字，所以所有类型都至少实现了 0 个方法，所以 所有类型都实现了空接口。这意味着，如果您编写一个函数以`interface{}`值作为参数，那么您可以为该函数提供任何值。例如：

```go
func DoSomething(v interface{}) {
   // ...
}
```

这里是让人困惑的地方：在`DoSomething`函数内部，`v`的类型是什么？新手们会认为`v`是任意类型的，但这是错误的。`v`不是任意类型，它是`interface{}`类型。对的，没错！当将值传递给`DoSomething`函数时，Go运行时将执行类型转换(如果需要)，并将值转换为`interface{}`类型的值。所有值在运行时只有一个类型，而`v`的一个静态类型是`interface{} `。

在我们上面的例子中，当我们初始化变量`animals`时，我们不需要像这样`Animal(Dog{})`来显示的转型，因为这是自动地。这些元素都是`Animal`类型，但是他们的底层类型却不相同。

```go
package main
 
import (
    "fmt"
)
 
func PrintAll(vals []interface{}) {
    for _, val := range vals {
        fmt.Println(val)
    }
}
 
func main() {
    names := []string{"stanley", "david", "oscar"}
    PrintAll(names)
}
```

运行这段代码你会得到如下错误：`cannot use names (type []string) as type []interface {} in argument to PrintAll`。如果想使其正常工作，我们必须将`[]string`转为 `[]interface{}`：

```go

import (
    "fmt"
)
 
func PrintAll(vals []interface{}) {
    for _, val := range vals {
        fmt.Println(val)
    }<br>
}
 
func main() {
    names := []string{"stanley", "david", "oscar"}
    vals := make([]interface{}, len(names))
    for i, v := range names {
        vals[i] = v
    }
    PrintAll(vals)
}
```

事实上，这种情况不会经常发生，因为`[]interface{}`并没有像你想象的那样有用。

### 指针和接口

接口的另一个微妙之处是接口定义没有规定一个实现者是否应该使用一个指针接收器或一个值接收器来实现接口。当给定一个接口值时，不能保证底层类型是否为指针。在前面的示例中，我们将方法定义在值接收者之上。让我们稍微改变一下，将`Cat`的`Speak()`方法改为指针接收器：
```go
func (c *Cat) Speak() string {
    return "Meow!"
}
```

运行上述代码，会得到如下错误：
`cannot use Cat literal (type Cat) as type Animal in array or slice literal:
    Cat does not implement Animal (Speak method has pointer receiver)`

该错误的意思是：你尝试将`Cat`转为`Animal`，但是只有`*Cat`类型实现了该接口。你可以通过传入一个指针 （`new(Cat)`或者`&Cat{}`）来修复这个错误。
```go
animals := []Animal{Dog{}, new(Cat), Pig{}}
```

让我们做一些相反的事情：我们传入一个`*Dog`指针，但是不改变`Dog`的`Speak()`方法：
```go
animals := []Animal{new(Dog), new(Cat), Pig{}}
```

这种方式可以正常工作，因为一个指针类型可以通过其相关的值类型来访问值类型的方法，但是反过来不行。即，一个`*Dog`类型的值可以使用定义在 `Dog`类型上的`Speak()`方法，而`Cat`类型的值不能访问定义在`*Cat`类型上的方法。

这可能听起来很神秘，但当你记住以下内容时就清楚了：`Go`中的所有东西都是按值传递的。每次调用函数时，传入的数据都会被复制。对于具有值接收者的方法，在调用该方法时将复制该值。例如下面的方法：

```go
func (t T)MyMethod(s string) {
    // ...
} 
```

因为所有的参数都是通过值传递的，这就可以解释为什么`*Cat`的方法不能被`Cat`类型的值调用了。任何一个`Cat`类型的值可能会有很多`*Cat`类型的指针指向它，如果我们尝试通过`Cat`类型的值来调用`*Cat`的方法，根本就不知道对应的是哪个指针。相反，如果 `Dog`类型上有一个方法，通过`*Dog`来调用这个方法可以确切的找到该指针对应的`Gog`类型的值，从而调用上面的方法。运行时`Go`会自动帮我们做这些，所以我们不需要像 C语言中那样使用类似如下的语句 `d->Speak()`。

## 总结

- 通过考虑数据类型之间的相同功能来创建抽象，而不是相同字段
- `interface{}`的值不是任意类型，而是` interface{}`类型
- 接口包含两个字的大小，类似于`(type, value)`
- 函数可以接受`interface{}`作为参数，但最好不要返回 `interface{}`
- 指针类型可以调用其所指向的值的方法，反过来不可以
- 函数中的参数甚至接受者都是通过值传递
- 一个接口的值就是就是接口而已，跟指针没什么关系
- 如果你想在方法中修改指针所指向的值，使用`*`操作符