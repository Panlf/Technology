# Vue3快速入门

使用vite体验Vue3

```
npm init vite-app <project-name>
cd <project-name>
npm install
npm run dev
```



## 新特性

### Composition API

Composition API为vue应用提供更好的逻辑复用和代码组织。

```vue
<template>
  <h1>{{ msg }}</h1>
  <button @click="count++">count is: {{ count }}</button>
  <p>Hello,Vite!!!</p>
  <p>{{ counter }}</p>
  <p>{{ doubleCounter }}</p>
  <p>{{ msg2 }}</p>
  <p ref="desc"></p>
</template>

<script>

import { computed, onUnmounted, reactive,onMounted,ref, toRefs, watch } from 'vue'

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  /*data() {
    return {
      count: 0
    }
  },*/
  setup(){
    // count、counter相关
    const {counter,count,doubleCounter} = useCounter()  


    //单值响应式
    const msg2 = ref('some message')
  

    // 展开处理
    //toRefs(data)

    //使用元素引用
    const desc = ref(null)

    //侦听器
    watch(counter,(val,oldVal)=>{
      const p = desc.value
      p.textContent =  `counter change from ${oldVal} to ${val}`
    })
    return {counter,count,doubleCounter,msg2,desc}
  }
}

function useCounter(){
  const data = reactive({
      counter:1,
      count:1,
      doubleCounter: computed(()=> data.count * 2)
    })

    let timer

    onMounted(()=>{
      timer = setInterval(
        () => {
          data.counter++
        },1000)
    }) 

    onUnmounted(()=>{
      clearInterval(timer)
    })

  return toRefs(data)
}

</script>

```



### Teleport

传送门组件提供一种简洁的方式可以指定它里面内容的父元素。

```vue
<template>
    <div>
        <button @click="modelOpen=true">弹出一个模态窗口</button>

        <teleport to="body">
            <div v-if="modelOpen" class="model">
                <div>
                    这是一个弹窗 我的父元素是body
                    <button @click="modelOpen=false">关闭</button>
                </div>
            </div>
        </teleport>
    </div>
</template>

<script>
export default {
    data(){
        return {
            modelOpen: true
        }
    },
    setup(){

    }
}
</script>

<style scoped>
.model {
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    background-color: rgb(0, 0, 0,0.5);
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
}

.model div {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background-color: white;
    width: 300px;
    height: 300px;
    padding: 5px;
}

</style>
```



### Fragments

vue3中组件可以拥有多个根

```vue
<template>
	<header>...</header>
	<main v-bind="$attrs">...</main>
	<footer>...</footer>
</template>
```



### Emits Component Option

vue3中组件发送的自定义事件需要定义在emits选项中：

- 原生事件会触发两次，比如`click`
- 更好的指示组件工作方式
- 对象形式事件校验

```vue
<template>
	<div @click="$emit('click')">
        <h3>自定义事件</h3>
    </div>
</template>

<script>
export default {
    emits: ['click']
}
</script>
```



### `createRenderer` API用于创建自定义渲染器

vue3中支持自定义渲染器（Renderer）：这个API可以用来自定义渲染逻辑。

## 破坏性变化

### Global API改为应用程序实例调用

vue2中有很多全局api可以改变vue的行为，比如`Vue.component`等。这导致一些问题：

- vue2没有app概念，new Vue()得到的根实例被作为app，这样的话所有创建的根实例是共享相同的全局配置，这在测试时会污染其他测试用例，导致测试变得苦难
- 全局配置也导致没有办法在单页面创建不同全局配置的多个app实例

vue3中使用createApp返回app实例，由它暴露一系列全局api

```vue
import { createApp,h } from 'vue'
import App from './App.vue'

const app = createApp(App)
	.component('comp',{render: () => h('div','i am comp')})
	.mount('#app')
```

### Global and internal APIs重构为可做摇树优化

vue2中不少global-api是作为静态函数直接挂在构造函数上的，例如`Vue.nextTick()`，如果我们从未在代码中用过它们，就会形成所谓的`dead code`，这类global-api造成的`dead code`无法使用webpack的tree-shaking排除掉。

```vue
import Vue from 'vue'

Vue.nextTick(()=>{
	//something something DOM-related
})
```

vue3中做了相应的变化，将它们抽取成为独立函数，这样打包工具的摇树优化可以将这些`dead code`排除掉

```
import {nextTick} from 'vue'

nextTick(()=>{
	//something something DOM-related
})
```

 ### model选项和v-bind的sync修饰符被移除，统一为v-model参数形式

vue2中的.sync和v-model功能有重叠，容易混肴，vue3做了统一

```
<div id="app">
	<h3>{{ data }}</h3>
	<comp v-model="data"></comp>
	<!-- 等效于 -->
	<comp :modelValue="data" @update:modelValue="data=$event "></comp>
</div>


app.component('comp',{
template:`<div @click="$emit('update:modelValue','new value')">
	i am comp,{{modelValue}}
</div>
`,
props:['modelValue']
})
```

### 渲染函数API修改

渲染函数变得更简单好用了，修改主要以下几点：

- 不再传入h函数，需要手动导入；拍平的props结构；scopedSlots删掉了，统一到slots


### 函数式组件仅能通过简单函数方式创建，functional选项废弃

函数式组件变化较大，主要有以下几点

- 性能提升在vue3中可忽略不计，所以vue3中推荐使用状态组件
- 函数时组件仅能通过纯函数形式声明。接收`props`和`context`两个参数
- SFC中`<template>`不能添加`functional`特性声明函数是组件
- `{ functional: true}`组件选项移除

### 异步组件要求使用`defineAsyncComponent`方法创建

由于vue3中函数式组件必须定义为纯函数，异步组件定义时有如下变化：

- 必须明确使用`defineAsyncComponent`包裹
- `component`选项重命名为`loader`
- Loader函数不再接收`resolve` and `reject` 且必须返回一个Promise

定义一个异步组件

```
import { defineAsyncComponent } from 'vue'

//不带配置的异步组件
const asyncPage = defineAsyncComponent(() => import('./NextPage.vue'))
```

带配置的异步组件，loader选项是以前的component

```
import ErrorComponent from './components/ErrorComponent.vue'
import LoadingComponent from './components/LoadingComponent.vue'

//带配置的异步组件
const asyncPageWithOptions = defineAsyncComponent({
	loader: () => import('./NextPage.vue'),
	delay: 200,
	timout: 3000,
	errorComponent: ErrorComponent,
	loadingComponent: LoadingComponent
})
```

### 组件data选项应该总是声明为函数

vue3中data选项统一为函数形式，返回响应式数据

```
createApp({
	data(){
		return {
			apiKey : 'a1b2c3'
		}
	}
}).mount('#app')
```

### 自定义组件白名单

Vue3中自定义元素检测发生在模板编译时，如果要添加一些vue之外的自定义元素，需要在编译器选项中设置`isCustomElement`选项

使用构建工具时，模板都会用vue-loader预编译，设置它提供的compilerOptions即可：

```
rules:[
	{
		test:/\.vue$/,
		use: 'vue-loader',
		options:{
			compilerOptions:{
				isCustomElement: tag => tag === 'plastic-button'
			}
		}
	}
]
```

我们是用vite，在vite.config.js中配置`vueCompilerOptions`即可

```
module.exports = {
	vueCompilerOptions: {
		isCustomElement: tag => tag === 'piechart'
	}
}
```

### is属性仅限于用在component标签上

vue3中设置动态组件时，is属性仅能用于component标签上。

```
<component is="comp"></component>
```

dom内模板解析使用`v-is`代替

```
<table>
	<tr v-is="'blog-post-row'"></tr>
</table>
```

### $scopedSlots属性被移除，都用$slots代替

vue3中统一普通插槽和作用域插槽到$slots，具体变化如下：

- 插槽均以函数形式暴露
- $scopedSlots移除

### 自定义指令API和组件保持一致

vue3中指令api和组件保持一致，具体表现在

- bind  ->
- inserted ->
- beforeUpdate: new!
- update -> removed!和updated基本相同，因此被移除之，使用updated代替。
- componentUpdated -> updated
- beforeUnmount new!和组件生命周期钩子相似，元素将要被移除之前条用
- unbind -> unmounted

### transition类名变更

- v-enter -> v-enter-from
- v-leave -> v-leave-from

### 组件watch选项和实例方法$watch不再支持点分隔符字符串路径

以`.`分割的表达式不再被watch和$watch支持，可以使用计算函数作为$watch参数实现。

```
this.$watch(() => this.foo.bar,(v1,v2) => {
	console.log(this.foo.bar)
})
```

### Vue2中应用程序根容器的outerHTML会被根组件的模板替换（或被编译为template），Vue3.x现在使用根容器的innerHTML取代

## 移除

### keyCode作为v-on修饰符被移除

vue2中可以使用keyCode指代某个按键，vue3不再支持

```
<!-- keyCode方式不再支持-->
<input v-on:keyup.13="submit" />

<!-- 只能使用alias方式-->
<input v-on:keyup.enter="submit" />
```

### $on,$off and $once移除

上述3个方法被认为不应该由vue提供，因此被移除了，可以使用其他三方库实现。

```
<script src="https://unpkg.com/mitt/dist/mitt.umd.js"></script>
```

```
//创建emitter
const emitter = mitt()

//发送事件
emitter.emit('foo','foooooooooooo')

//监听事件
emitter.on('foo', msg => console.log(msg))
```

### Filters移除

vue3中移除了过滤器，请调用方法或者计算属性代替。

## 路由

### 实例创建方式

- history选项替代了mode选项 
  - history: createWebHistory()
  - hash:  createWebHashHistory()
  - abstract: createMemoryHistory()
- base选项移至createWebHistory等方法中
- 通配符*被移除
- isReady() 替代 onReady()

```
router.push()
// before
router.onReady(onSuccess,onError)
// now
router.isReady().then(onSuccess).catch(onError)
```

- scrollBehavior变化   x,y  变成  top,left
- 现在keep-alive和transition必须用在router-view内部

```
<!-- before -->
<keep-alive>
	<router-view></router-view>
</keep-alive>

<!-- after -->
<router-view v-slot="{ Component }">
	<keep-alive>
		<component :is="Component" />
	</keep-alive>
</router-view>
```

- router-link移除了一票属性

  - append

    ```
    <router-link to="child-route" append>
    
    <router-link :to="append($route.path,'child-route')">
    
    
    app.config.globalProperties.append=(path,pathToAppend) =>{
    	return path+pathToAppend
    }
    ```

  - tag/event

    ```
    <router-link to="/xx" tag="span" evnet="dblclick"></router-link>
    
    <router-link to="/xx" custom v-slot="{navigate}">
    	<span @dblclick="navigate">xxx</span>
    </router-link>
    ```

  - exact: 现在完全匹配逻辑简化了

- minins中的路由守卫将被忽略

- match方法被移除，使用resolve替代

- 移除router.getMatchedComponents()

  ```
  router.currentRoute.value.matched
  ```

- 包括首屏导航在内所有导航均为异步

  ```
  app.use(router)
  router.isReady().then(() => app.mount('#app'))
  ```

  > 如果首屏存在路由守卫，则可以不等待就绪直接挂载，产生结果将和vue2相同

- route的parent属性被移除

  ```
  const parent = this.$route.matched[this.$route.matched.length - 2]
  ```

- pathToRegexpOptions选项被移除

  - pathToRegexpOptions => strict
  - caseSensitive => sensitive

  ```
  createRouter({
  	strict: boolean,
  	sensitive: boolean
  })
  ```

- 使用history.state

  ```
  //之前
  history.pushState(myState,'',url)
  
  //现在
  router.push(url)
  history.replaceState({...history.state,...myState},'')
  ```

  ```
  //之前
  history.replaceState({},'',url)
  
  //现在
  history.replaceState(history.state,'',url)
  ```

  

- routes选项是必填项

  ```
  createRouter({routes:[]})
  ```

- 跳转不存在命名路由报错

  ```
  router.push({name:'dashboad'})
  ```

- 缺少必填参数会抛出异常

- 命名子路由如果path为空的时候不再追加/

  ```
  [
  	path:'/dashboard',
  	children:[
  		{path:'',component: DashboardDefault}
  	]
  ]
  ```

  以前生成url： /dashboard/

  副作用：给设置了重定向redirect选项的子路由带来副作用

  ```
  [
  	path:'/dashboard',
  	children:[
  		{path:'',redirect:'home'},
  		{path:'home',component:Home}
  	]
  ]
  ```

  /dashboard/home

  /home

- $route属性编码行为

  - params/query/hash
  - Path/fullpath不再做解码
  - hash会被解码
  - push、resolve和replace，字符串参数，或者对象参数path属性必须编码
  - params / 会被解码
  - query中+不处理，stringifyQuery

## Vite

Vite是开发构建工具，开发期它利用浏览器native ES Module特性导入组织代码，生产中利用rollup作为打包工具，它有如下特点：

- 光速启动
- 热模块替换
- 按需编译

### 安装

```
npm init vite-app <project-name>
cd <project-name>
npm install
npm run dev
```

### CSS文件导入

vite中可以直接导入`.css`文件，样式将影响导入的页面，最终会被打包到`style.css`

```
import { createApp } from 'vue'
import App from './App.vue'
import './index.css'
```

### CSS Module

SFC使用CSS Module

```
<style module>
```

### CSS预处理器

安装对应的预处理器就可以直接在vite项目中使用

```
<style lang="scss">
</style>
```

或者在JS中导入

```
import './style.scss'
```

### PostCSS

Vite自动对*.vue文件和导入.css文件应用PostCSS配合，我们只需要安装必要的插件和添加`postcss.config.js`文件即可。

```
module.exports = {
	plugins:[
		require('autoprefixer')
	]	
}
```

### 引用静态资源

我们可以在*.vue文件的template，style和纯.css文件中以相对和绝对路径方式引用静态资源。

```
<!-- 相对路径 -->
<img src="./assets/logo.png">
<!-- 绝对路径 -->
<img src="/src/assets/logo.png">

<style scoped>
	#app {
		background-image: url('./assets/logo.png')
	}
</style>
```

public目录下可以存放未在源码中引用的资源，它们会被留下且文件名不会有哈希处理。这些文件会被原封不动拷贝到发布目录的根目录下

```
<img src="/logo.png">
```

### typescript整合

Vite可直接导入.ts文件，在SFC中通过<script lang="ts">使用

案例

```
<script lang="ts">
	import { defineComponent } from 'vue'
	
	interface Course {
		id: number,
		name: string
	}
	
	export default defineComponent({
		setup(){
			const state = ref<Course[]>([])
			setTimeout(()=>{
				state.value.push({id:1,name:'vue3'})
			},1000)
			
			return {state}
		}
	})
</script>
```

