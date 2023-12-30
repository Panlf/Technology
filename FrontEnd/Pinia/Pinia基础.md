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

### Pinia特点
- Vue2和Vue3都支持
- 支持Vue DevTools
- 模块热更新
- 支持使用插件扩展Pinia功能
- 相比Vuex有更好完美的TypeScript支持
- 支持服务端渲染

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
- 没有命名空间模块。鉴于store的扁平架构，“命名空间”store是其方式所固有的，您可以说所有stores都是命名空间。

## Pinia基本使用

新建Vite项目
```
pnpm create vite

pnpm install pinia

pnpm run dev
```

main.ts
```
import { createApp } from 'vue'
import './style.css'
import App from './App.vue'
import {createPinia} from 'pinia'

createApp(App).use(createPinia()).mount('#app')

```

HelloWorld.vue
```
<template>
  <h1>{{ mainStore.count }}</h1>

  <hr>

  <h1>{{ mainStore.count10 }}</h1>

  <hr>

  <h1>{{ count }}</h1>
  
  <p>
    <button @click="handleChangeState">修改数据</button>
  </p>
</template>

<script setup lang="ts">
import { storeToRefs } from 'pinia'
import { useMainStore } from '../store/index'

const mainStore = useMainStore()

// 这是有问题的，因为这样拿到的数据不是响应式的，是一次性的
// Pinia 其实就是把 state 数据都做了 reactive 处理了 
// const {count} = mainStore

// 把解构出来的数据做 ref 响应式处理
const {count} = storeToRefs(mainStore)

const handleChangeState = ()=>{
  // 方式一：最简单的方式
   // mainStore.count++

   // 方式二：如果需要修改多个数据，建议使用 $patch 批量更新
   //mainStore.$patch({
   //  count: mainStore.count + 1
   //})

   // 方式三：$patch 一个函数
   //mainStore.$patch(state=>{
   //   state.count++
   //})

   //方式四：逻辑比较多的时候可以封装 actions 做处理
   mainStore.changeState(10)
}
</script>


<style scoped>

</style>
```

index.ts
```
import { defineStore } from 'pinia'

// 1. 定义并导出容器
// 参数1：容器的ID，必须唯一，将来 Pinia 会把所有的容器挂载到根容器
// 参数2：选项对象
// 返回值：一个函数，调用得到容器实例
export const useMainStore = defineStore('main',{
    /**
     * 类似于组件的data，用来存储全局状态的
     * 1. 必须是函数：这样是为了在服务端渲染的时候避免交叉请求导致的数据状态污染
     * 
     * 2. 必须是箭头函数，这是为了更好的TS类型推导
     */
    state: ()=>{
        return {
            count: 100
        }
    },

    /**
     * 类似于组件的 computed。用来封装计算属性，有缓存功能
     */
    getters: {
        // 函数接受一个可选参数：state状态对象
        //count10(state) {
        //   return state.count += 5
        //}

        //如果在 getters 中使用了 this 则必须手动指定返回值的类型，否则类型推导不出来
        count10():number{
            return this.count + 10
        }
    },

    /**
     * 类似于组件的methods，封装业务逻辑，修改 state
     */
    actions:{
        changeState(num:number){
            this.count += num;
        }
    }
})

//2. 使用容器中的 state

//3. 修改state

//4. 容器中的action的使用
```