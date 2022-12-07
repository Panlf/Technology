## Vuex


### 组件之间共享数据的方式

- 父向子传值：v-bind属性绑定
- 子向父传值：v-on 事件绑定
- 兄弟组件之间共享数据 EventBus
  - $on 接收数据的那个组件
  - $emit 发送数据的那个组件



### Vuex是什么

Vuex是实现组件共享状态（数据）管理的一种机制，可以方便的实现组件之间数据的共享。

什么样的数据适合存储到Vuex中

一般情况下，只有组件之间共享的数据，才有必要存储到vuex中；对于组件中的私有数据，依旧存储在组件自身的data中即可。



### Vuex核心概念

- State
- Mutation
- Action
- Getter

#### State

State提供唯一的公共数据源，所有共享的数据都要统一放到Store的State中进行存储。

```vue
//创建store数据源，提供唯一公共数据
const store = new Vuex.Store({
	state : {count:0}
})
```

组件访问State中数据的第一种方式

```vue
this.$store.state.全局数据名称
```

组件访问State中数据的第二种方式

```vue
//1、从vuex中按需导入mapState函数
import {mapState} from 'vuex'
```

通过刚才导入的mapState函数，将当前组件需要的全局数据，映射为当前组件的computed计算属性

```vue
//2、将全局数据，映射为当前组件的计算属性
computed：{
	...mapState(['count'])
}
```

#### Mutation

Mutation用于变更Store中的数据

- 只能通过mutation变更Store数据，不可以直接操作Store中的数据
- 通过这种方式虽然操作起来稍微繁琐一些，但是可以集中监控所有数据的变化

```vue
//定义Mutation
const store = new Vuex.Store({
	state:{
		count:0
	},
	mutations:{
		add(state) {
			//变更状态
			state.count++
		}
	}
})
```



```vue
methods:{
	handle1(){
		//触发mutations的第一种方式
		this.$store.commit('add')
	}
}
```

可以在触发mutations时传递参数

```vue
//定义Mutation
const store = new Vuex.Store({
	state:{
		count:0
	},
	mutations:{
		addN(state,step) {
			//变更状态
			state.count += step
		}
	}
})
```

```vue
methods:{
	handle2(){
		//触发mutations时携带参数
		//在调用commit参数
		this.$store.commit('addN',3 )
	}
}
```

触发mutations的第二种方式

```vue
//1、从vuex中按需导入mapMutations函数
import { mapMutations } from 'vuex'
```

通过刚才导入的mapMutations函数，将需要的mutations函数，映射为当前组件的methods方法：

```vue
//2、将指定的mutations函数，映射为当前组件的methods函数
methods:{
	...mapMutations(['addCount'],['addN'])
}
```

<u>**mutations里不能写异步的代码**</u>

#### Action

Action用于处理异步任务

如果通过异步操作变更数据，必须通过Action，而不能使用Mutation，但是在Action中还是要通过触发Mutation的方式间接变更数据。

```vue
const store = new Vuex.Store({
	//...省略其他代码
	mutations:{
		add(state){
			state.count++
		}
	},
	actions:{
		addAsync(context){
			setTimeout(()=>{
				context.commit('add')
			},1000)
		}
	}
})
```

```
//触发Action
methods:{
	handle(){
		this.$store.dispatch('addAsync')
	}
}
```

触发actions异步任务时携带参数

```vue
//定义Action
const store = new Vuex.Store({
	//...省略其他代码
	mutations:{
		addN(state,step){
			state.count += step
		}
	},
	actions:{
		addNAsync(context,step){
			setTimeout(()=>{
				context.commit('addN',step)
			},1000)
		}
	}
})
```



```
//触发Action
methods:{
	handle(){
		//在调用dispatch函数
		//触发actions时携带参数
		this.$store.dispatch('addNAsync',5)
	}
}
```

触发actions的第二种方式

```vue
//1、从vuex中按需导入mapActions
import { mapActions } from 'vuex'
```

通过刚才导入的mapActions函数，将需要的actions函数，映射为当前组件的methods：

```vue
//2、将指定的actions函数，映射为当前组件的methods函数
methods:{
	...mapActions(['addAsync','addNAsync'])
}
```

调用

```vue
//1、直接调用
<button @click="addAsync">+1_Street</button>

//2、申明函数
<button @click="handle1">+1</button>

methods:{

	...mapActions(['addAsync']),
	handle1(){
		this.addAsync()
	}
}

```

#### Getter

Getter用于对Store中的数据进行加工处理形成新的数据

- Getter可以对Store中已有的数据加工处理之后形成新的数据，类似Vue 的计算属性
- Store中数据发生变化，Getter的数据也会跟着变化。

```vue
//定义Getter
const store = new Vuex.Store({
	state:{
		count:0
	},
	getters:{
		showNum: state =>{
			return '当前最新的数量是【'+state.count+'】'
		}
	}
})
```

使用getters的第一种方式

```
this.$store.getters.名称
```

```vue
<h3>{{ this.$store.getters.showNum }}</h3>
```

使用getters的第二种方式

```vue
import { mapGetters } from 'vuex'

computed:{
	...mapGetters(['showNum'])
}
```

```vue
<h3>{{showNum}}</h3>
```

