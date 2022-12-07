## GO语言

###  特点

- 没有"对象"，没有继承多态，没有泛型，没有try/catch
- 有接口，函数式编程，CSP并发模型(goroutine+channel)

###  变量

- 变量类型写在变量名之后
- 编译器可推测变量类型
- 没有char，只有rune
- 原生支持复数类型

####  变量定义

- var a,b,c bool
- var s1,s2 string ="hello","world"
- 可放在函数内，或直接放在包内
- 使用var()集中定义变量
- 让编译器自动决定类型 var a,b,c=1,true,"123"
- 使用 := 定义变量  a,b,c := 1,true,"123" , 只能在函数内使用

####  内建变量类型

- bool，string
- (u)int,(u)int8,(u)int16,(u)int32,(u)int64,uintptr
- byte,rune
- float32,float64,complex64,complex128

####  强制类型转换

- 类型转换是强制的

####  常量的定义

- const filename = "abc.txt"
- const 数值可作为各种类型使用
- const a,b = 3,4
- 使用常量定义枚举类型 普通枚举类型、自增值枚举类型

###  条件

####  if

- if的条件里可以赋值
- if的条件里赋值的变量作用域就在这个if语句里

####  switch

- switch会自动break，除非使用fallthrough
- switch后可以没有表达式

###  循环

- for,if 后面的条件没有括号
- if条件里也可定义变量
- 没有while
- switch不需要break，也可以直接switch多个条件

####  for

- for的条件里不需要括号
- for的条件里可以省略初始条件，结束条件，递增表达式

###  函数

- 返回值类型写在最后面
- 可返回多个值
- 函数作为参数
- 没有默认参数，可选参数

###  指针

- 值传递和引用传递
- go语言只有值传递一种方式

###  数组

- [10]int和[20]int是不同的类型
- 调用func f(arr [10]int)会拷贝数组
- 在go语言中一般不直接使用数组，建议使用切片

###  切片

- slice可以向后扩展，不可以向前扩展
- 添加元素时，如果超越cap，系统会重新分配更大的底层数组
- 由于值传递的关系，必须接收append的返回值

###  map

- 创建 make(map[string]int)
- 获取元素 m[key]
- key不存在时，获得Value类型的初始值
- 用value,ok := m[key] 来判断是否存在key
- 使用range遍历key，或者遍历value对
- 不保证遍历顺序，如需顺序，需手动对key排序
- map使用哈希表，必须可以比较相等
- 除了slice，map，function的内建类型都可以作为key
- Struct类型不包含上述字段，也可以作为key

### 面向对象
- go语言仅支持封装，不支持继承和多态
- go语言没有class，只有struct

#### 结构创建
```
type  treeNode struct {
	value int
	left,right *treeNode
}

func main(){
	var root treeNode
	root = treeNode{value: 3}

	root.left = &treeNode{}
	root.right = &treeNode{5,nil,nil}
	root.right.left = new(treeNode)
	fmt.Println(root)
}
```

不论地址还是结构本身，一律使用.来访问成员

为结构体定义方法
```
func (node treeNode) print(){
	fmt.Println(node.value)
}

调用则可以使用 root.print()
```

指针作为方法接收者
```
//如果接受指针，则直接改掉值
//只有使用指针才可以改变结构内容
//
func (node *treeNode) setValue(value int){
	node.value = value
}
```

### 包
- 每个目录一个包
- main包包含可执行入口
- 为结构定义的方法必须放在同一个包内
- 可以是不同的文件

如何扩充系统类型或者别人的类型
- 使用别名
- 使用组合

### 接口
- 接口由使用者定义

接口的定义
```
type Retriever interface{
	Get(source string) string
}

func download(retriever Retriever) string{
	return retriever.Get("www.imooc.com")
}
```
- 接口变量自带指针
- 接口变量同样采用值传递，几乎不需要使用接口的指针
-  指针接收者实现只能以指针方式使用；值接收者都可

### 函数式编程
- 函数是一等公民: 参数，变量，返回值都可以是函数
- 高阶函数
- 函数 -> 闭包
