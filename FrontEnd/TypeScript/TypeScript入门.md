## 基本类型

### 类型声明

- 类型声明是TS非常重要的一点

- 通过类型声明可以指定TS中变量（参数、形参）的类型

- 指定类型后，当为变量赋值时，TS编译器会自动检查值是否符合类型声明，符合则赋值，否则报错

- 简而言之，类型声明给变量设置了类型，使得变量只能存储某种类型的值

- 语法

  ```
  let 变量:类型;
  
  let 变量名:数据类型 = 值;
  
  function fn(参数: 类型, 参数: 类型):类型{
  	...
  }
  ```

- 自动类型判断

  - TS拥有自动的类型判断机制
  - 当对变量的声明和赋值是同时进行的，TS编译器会自动判断变量的类型
  - 所以如果你的变量的声明和赋值同时进行的，可以省略掉类型声明

- 类型

TS类型种类

| 类型    | 例子            | 描述                             |
| ------- | --------------- | -------------------------------- |
| number  | 1,-12,2         | 任意数字                         |
| string  | 'hello'         | 任意字符串                       |
| boolean | true、false     | 布尔值true或false                |
| 字面量  | 其本身          | 限制变量的值就是该字面量的值     |
| any     | *               | 任意类型                         |
| unknown | *               | 类型安全的any                    |
| void    | 空值(undefined) | 没有值(或undefined)              |
| never   | 没有值          | 不能是任何值                     |
| object  | {name:'张三'}   | 任意JS对象                       |
| array   | [1,2,3]         | 任意JS数组                       |
| tuple   | [4,5]           | 元素，TS新增类型，固定长度的数组 |
| enum    | enum{A,B}       | 枚举，TS中新增类型               |



数组类型

```
// 数组定义方式1
// let 变量名:数据类型[] = [值1,值2,值3]
let arr1: number[] = [10,20,30]

// 数组定义方式2：泛型的写法
// 语法：let 变量名: Array<数据类型> = [值1,值2,值3]
let arr2: Array<number> = [100,200,300]
```

注意问题：数组定义后，里面的数据的类型必须和定义数组的时候的类型是一致的，否则有错误提示信息，也不会编译通过。

元组类型

```
//元组类型：在定义数组的时候，类型和数据的个数一开始就已经限定了
let arr3:[string,number,boolean]=['hello',29,true]
```

注意问题：元组类型在使用的时候，数据的类型的位置和数据的个数，应该和在定义元组的时候的数据类型及位置应该是一致的

枚举类型

```
//枚举类型，枚举里面的每个数据值都可以叫做元素，每个元素都有自己的编号，编号是从0开始的，依次的递增加1
enum Color {
	RED,
	GREEN,
	BLUE
}

//定义一个Color的枚举类型的变量来接收枚举的值
let color: Color = Color.RED
```

any类型

```
let str: any = 100
// 当一个数组中要存储多个数据，个数不确定，类型不确定，此时也可以使用any类型来定义数组
let arr: any[]
```

void类型

```
//void类型，在函数声明的时候，小括号的后面使用:void，代表的是该函数没有任何的返回值
function showMsg():void {
	console.log('hello void')
}
```

object类型

```
//定义一个函数，参数是object类型，返回值也是object类型
function getObj(obj: object): object {
	console.log(obj)
	return {
	  name:'John',
	  age:27
	}
}
```

联合类型

```
//联合类型 表示 取值可以为多种类型中的一种
function toString(x:number | string):string{
	return x.toString()
}
```

类型断言

```
//类型断言：告诉编译器，我知道我自己是什么类型，也知道自己在什么
//类型断言的语法方式1：<类型>变量名
//类型断言的语法方式2：值 as 类型
function getString(str:number | string):number {
	if((<string>str).length){
		//return (<string>str).length
		return (str as string).length
	}else{
		//此时说明str是number类型
		return str.toString().length
	}
}
```



### undefined和null

TypeScript里，undefined和null两者各自有自己的类型分别叫做undefined和null。它们的本身的类型用处不是很大。

```
let u: undefined = undefined
let n: null = null
```

默认情况下，null和undefined是所有类型的子类型。就是说你可以把null和undefined赋值给number类型的变量。

### 类型注解

类型注解是一种轻量级的为函数或者变量添加约束的方式。在这个例子中，如果函数声明接受一个string类型参数，但是如果尝试传入一个数组，就会报错。

```
function sayHi(str:string){
	return 'Hello,'+str
}

let user = [0,1,2]

console.log(sayHi(user))
```



### 编译选项

- 自动编译

  - 编译文件时，使用`-w`指令后，TS编译器会自动监视文件的变化，并在文件发生变化时对文件进行重新编译

  - 示例

    ```
    tsc xxx.ts -w
    ```

- 自动编译整个项目

  - 如果直接使用tsc指令，则可以自动将当前项目下的所有ts文件编译为js文件
  - 但是能直接使用tsc命令前提时，要先在项目根目录下创建一个ts的配置文件tsconfig.json
  - tsconfig.json是一个JSON文件，添加配置文件后，只需tsc命令即可完成对整个项目的编译

