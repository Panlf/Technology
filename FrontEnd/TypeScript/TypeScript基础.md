# TypeScript基础

> TypeScript是JavaScript类型的超集，它可以编译成纯JavaScript。TypeScript可以在任何浏览器、任何计算器和任何操作系统上运行，并且是开源的。

## 安装编译
```powershell
npm install -g typescript

tsc helloword.ts
```
## TS配置文件
```json
//ts编译器的配置文件
{
  //用来指定哪些ts文件需要被编译
  //**表示任意目录
  //*表示任意文件
  "include": [
    "./src/**/*"
  ],
  //不需要被编译的文件目录
  // 默认值 ["node_modules"],["brower_components"],["jspm_packages"]
  //"exclude": [],
  //compilerOptions编译器选项
  "compilerOptions": {
    // 默认是ES3 用来指定ts被编译后的ES版本
    //es3、es5、es6 es2015...es2021
    "target": "es6",
    //module 指定要使用的模块化的规范
    //none commonjs amd system umd es6 es2015 es2020 esnext
    "module": "system",
    //lib 用来指定项目中要使用的库
    //"lib": [],
    // 用来指定编译后文件所在的目录 
    "outDir": "./dist",

    //将代码合并为一个文件
    // 设置outFile后，所在的全局作用域中的代码会合并到同一个文件中 
    "outFile": "./dist/app.js",

    // 是否对js文件进行编译 默认false
    "allowJs": true,
    //是否检查js代码是否符合语法规范 默认false
    "checkJs": true,
    //是否移除注释
    "removeComments": true,
    //不生成编译后的文件
    "noEmit": false,
    //当有错误时不生成编译后的文件
    "noEmitOnError": true,

    // 所有严格检查的总开关
    "strict": true,

    //用来设置编译后的文件是否使用严格模式，默认false
    "alwaysStrict": true,
    // 不允许隐式的any类型
    "noImplicitAny": true,
    //不允许不明确类型的this
    "noImplicitThis": true,
    //严格的检查空置
    "strictNullChecks": true
  }
}
```
## 基础数据类型
### 布尔、数字、字符串、数组
```javascript
let a:number=1
let b:string='a'
let c:boolean=true
let d:undefined=undefined
let e:null=null

let f:Array<string> = ["1","2"]
let g:number[]=[1,2,3]
```
### 元组
```javascript
//元组  元组就是固定长度的数组
//语法:[类型,类型]
let type_m:[string,string]
type_m=['hello','world']
```
### 枚举
补充数据类型，生成一个相互引用的对象
```javascript
enum msg {
	"success"=200,
    "error" //201
}
```
### any
任意类型
```javascript
let a:any=1
```
### void
没有值，最佳使用场景：规范一个函数没有返回值
```javascript
let vTs:void
vTs = undefined
```
### Function
```javascript
let fun:Function = () =>{}
fun = function(){
   console.log(1)
}
```
## 接口
使用接口（Interfaces）来定义对象的类型。除了可用于对类的一部分行为进行抽象以外，也常用于对对象数据进行描述
使用interface关键字定义一个必须包含name属性且值是string类型的接口
```javascript
interface IUserName{
	name:string
}

let userName:IUserName = {
   name:"Tina"
}
```
### 可选属性
接口里常规定义的属性都是必须的，可以通过`?`标记一个非必须的属性
```javascript
interface IUserName{
	name:string
  age?:number
}

let userName:IUserName = {
   name:"Tina",
   age:12
}

```
### 只读属性
`readonly`指定属性只能在定义的时候赋值，后期使用只能被读取不允许被修改
```javascript
interface IUserName{
	name:string
  age?:number
  readonly type:number
}

let userName:IUserName = {
   name:"Tina",
   age:12,
   type:1
}
```
### 函数属性
```javascript
interface IUserName{
	name:string
  age?:number
  readonly type:number
  sayHi():void
}

let userName:IUserName = {
   name:"Tina",
   age:12,
   type:1,
   sayHi:()=>console.log("hi")
}
```
### 额外属性
```javascript
interface IUserName{
	name:string
  age?:number
  readonly type:number
  sayHi():void
  [propsName:string]:any
}

let userName:IUserName = {
   name:"Tina",
   age:12,
   type:1,
   sayHi:()=>console.log("hi"),
   phone:"123456"
}
```
### 继承接口
接口继承就是从一个接口里复制成员到另一个接口里，可以更灵活地将接口分割到可重用的模块里。
```javascript
interface IUserName {
	name:string
}

interface IUserId {
	id:number
}

interface IUserInfo extends IUserName,IUserId {
	type:number
}

let userInfo:IUserInfo = {
  name:"tina",
  id:1,
  type:1
}
```
### 混合类型
用于规定一个函数内部地数据结构
```javascript
interface ICount {
		count:number
    ():void
}

let getCount = function():ICount {
    let fun = <ICount>function(){
      fun.count++
      console.log(fun)
    }
  fun.count = 1
  return fun
}
```
## 类
类就是基于面向对象的方式实现常规的构造函数创建对象的另一种实现
使用js是使用构造函数基于原型的继承来创建可复用的对象
```javascript
class UserInfo {
	userName:string
  constructor(name:string){
    this.userName=name
  }
  getName(){
    return `当前的名字是:${this.userName}`
  }
}

let userInfo = new UserInfo("Tina")
userInfo.getName()

// 继承
class NUserInfo extends UserInfo{
	constructor(name:string){
    super(name)
  }
}
```
## 函数
```javascript
//1. number3是而可选类型
//2. 可以不传，如果传递了，那就必须按照要求的类型去传
//3. 可选参数需要定义在必传参数的后面
function getNum(number1:number,number2:number,number3?:number){
   
}


function getNum(number1:number,number2:number,...numbers:number[]){
   
}
```
## 泛型
```javascript
function getNum<T>(number1:T):T{
  return number1
}
```
### 泛型接口
```javascript
interface ConfigInt{
	<T>(userName:T):T
}
const userInfo:ConfigInt<string> = (userName) => userName
userInfo("萧萧")
```
### 泛型约束类
```javascript
class UserInfo<T>{
  userName:T
  id:number
}
const ame = new UserInfo<string>()
ame.userName="潇潇"
ame.id=1
```
## 类型兼容
```javascript
interface UserInfoInt {
	name:string
  id:number
  age:number
}

let userInfo:UserInfoInt

let info = {
  name:"Ame",
  id:1,
  age:19,
  type:"vip"
}
/**
	接口的类型兼容主要看数据的属性
  如果userInfo所需的属性在info里面都能找到，那么类型就兼容
*/
userInfo = info

/**
	函数兼容
  如果x的参数能满足y函数所需，那么类型就兼容
*/
let x = (a:string) => "我有一个参数"
let y = (a:string,b:string) => "我有两个参数"

y = x


class Obj {
	id:number
  constructor(id:number){
    this.id = id
  }
}

class Obj2 {
	id:number
  name:string
  constructor(id:number,name:string){
    this.id = id
    this.name=name
  }
}

let obj1 = new Obj(1)
let obj2 = new Obj2(2,"Ame")

obj1 = obj2
```
## 交叉类型&联合类型
```javascript
//交叉类型
interface type1 {
		name:string
    id:number
}

interface type2 {
   type:string
}

type typeInt = type1 & type2

const userInfoData:typeInt = {
    name:"Ame",
    id:1,
  	type:"vip"
}

//联合类型
const userInfoData1:type1 | type2 = {
  type:"vip",
  name:"maybe",
  id:1
}
```
## 类型断言
```javascript
enum Type {java,js}

class Java {
	helloJava(){
    console.log("java")
  }
}

class JavaScript {
  helloJs(){
    console.log("js")
  }
}

function getLang(type:Type){
	let lang = type === Type.java?new Java():new JavaScript()
  if((lang as Java).helloJava){
    (lang as Java).helloJava()
  }else{
    (lang as JavaScript).helloJs()
  }
  return lang
}
```
## 命名空间
```javascript
namespace info {
	export const type = "VIP"
  export const name = "Ame"
}
```
```javascript
/// <reference path="user.ts">

info.name
```
## 装饰器
```javascript
//装饰器
function initAge(){
  //console.log(arg)
  console.log("装饰器1")
  return (arg: any): void => {
    console.log("装饰器1执行了")
    arg.prototype.age = 18
  }
}

function initSex(){
  console.log("装饰器2")
  return (arg: any): void => {
    console.log("装饰器2执行了")
    arg.prototype.sex = "男"
  };
}

function initAge1(arg: any): void{
  console.log("装饰器1")
  arg.prototype.age = 18
}

function initSex1(arg: any): void{
  console.log("装饰器2")
  arg.prototype.sex = "男"
}

/*
   类装饰器
    类装饰器会在类执行前先一步执行
    类装饰器内部可以拿到类本身
  */
// 多装饰器的类，会按照书写顺序，从下网上去执行装饰器
// 多装饰器的类，会按照书写顺序去执行装饰器进行求值
@initAge()
@initSex()
class Info {
  name: string;
  constructor(str: string) {
    console.log("class内部");
    this.name = str;
  }
}

```
```javascript
function getStr(str:string){
  return (target:any,prototype:any,desc:any){
    target.age=18
    target.name=str
    desc.value = function(){
      return "函数改变了"
    }
  }
}


class Info {
  name: string;
  constructor(str: string) {
    this.name = str;
  }

  @getStr("Ame")
  getName(){
    return this.name
  }
}

```
```javascript
function nameFun(target:any,prototypeKey:any){
    target[prototypeKey]="Hello XXX"
}

class Info {
  @nameFun
  name: string;
  constructor(str: string) {
    this.name = str;
  }
}
```
```javascript
function logPar(target:any,prototypeKey:any,index:any){
  consoloe.log(target)
  consoloe.log(prototypeKey)
  consoloe.log(index)
}

class Info {
  name: string;
  constructor(@logPar str: string) {
    this.name = str;
  }
}
```
