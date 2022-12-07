# Vuex4使用

vuex 4 是vue3的兼容版本，关注于兼容性，提供和vuex3相同API，因此我们可以在vue3中复用之间已存在vuex代码

## 安装vuex 4

```
npm i vuex@next
```

## 初始化方式

为了向Vue 3初始化方式看齐，Vuex 4 初始化方式做了相应变化：使用新的createStore函数创建新的store实例。 

```
import { createStore } from 'vuex'

const store = createStore({
    state(){
        return {
            count: 1
        }
    },
    mutations:{
        add(state){
            state.count++
        }
    }
})

```

```
<template>
   <!-- 传统写法 -->
   <p @click="$store.commit('add')">{{$store.state.count}}</p>
  
   <!-- composition写法 -->
   <p @click="add">{{ count}}</p>

</template>

<script>
import { toRefs } from 'vue'
import { useStore } from 'vuex'
export default {
  name: 'App',
  components: {
    
  },
  setup(){
    const store = useStore()
    return {
      ...toRefs(store.state),
      add(){
        store.commit('add')
      }
    }

  }
}
</script>

```

