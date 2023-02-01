# Pinia基础

> 一个全新的用于Vue的状态管理库。
> 
> 下一个版本的Vuex，也就是Vuex 5.0



## Pinia介绍

- Vue2和Vue3都支持
  
  - 除了初始化安装和SSR配置之外，两者的API都是相同的
  
  - 官方文档中主要针对Vue 3进行说明，必要的时候会提供Vue 2的注释

- 支持Vue DevTools
  
  - 跟踪actions、mutations的时间线
  
  - 在使用容器的组件中就可以观察到容器本身
  
  - 支持time travel更容易的调试功能
  
  - 在Vue2中Pinia使用Vuex的现有接口，所以不能与Vuex一起使用
  
  - 但是针对Vue3中的调试工具支持还不够完美，比如还没有time-travel调试功能

- 模板热更新
  
  - 无需重新加载页面即可修改您的容器
  
  - 热更新的时候保持任何现有状态

- 支持使用插件扩展Pinia功能

- 相比Vuex有更好完美的TypeScript支持

- 支持服务器端渲染



### Pinia核心概念

Pinia从使用角度和之前的 Vuex 几乎是一样的，比 Vuex 更简单了。



在Vuex中有四个概念

- State

- Getters

- Mutations

- Actions

在Pinia中

- State

- Getters

- Actions 同步和异步都支持



*Pinia中没有Mutations*



Store是一个保存状态和业务逻辑的实体，它从不绑定到您的组件树。换句话说，它承载全局state。它有点像一个始终存在的组件，每个人都可以读取和写入。它有三个核心概念



- state ： 类似组件的data，用来存储全局状态
  
  ```json
  {
      todos:[
          {id:1,title:"吃饭",done:false},
          {id:2,title:"吃饭",done:false}
      ]
  }
  ```

- getters 类似组件的computed，根据已有State封装派生数据，也具有缓存的特性

```
doneCount(){
      return todos.filter(item=>item.done).length
}
```

- actions 类似组件的methods，用来封装业务逻辑，同步和异步都可以
  
  - 在 Vuex 中同步操作用mutations，异步操作用actions，太麻烦了！ 



Pinia API 与 Vuex <= 4有很大的不同，即：

- 没有mutations。mutations被认为是非常冗长的。最初带来了devtools集成，但这不再是问题

- 不再有模块的嵌套结构。您仍然可以通过在另一个store中导入和使用store来隐式嵌套store，但Pinia通过设计提供扁平结构，同时仍然支持store之间交叉组合方式。您甚至可以拥有store的循环依赖关系。

- 更好typescript支持。无需创建自定义的复杂包装器来支持TypeScript，所有内容都是类型化的，并且API的设计方式尽可能地利用TS类型推断。

- 不再需要注入、导入函数、调用它们，享受自动补全

- 无需动态添加stores，默认情况下它们都是动态的，您甚至不会注意到。请注意，您仍然可以随时手动使用store注册它，但是因为它是自动的，所以您无需担心。

- 没有命名空间模块。鉴于store的扁平架构，“命名空间”store是其方式所固有的，您可以说所有stores都是命名空间。。


