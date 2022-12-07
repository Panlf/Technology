# TypeScript接口

## 接口

接口是对象的状态（属性）和行为（方法）的抽象（描述）。

接口是一种类型、一种规范、一种规则、一种能力、一种约束。

```
//定义一个接口，该接口作为person对象的类型使用，限定或者是约束该对象中的属性数据
interface IUser {
    readonly id: number //readonly只读
    name: string
    age: number
    sex?: string //加上？ 属性可选
}
```

## 函数类型

为了使用接口表示函数类型，我们需要给接口定义一个调用签名。

它就像是一个只有参数列表和返回值类型的函数定义。参数列表里的每个参数都需要名字和类型。

函数类型：通过接口的方式作为函数的类型来使用

```typescript
//定义一个接口，用来作为某个函数的类型使用
interface ISearchFunc {
    //定义一个调用签名
    (source:string,subString:string):booelan
}

//定义一个函数，该类型就是上面定义的接口
const searchString:ISearchFunc = function (source: string,subString: string):boolean {
    //在source字符串中查找subString这个字符串
    return source.search(subString) > -1
}
```

## 类类型

类实现接口

```typescript
/*
类类型：实现接口
1、一个类可以实现多个接口
2、一个接口可以继承多个接口
*/

interface Alarm {
    alert(): any;
}

interface Light {
    lightOn(): void;
    lightOff(): void;
}

class Car implements Alarm {
    alert(){
        console.log('Car alert')
    }
}
```

总结

- 类可以通过接口的方式来定义当前这个类的类型

- 类可以实现一个接口，类也可以实现多个接口，要注意，接口中的内容都要真正的实现
- 接口和接口之间叫继承（使用的是extends关键字），类和接口之间叫实现（使用的是implements）