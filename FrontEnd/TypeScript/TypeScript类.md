# TypeScript类

类可以理解为模板，通过模板可以实例化对象

面向对象的编程对象

```typescript
class Person {
	//定义属性
	name: string
	age: number
	gender: string
	
	//定义构造函数：为了将来实例化对象的时候，可以直接对属性的值进行初始化
	constructor(name:string,age:number,gender:string){
		this.name=name
		this.age=age
		this.gender=gender
	}
	
	//定义实例方法
	sayHi(str:string){
		console.log(`大家好，我是${this.name}`,str)
	}
	
	//ts中使用类，实例化对象，可以直接进行初始化操作
	
}

const person = new Person('XXY',20,'男')
person.sayHi('您好啊！！！')
```

## 继承

继承：类与类之间的关系

继承后类与类之间的叫法：A类继承了B这个类，那么此时A类叫子类，B类叫基类

子类 ---> 派生类

基类 ---> 超类（父类）

一旦发生了继承的关系，就出现了父子类的关系

```typescript
class Student extends Person{
	constructor(name:string,age:number,gender:string){
		super(name,age,gender)
	}
	
	//可以调用父类的方法
	sayHi(){
		console.log('我是学生类中的sayHi方法')
		super.sayHi('haha')
	}
}
```

总结

- 类和类之间如果要有继承关系，需要使用extends关键字
- 子类中可以调用父类中的构造函数，使用的是super关键字（包括调用父类中的实例方法，也可以使用super）
- 子类中可以重写父类的方法

## 多态

父类型的引用指向子类型的对象，不同的类型的对象针对相同的方法，产生了不同的行为

## 修饰符

类中的成员的修饰符：主要是描述类中的成员（属性、构造函数、方法）的可访问性，类中的成员都有自己的默认的访问修饰符，public

### 默认public

自由的访问程序里定义的成员。公共的，类中成员默认的修饰符，代表的是公共的，任何位置都可以访问类中的成员

### private

当成员标记成private时，它就不能在声明它的类的外部访问。私有的，类中的成员如果使用private来修饰，那么外部是无法访问这个成员数据的，当然子类也是无法访问该成员数据的。

### protected

protected修饰符与private修饰符的行为很相似，但有一点不同，protected成员在派生类中仍然可以访问。受保护的，类中的成员如果使用protected来修饰，那么外部是无法访问这个成员数据的，当然子类是可以访问该成员数据的。

## readonly修饰符

可以使用readonly关键字将属性设置为只读的。只读属性必须在声明时或者构造函数里被初始化。

readonly对类中的属性成员进行修饰，修饰后该属性成员就不能在外部被随意的修改了。

构造函数中，可以对只读的属性成员的数据进行修改

如果构造函数中没有任何的参数，类中的属性成员此时已经使用readonly进行修饰了，那么外部也是不能对这个属性值进行更改。

构造函数中的参数可以使用readonly进行修饰，一旦修饰了，那么该类中就有了这个只读的成员属性了，外部可以访问，但是不能修改

构造函数中的参数可以使用public及private和protected进行修饰，无论是哪个进行修饰，该类中都会自动的添加这么一个属性成员

## 存取器

TypeScript支持通过getters/setters来截取对对象成员的访问。

```
 class Student implements IPerson {
     firstName:string
     lastName:string

     constructor(firstName: string,lastName: string){
         this.firstName = firstName
         this.lastName = lastName
     }
     //读取器   负责读取数据的
     get fullName(){
        return this.firstName+"."+this.lastName
     }
     //设置器  负责设置数据的(修改)
     set fullName(val){
         let names = val.split(".")
         this.firstName=names[0]
         this.lastName=names[1]
     }

 }

 const student = new Student('周','瑜')
 console.log(student.fullName)
```

## 静态属性

在类中通过static修饰的属性或者方法，那么就是静态的属性及静态的方法，也称之为静态成员

构造函数是不能通过static来进行修饰的

静态成员在使用的时候也是通过类名.的这种语法来调用的

```
static name:string 
...


//访问 通过[类名.静态属性]的方式来访问该成员数据
Person.name
```

## 抽象类

抽象类作为其他派生类的基类使用。它们不能被实例化。不同于接口，抽象可以包含成员的实现细节。abstract关键字是用于定义抽象类和在抽象类内部定义抽象方法。

抽象类：包含抽象方法（抽象方法一般没有任何的具体内容的实现），也可以包含实例方法，抽象类是不能被实例化，为了让子类进行实例化及实现内部的抽象方法。

抽象类的目的或者是作用最终都是为子类服务的

```typescript
abstract class Animal {
 	//抽象方法
	abstract cry()
	//实例方法
	run(){
		console.log('run()')
	}
}

class Dog extends Animal {
	cry(){
	  console.log('Dog cry()')
	}
}
//不能实例化抽象类的对象
const dog = new Dog()
dog.cry()
dog.run()
```

## 函数

函数声明，命名函数

函数中的x和y参数的类型都是string类型，小括号后面的:string，代表的是该函数的返回值也是string类型的

```
function add(x:string,y string): string {
	return x+y
}
```

函数表达式，匿名函数

函数中的x和y参数类型都是number类型的，小括号后面的:number，代表的是该函数的返回值也是number类型的

```
const add2 = function (x:number,y number): number {
	return x+y
}
```

函数的完整写法

```
//add3 ---->  变量名 ----> 函数add3
//(x:number,y:number) => number 当前的这个函数的类型
//function (x:number,y:number):number {return x+y} 就相当于符合上面的这个函数类型的值
const add3:(x:number,y:number) => number = function(x:number,y:number):number{
	return x+y
}
```

## 参数类型

- 可选参数  函数在声明的时候，内部的参数使用了?进行修饰，那么就表示该参数可以传入也可以不传入，叫可选参数

- 默认参数  函数在声明的时候，内部的参数有自己的默认值，此时这个参数就可以叫默认参数

```
const getFullName = function(firstName:string='诸葛',lastName?:string):string{
	if(lastName){
		return firstName+"."+lastName
	}else{
		return firstName
	}
}
```



## 剩余参数

必要参数，默认参数和可选参数有个共同点：它们表示某个参数。有时，你想同时操作多个参数，或者你并不知道会有多少参数传递进来。在JavaScript里，你可以使用arguments来访问所有传入的参数。

在TypeScript里，你可以把所有参数收集到一个变量里：

剩余参数会被当作个数不限的可选参数。可以一个都没有，同样也可以有任意个。编译器创建参数数组，名字是你在省略号（...）后面给定的名字，你可以在函数体内使用这个数组

```
//...argsstring[]  剩余的参数，放在了一个字符串的数组中，args里面
function info(x:string,...args:string[]){
	console.log(x,args)
}

info('abc','c','b','a')
```

## 函数重载

函数名相同，而形参不同的多个函数。

函数名字相同，函数的参数及个数不同

在JS中，由于弱类型的特点和形参与实参可以不匹配，是没有函数重载这一说的，但是在TS中，与其他面向对象的语言就存在此语法。

```
/*
函数重载 函数名相同，而形参不同的多个函数
*/

//重载函数声明
function add(x:string,y:string): string 
function add(x:number,y:number): number 

//定义函数实现
function add(x:string | number , y:string | number):string | number {
	//在实现上我们要注意严格判断两个参数的类型是否相等
	if(typeof x === 'string' && typeof y === 'string'){
		return x+y
	}else if(typeof x === 'number' && typeof y === 'number'){
	return x+y
	}
}


add(1,2)
add('a','b')
```

## 泛型

指在定义函数、接口或类的时候，不预先指定具体的类型，而在使用的时候再指定具体类型的一种特性。

在定义函数、接口、类的时候不能预先确定要使用的数据的类型，而是在使用函数、接口、类的时候才能确定数据的类型

```
function getArr<T>(value:T,count:number):T[]{
	const arr:Array<T>=[]
	for(let i=0;i<count;i++){
		arr.push(value)
	}
	return arr
}

const arr = getArr<number>(200,100)
```

### 多个泛型参数的函数

一个函数可以定义多个泛型参数

```
function swap <K,V> (a:K,b:V):[K,V]{
	return [a,b]
}
const result = swap<string,number>('abc',123)
console.log(result[0].length,result[1].toFixed())
```

### 泛型接口

在定义接口时，为接口中的属性或方法定义泛型类型

在使用接口时，再指定具体的泛型类型

```
//定义一个泛型接口
interface IBaseCURD<T>{
   data: Array<T>
   add: (t:T) => T
   getUserId:(id:number) => T
}

//定义一个用户信息的类
class User {
  id?: number // 用户ID ?属性可有可无
  name: string
  age: number 
}

class UserDao implements IBaseCURD<User> {
	data: Array<User> = []
	add(user: User):User {
		user.id = Date.now()+Math.random()
		this.data.push(user)
		return user
	}
	
	getUserId(id:number): User {
		return this.data.find(user => user.id === id)
	}
}

cosnt userDao:UserDao = new UserDao()
userDao.add(new User('joe',20))
```

### 泛型类

在定义类时，为类中的属性或方法定义泛型类型，在创建类的实例时，再指定特定的泛型类型。

```
//定义一个类，类中的属性值得类型是不确定的，方法中的参数及返回值的类型也是不确定的
//定义一个泛型类
class GenericNumber<T> {
	//默认的属性的值的类型
	defaultValue: T
	add: (x: T,y: T) => T
}

//在实例化类的对象的时候，再确定泛型的类型
const g1:GenericNumber<Number> = new GenericNumber<Number>()
g1.defaultValue =100
g1.add = function(x,y){
	return x+y
}
```



### 泛型约束

如果我们直接对一个泛型参数取length属性会报错，因为这个泛型根本就不知道它有这个属性

```
//如果没有泛型约束
function fn<T>(x: T):void{
	//console.log(x.length) //error
}


//我们可以使用泛型约束来实现
interface Lengthwise {
	length: number
}

//指定泛型约束
function fn2 <T extends Lengthwise>(x:T):void{
	console.log(x.length)
}

//我们需要传入符合约束类型的值，必须包含必须length属性
fn2('abc')
```

