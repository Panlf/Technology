# Vue3快速入门

## 创建项目

### 使用vue-cli创建

```
npm install -g @vue/cli

vue --version

vue create my-project
```

### 使用vite创建

```
npm init vite-app <project-name>
cd <project-name>
npm install
npm run dev
```

创建可供选择的项目

```
npm init @vitejs/app <project-name>
```

## setup使用

### setup基本使用

```vue
<template>
    <div>{{ number }}</div>
</template>

<script lang="ts">
    export default {
        name:'App',
        setup(){
            const number = 10
            return {
                number
            }
        }
    }
</script>

```

### setup和ref基本使用

```vue
<template>
    <div>{{ number }}</div>
    <button @click="updateNumber">更新Number</button>
</template>

<script lang="ts">
    import { ref } from 'vue'

    export default {
        name:'App',
        setup(){
            //ref是一个函数，作用：定义一个响应式的数据，
            // 返回的是一个Ref对象，对象中有一个value属性，如果需要对数据进行操作
            // 需要使用该Ref对象调用value属性的方式进行数据的操作
            // html模板中是不需要使用.value属性的写法
            // 一般用来定义一个基本类型的响应式数据
            // number的类型 Ref类型
            const number = ref(0)

            function updateNumber(){
                number.value++
            }

            //返回的是一个对象
            return {
                number,
                updateNumber
            }
        }
    }

</script>

```

## vue2和vue3的响应式

### vue2的响应式

- 核心
  - 对象：通过defineProperty对对象的已有属性值的读取和修改进行劫持（监视/拦截）
  - 数组：通过重写数组更新数组一系列更新元素的方法来实现元素修改的劫持
- 问题
  - 对象直接新添加的属性或者删除已有属性，界面不会自动更新
  - 直接通过下标替换元素或更新length，界面不会自动更新arr[1] = {}

### vue3的响应式

- 核心
  - 通过Proxy（代理）：拦截对data任意属性的任意（13种）操作，包括属性值的读写、属性的添加、属性的删除等
  - 通过Reflect(反射)：动态对被代理对象的相应属性进行特定的操作

```vue
<template>
    <h2>reactive的使用</h2>
    <h3>名字:{{user.name}}</h3>
    <h3>年龄:{{user.age}}</h3>
    <h3>媳妇:{{user.wife}}</h3>
    <button @click="updateUser">更新User</button>
</template>

<script lang="ts">
    import { reactive } from 'vue'

/*
    reactive
    作用：定义多个数据的响应式
    const proxy = reactive(obj) 接收一个普通对象然后返回该普通对象的响应式代理器对象
    响应式转换是"深层的"：会影响对象内部所有嵌套的属性
    内部基于ES6的Proxy实现，通过代理对象操作源对象内部数据都是响应式的
*/
    export default {
        name:'App',
        setup(){

            const obj = {
                name:'小明',
                age: 20,
                wife:{
                    name:'小甜甜',
                    age: 18,
                    cars:['奔驰','宝马','奥迪']
                }
            }

            //把数据变成响应式数据
            //返回的是一个Proxy的代理对象，被代理的目标对象就是obj对象
            //user 现在是代理对象，obj是目标对象
            // user对象的类型是Proxy
            const user = reactive(obj)
            const updateUser = ()=>{
                //直接使用目标对象的方式来更新目标对象中的成员的值，是不可能的，
                //只能使用代理对象的方式来更新数据（响应式数据）
                user.name='小红'
                user.age += 2
            }
            return {
                user,
                updateUser
            }
        }
    }

</script>

```



总结

如果操作代理对象，目标对象中的数据也会随之变化，同时如果想要在操作数据的时候，界面也要跟着重新更新渲染，那么也是操作代理对象

## setup细节

- seup执行的时机
  - 在beforeCreate之前执行（一次），此时组件对象还没有创建
  - this是undefined，不能通过this来访问data/computed/methods/props
  - 其实所有的composition API相关回调函数中也都不可以
- setup的返回值
  - 一般都返回一个对象：为模板提供数据，也就是模板中可以直接使用此对象中的所有属性/方法
  - 返回对象中的属性会与data函数返回的对象的属性合并成为组件对象的属性
  - 返回对象中的方法会与methods中的方法合并成功组件对象的方法
  - 如果有重名，setup优先
  - 注意
  - 一般不要混合使用：methods中可以访问setup提供的属性和方法，但在setup方法中不能访问data和methods
  - setup不能是一个async函数：因为返回值不再是return对象，而是promise，模板看不到return对象中的属性数据
- setup参数
  - setup(props,context) / setup(props,{attrs,slots,emit})
  - props 包含props配置声明且传入了的所有属性的对象
  - attrs 包含没有在props配置中声明的属性的对象，相当于this.$attrs
  - slots 包含所有传入的插槽内容的对象，相当于this.$slots
  - emit 用来分发自定义事件的函数，相当于this.$emit

App.vue

```vue
<template>
    <h2>父组件</h2>
    <h3>父组件msg:{{ msg }}</h3>
    <!-- //@子组件的方法="父组件的方法" -->
    <children :msg = "msg" msg2="黑恶势力" @childrenUpdate="parentUpdate"></children>
</template>

<script lang="ts">
import { ref } from 'vue'

    import Children from './components/Children.vue'

    export default {
        name:'App',
        components: {
            Children
        },
        setup(){
            const msg = ref('我是父组件的消息')
            function parentUpdate(txt:string){
                console.log('parentUpdate:txt',txt)
                msg.value += txt
            }
            return {
                msg,
                parentUpdate
            }
        }
        
    }

</script>

```

Children.vue

```vue
<template>
    <h2>我是子组件</h2>
    <h3>子组件msg:{{msg}}</h3>
    <button @click="updateMsg">分发数据</button>
</template>

<script>
export default {
    name:'Children',
    props:['msg'],
    // setup细节问题
    // setup是在beforeCreate生命周期回调之前就执行了，而且就执行一次
    // 数据初始化的生命周期回调

    // 由此可以推断出：setup在执行的时候，当前的组件还没有创建出来，也就意味着：组件实例对象this根本就不能用
    // this是undefined，说明就不能通过this再去调用data/computed/methods中的相关内容了
    beforeCreate(){
        console.log('beforeCreate执行了')
    },
    //界面渲染完毕
    //mounted(){},
    setup(props,{attrs,slots,emit}){
        console.log('setup执行了',this)

        console.log(attrs.msg2)

        function updateMsg(){
            console.log('updateMsg')
            emit('childrenUpdate','children')
        }

        return {
            //setup中一般都是返回一个对象，对象中的属性和方法都可以在html模板中直接使用
            //setup中的对象内部的属性和data函数中的return对象的属性都可以在html模板中使用
            //setup中的对象中的属性和data函数中的对象中的属性会合并为组件对象的属性
            //setup中的对象中的方法和methods对象中的方法会合并为组件对象的方法
            //在Vue3中尽量不要混合的使用data和setup及methods和setup
            //一般不要混合使用：methods中可以访问setup提供的属性和方法，但在setup方法中不能访问data和methods
            //setup不能是一个async函数：因为返回值不再是return的对象，而是promise，模板看不到return对象中的属性数据
        updateMsg
     }
    }
}
</script>

<style>

</style>
```



## reactive与ref-细节

- 是Vue3的composition API中2个最重要的响应式API
- ref用来处理基本数据类型，reactive用来处理对象（递归深度响应式）
- 如果用ref对象/数组，内部会自动将对象/数组转换为reactive的代理对象
- ref内部：通过给value属性添加getter/setter来实现对数据的劫持
- reactive内部：通过使用Proxy来实现对象内部所有数据的劫持，并通过Reflect操作对象内部数据
- ref的数据操作：在js中要.value，在模板中不需要（内部解析模板时会自动添加.value）

## 计算属性与监视

- computed函数
  - 与computed配置功能一致
  - 只有getter
  - 有getter和setter
- watch函数
  - 与watch配置功能一致
  - 监视指定的一个或多个响应式数据，一旦数据变化，就自动执行监视回调
  - 默认初始时不执行回调，但可以通过配置immediate为true，来指定初始时立即执行第一次
  - 通过配置deep为true，来指定深度监视
- watchEffect函数
  - 不用直接指定要监视的数据，回调函数中使用的哪些响应式数据就监视哪些响应式数据
  - 默认初始时就会执行第一次，从而可以收集需要监视的数据
  - 监视数据发生变化时回调

```vue
<template>
    <h2>计算属性和监视</h2>
    <fieldset>
        <legend>姓名操作</legend>
        姓氏：<input type="text" placeholder="请输入姓氏" v-model="firstName"/><br>
        名字：<input type="text" placeholder="请输入名字" v-model="lastName" /><br>
    </fieldset>
    <fieldset>
        <legend>计算属性和监视的演示</legend>
        姓名<input type="text" placeholder="显示姓名" v-model="fullName1" /><br>
        姓名<input type="text" placeholder="显示姓名" v-model="fullName2"/><br>
        姓名<input type="text" placeholder="显示姓名" v-model="fullName3" /><br>
    </fieldset>

</template>

<script lang="ts">
    import { reactive,toRefs,computed,watch, ref, watchEffect } from 'vue'
    
    export default {
        name:'App',
        components: {
            
        },
        setup(){
            //定义一个响应式对象
            const user = reactive({
                firstName:'东方',
                lastName:'不败'
            })

            //通过计算属性的方式，实现第一个姓名的显示
            //返回的是一个Ref类型的对象
            const fullName1 = computed(()=>{
                return user.firstName+"-"+user.lastName
            })

            const fullName2 = computed({
               get(){
                    return user.firstName+"-"+user.lastName
               },
               set(val:string){
                   let names = val.split('-')
                   user.firstName = names[0]
                   user.lastName = names[1]
               }
            })

            const fullName3 = ref('')
            //监视---监视指定的数据
            watch(user,({firstName,lastName})=>{
                fullName3.value = firstName+"-"+lastName
            },{immediate:true})
            //immediate默认会执行一次watch，deep深度监视

            //监视，不需要配置immediate，本身默认就会进行监视，（默认执行一次）
            /*watchEffect(()=>{
                fullName3.value = user.firstName+"-"+user.lastName
            })*/

            //监视fullName3的数据，改变firstName和lastName
            watchEffect(()=>{
                    let names = fullName3.value.split('-')
                   user.firstName = names[0]
                   user.lastName = names[1]
            })

            //watch---可以监视多个数据的
            watch([user.firstName,user.lastName,fullName3],()=>{
                console.log('1111')
            })

            //当我们使用watch监视非响应式的数据的时候，代码需要改一下
            watch([()=>user.firstName,()=>user.lastName,fullName3],()=>{
                console.log('2222')
            })

            return {
                ...toRefs(user),
                fullName1,
                fullName2,
                fullName3
            }
        }
    }

</script>

```

## 生命周期钩子函数

与2.x版本生命周期想对应的组合式API

- beforeCreate -> 使用 setup()
- created -> 使用setup()
- beforeMount -> onBeforeMount
- mounted -> onMounted
- beforeUpdate -> onBeforeUpdate
- updated -> onUpdated
- beforeDestroy -> onBeforeUnmount
- destroyed -> onUnmounted
- errorCaptured -> onErrorCaptured

新增的钩子函数

除了和2.x生命周期等效项之外，组合式API还提供了以下调试钩子函数

- onRenderTracked
- onRenderTriggered

两个钩子函数都接收一个DebuggerEvent，与watchEffect参数选项中的onTrack和onTrigger

## 自定义hook函数

- 使用vue3的组合API封装的可复用的功能函数
- 自定义hook的作用类似于vue2中的mixin技术
- 自定义hook的优势：很清楚复用功能代码的来源，更清楚易懂

useMousePosition.ts

```typescript
import { onBeforeUnmount, onMounted, ref } from 'vue'

export default function(){
    const x = ref(-1)
    const y = ref(-1)

    //点击事件的回调函数
    const clickHandler = (event:MouseEvent)=>{
        x.value = event.pageX
        y.value = event.pageY
    }

    // 页面已经加载完毕了，再进行点击的操作
    // 页面加载完毕的生命周期组合API
    onMounted(()=>{
        window.addEventListener('click',clickHandler)
    })

    onBeforeUnmount(()=>{
        window.removeEventListener('click',clickHandler)
    })
    return {
        x,
        y
    }
}
```

App.vue

```vue
<template>
    <h2>自定义hook函数</h2>
    <h3>x:{{ x }},y:{{ y }}</h3>
</template>

<script lang="ts">
   import useMousePostion from './hooks/useMousePosition'
    
    export default {
        name:'App',
        components: {
            
        },
        setup(){
            const {x,y} = useMousePostion()

            return {
                x,
                y,
            }

        }
            
    }

</script>

```

## toRefs

把一个响应式对象转换成普通对象，该普通对象的每个property都是一个ref

应用：当从合成函数返回响应式对象时，toRefs非常有用，这样的消费组件就可以在不丢失响应式的情况下对返回的对象进行分解使用

问题：reactive对象取出的所有属性值都是非响应式的

解决：利用toRefs可以将一个响应式reactive对象的所有原始属性转换为响应式的ref属性

## ref获取元素

利用ref函数获取组件中的标签元素

```vue
<template>
     <input type="text" ref="inputRef">
</template>

<script lang="ts">
    import { onMounted, ref } from 'vue'
    
    export default {
        name:'App',
        components: {
            
        },
        setup(){
          const inputRef = ref<HTMLElement|null>(null)

          onMounted(()=>{
              inputRef.value && inputRef.value.focus()//页面加载完成自动获取焦点
          })

         return {
             inputRef
         }
        }
            
    }

</script>

```

## shallowReactive与shallowRef

- shallowReactive ：只处理了对象内最外层属性的响应式（也就是浅响应式）
- shallowRef：只处理了value的响应式，不进行对象的reactive处理
- 什么时候浅响应式？
  - 一般情况下使用ref和reactive即可
  - 如果有一个对象数据，结构比较深，但变化时只是外层属性变化 ====> shallowReactive 
  - 如果有一个对象数据，后面会产生新的对象来替换 ====> shallowRef

## readonly与shallowReadonly

- readonly
  - 深度只读数据
  - 获取一个对象（响应式或纯对象）或ref并返回原始代理的只读代理
  - 只读代理是深层的：访问的任何嵌套property也是只读的
- shallowReadonly
  - 浅只读数据
  - 创建一个代理，使其自身的property为只读，但不执行嵌套对象的深度只读转换
- 应用场景
  - 在某种特定情况下，我们可能不希望对数据进行更新操作，那就可以包装生成一个只读代理对象来读取数据，而不能修改或删除

## toRaw与markRaw

- toRaw
  - 返回由reactive或readonly方法转换为响应式代理的普通对象
  - 这是一个还原方法，可用于临时读取，访问不会被代理/跟踪，写入时也不会触发界面更新
- markRaw
  - 标记一个对象，使其永远不会转换为代理，返回对象本身
  - 应用场景
    - 有些值不应被设置为响应式，例如复杂的第三方类实例或Vue组件对象
    - 当渲染具有不可变数据源的大列表时，跳过代理转换可以提高性能

## toRef

- 为源响应式对象上的某个属性创建一个ref对象，二者内部操作的是同一个数据值，更新时二者是同步的
- 区别ref：拷贝了一份新的数据值单独操作，更新时相互不影响
- 应用：当要将某个prop的ref传递给复合函数时，toRef很有用

```vue
<template>
    <h2>toRef的使用和特点</h2>  
    <h3>state:{{state}}</h3>
    <h3>age:{{age}}</h3>
    <h3>money:{{money}}</h3>
    <button @click="update">更新数据</button>
</template>

<script lang="ts">
   
    import {reactive, toRef, ref } from 'vue'

    export default {
        name:'App',
        components: {
            
        },
        setup(){
            const state = reactive({
                age:5,
                money:100
            })
            //把响应式数据state对象中某个属性age变成了ref对象
            const age = toRef(state,'age')
            //把响应式对象中的某个属性使用ref进行包装，变成了一个ref对象
            const money = ref(state.money)

            const update = ()=>{
               // age.value += 2
               state.money += 100
            }

            return {
                state,
                age,
                money,
                update
            }

        }
            
    }

</script>

```

## customRef

- 创建一个自定义的ref，并对其依赖项跟踪和更新触发进行显式控制

```
<script>
	//自定义hook防抖函数
	//value传入的数据，将来数据的类型不确定，所以用泛型，delay防抖的间隔时间，默认是200毫秒
	import {customRef} from 'vue'
	function useDebounceRef<T>(value: T ,delay = 200){
		let timeOutId:number
		return customRef((track,trigger) => {
			return {
				get(){
					//告诉vue追踪数据
					track()
					return value
				},
				set(newValue: T){
					clearTimeout(timeOutId)
					//开启定时器
				timeOutId =	setTimeout(()=>{
						value = newValue
						//告诉Vue更新页面
						trigger()
					},delay)
				}				
			}	
		})
	}

</script>
```

## provide与inject

- provide和inject提供依赖注入，功能类似2.x的provide/inject
- 实现跨层级组件（祖孙）间通信

GrandSon.vue

```vue
<template>
  <h2 :style="{color}">GrandSon组件</h2>
</template>

<script>
import { inject } from 'vue'
export default {
    name: 'GrandSon',
    setup(){
        const color = inject('color')
        return {
            color
        }
    }
}
</script>

<style>

</style>
```

Son.vue

```vue
<template>
  <h2>Son子级组件</h2>
  <grand-son></grand-son>
</template>

<script>

import GrandSon from './GrandSon.vue'

export default {
    name: 'Son',
    components:{
        GrandSon
    },
    setup(){

    }
}
</script>

<style>

</style>
```

App.vue

```vue
<template>
    <h2>provide与inject</h2>
    <p>当前颜色：{{color}}</p>
    <button @click="color='red'">红色</button>
    <button @click="color='yellow'">黄色</button>
    <button @click="color='green'">绿色</button>
    <son></son>
</template>

<script lang="ts">
   
    import {provide, ref } from 'vue'
    import Son from './components/Son.vue'

    export default {
        name:'App',
        components: {
            Son
        },
        setup(){
            const color = ref('red')

            provide('color',color)

            return {
               color
            }

        }
            
    }

</script>

```



## 响应式数据的判断

- isRef：检查一个值是否为ref对象
- isReactive：检查一个对象是否是由reactive创建的响应式代理
- isReadonly：检查一个对象是否是由readonly创建的只读代理
- isProxy：检查一个对象是否是由reactive或者readonly方法创建的代理

## Fragment

片段

- 在vue2中，组件必须有一个根标签
- 在vue3中，组件可以没有根标签，内部会将多个标签包含在一个Fragment虚拟元素中
- 好处，减少标签层级，减小内存占用

## Teleport

瞬移

- Teleport提供一种干净的方法，让组件的html在父组件界面外的特定标签（很可能是body）下插入显示

## Suspense

不确定的

- 它们允许我们的应用程序在等待异步组件时渲染一些后备内容，可以让我们创建一个平滑的用户体验
