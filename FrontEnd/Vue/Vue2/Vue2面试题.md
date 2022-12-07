# Vue2面试题

## Vue2的生命周期

```
1、有哪些生命周期
    beforeCreate
    created
    beforeMount
    mounted
    beforeUpdate
    updated
    beforeDestroy
    destroyed

2、一旦进入到页面或者组件，会执行哪些生命周期，顺序
    beforeCreate
    created
    beforeMount
    mounted

3、在哪个阶段有$el，在哪个阶段有$data
    beforeCreate  啥也没有
    created 有data没有el
    beforeMount 有data没有el
    mounted 都有

4、如果加入了keep-alive会多两个生命周期
    activated、deactivated

5、如果加入了keep-alive，第一次进入组件会执行哪些生命
    beforeCreate
    created
    beforeMount
    mounted
    activated

6、如果加入了keep-alive，第二次或者第N次进入组件会执行哪些生命周期
    只执行一个生命周期  activated
```

## 谈谈你对kee-alive的了解

1、是什么
Vue系统自带的一个组件，功能：是用来缓存组件的。 ===> 提升性能

2、使用场景
就是来缓存组件，提升项目的性能。具体实现比如：首页进入到详情页，如果用户在首页每次点击都是相同的，
那么详情页就没必要请求N次了，直接缓存起来就可以了，当然如果点击的不是同一个，那么就直接请求。

## v-if和v-show区别

1、展示形式不同
v-if 创建一个dom节点
v-show 是display:none、black

2、使用场景不同
初次加载v-if要比v-show好，页面不会做加载盒子
频繁切换v-show要比v-if好，创建和删除的开销太大了，显示和隐藏的开销较小

## v-if和v-for优先级

v-for的优先级要比v-if高
在源码中体现的：function genElement 

## ref是什么

来获取dom的

## nextTick是什么

获取更新后的dom内容

## scoped原理

作用：让样式在本组件中生效，不影响其他组件

原理：给节点新增自定义属性，然后css根据属性选择器添加样式

## 样式穿透

父元素 /deep/ 子元素

父元素 >>> 子元素

## Vue组件之间如何传值通信

### 父组件传值子组件

父组件
```
<header :msg='msg'></header>
```

子组件
```
props:['msg']

props:{
    msg:数据类型
}
```

### 子组件传值父组件
子组件
```
this.$emit("自定义事件名称",要传的数据);
```

父组件
```
<header @childInput='getVal'></header>

methods:{
    getVal(msg){
        //msg就是子组件传递的数据
    }
}
```

### 兄弟组件之间的传值

通过一个中转 bus

A兄弟传值
```
import bus from '@/common/bus'

bus.$emit('toFooter',this.msg)
```

B兄弟接收
```
import bus from '@/common/bus'

bus.$on('toFooter',(data)=>{
    //data是this.msg数据
})
```

## computed、methods、watch有什么区别

```
1、computed、methods区别
    computed是有缓存的
    methods没有缓存

2、computed、watch区别
    watch是监听，数据或者路由发生了改变才可以响应（执行）
    computed计算某一个属性的改变，如果某一个值改变了，计算属性会监听到进行返回
    watch是当前监听到数据改变了，才会执行内部代码
```

## props和data优先级谁高
```
props ===> methods ===> data ===> computed ===> watch
```

## Vuex面试题
### Vuex有哪些属性
```
state、getters、mutations、actions、modules

state   类似于组件中的data，存放数据
getters 类似于组件中computed
mutations 类似于组件中methods
actions 提交mutations的
modules 把以上4个属性再细分，让仓库更好管理
```

### Vuex是单向数据流还是双向数据流

Vuex是单向数据流

### Vuex中的mutations和actions区别
- mutations 都是同步事务
- actions 可以包含任意异步操作

### Vuex如何做持久化存储
```
Vuex本身不是持久化存储
1、使用localStorage自己写
2、使用vuex-persist插件
```

