# Pinia的使用

与 Vuex 相比，Pinia 提供了更简单的 API，更少的规范，以及 Composition-API 风格的 API 。更重要的是，与 TypeScript 一起使用具有可靠的类型推断支持。

## Pinia 与 Vuex 3.x/4.x 的不同

- mutations 不复存在。只有 state 、getters 、actions。
- actions 中支持同步和异步方法修改 state 状态。
- 与 TypeScript 一起使用具有可靠的类型推断支持。
- 不再有模块嵌套，只有 Store 的概念，Store 之间可以相互调用。
- 支持插件扩展，可以非常方便实现本地存储等功能。
- 更加轻量，压缩后体积只有 2kb 左右。

## 基本用法

安装
```
npm install pinia
```

在 main.js 中 引入 Pinia// src/main.js
```
import { createPinia } from 'pinia'

const pinia = createPinia()
app.use(pinia)
```

定义一个 Store

在 src/stores 目录下创建 counter.js 文件，使用 defineStore() 定义一个 Store 。defineStore() 第一个参数是 storeId ，第二个参数是一个选项对象：

```
// src/stores/counter.js
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  state: () => ({ count: 0 }),
  getters: {
    doubleCount: (state) => state.count * 2
  },
  actions: {
    increment() {
      this.count++
    }
  }
})
```

我们也可以使用更高级的方法，第二个参数传入一个函数来定义 Store ：
```
// src/stores/counter.js
import { ref, computed } from 'vue'
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', () => {
  const count = ref(0)
  const doubleCount = computed(() => count.value * 2)
  function increment() {
    count.value++
  }

  return { count, doubleCount, increment }
})
```

在组件中使用

在组件中导入刚才定义的函数，并执行一下这个函数，就可以获取到 store 了：
```
<script setup>
import { useCounterStore } from '@/stores/counter'

const counterStore = useCounterStore()
// 以下三种方式都会被 devtools 跟踪
counterStore.count++
counterStore.$patch({ count: counterStore.count + 1 })
counterStore.increment()
</script>

<template>
  <div>{{ counterStore.count }}</div>
  <div>{{ counterStore.doubleCount }}</div>
</template>
```

## State

### 解构 store

tore 是一个用 reactive 包裹的对象，如果直接解构会失去响应性。我们可以使用 storeToRefs() 对其进行解构：

```
<script setup>
import { storeToRefs } from 'pinia'
import { useCounterStore } from '@/stores/counter'

const counterStore = useCounterStore()
const { count, doubleCount } = storeToRefs(counterStore)
</script>

<template>
  <div>{{ count }}</div>
  <div>{{ doubleCount }}</div>
</template>
```

### 修改 store

除了可以直接用 store.count++ 来修改 store，我们还可以调用 $patch 方法进行修改。$patch 性能更高，并且可以同时修改多个状态。
```
<script setup>
import { useCounterStore } from '@/stores/counter'

const counterStore = useCounterStore()
counterStore.$patch({
  count: counterStore.count + 1,
  name: 'Abalam',
})
</script>
```

但是，这种方法修改集合（比如从数组中添加、删除、插入元素）都需要创建一个新的集合，代价太高。因此，$patch 方法也接受一个函数来批量修改：
```
cartStore.$patch((state) => {
  state.items.push({ name: 'shoes', quantity: 1 })
  state.hasChanged = true
})
```

### 监听 store

我们可以通过 $subscribe() 方法可以监听 store 状态的变化，类似于 Vuex 的 subscribe 方法。与 watch() 相比，使用 $subscribe() 的优点是，store 多个状态发生变化之后，回调函数只会执行一次。

```
<script setup>
import { useCounterStore } from '@/stores/counter'

const counterStore = useCounterStore()
counterStore.$subscribe((mutation, state) => {
  // 每当状态发生变化时，将 state 持久化到本地存储
  localStorage.setItem('counter', JSON.stringify(state))
})
</script>
```

也可以监听 pinia 实例上所有 store 的变化
```
// src/main.js
import { watch } from 'vue'
import { createPinia } from 'pinia'

const pinia = createPinia()
watch(
  pinia.state,
  (state) => {
    // 每当状态发生变化时，将所有 state 持久化到本地存储
    localStorage.setItem('piniaState', JSON.stringify(state))
  },
  { deep: true }
)
```

## Getters

### 访问 store 实例

大多数情况下，getter 只会依赖 state 状态。但有时候，它会使用到其他的 getter ，这时候我们可以通过 this 访问到当前 store 实例。

```
// src/stores/counter.js
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  state: () => ({ count: 0 }),
  getters: {
    doubleCount(state) {
      return state.count * 2
    },
    doublePlusOne() {
      return this.doubleCount + 1
    }
  }
})
```

### 访问其他 Store 的 getter

要使用其他 Store 的 getter，可以直接在 getter 内部使用：

```
// src/stores/counter.js
import { defineStore } from 'pinia'
import { useOtherStore } from './otherStore'

export const useCounterStore = defineStore('counter', {
  state: () => ({
    count: 1
  }),
  getters: {
    composeGetter(state) {
      const otherStore = useOtherStore()
      return state.count + otherStore.count
    }
  }
})
```

### 将参数传递给 getter

getter 本质上是一个 computed ，无法向它传递任何参数。但是，我们可以让它返回一个函数以接受参数：

```
// src/stores/user.js
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  state: () => ({
    users: [{ id: 1, name: 'Tom'}, {id: 2, name: 'Jack'}]
  }),
  getters: {
    getUserById: (state) => {
      return (userId) => state.users.find((user) => user.id === userId)
    }
  }
})
```

在组件中使用：

```
<script setup>
import { storeToRefs } from 'pinia'
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()
const { getUserById } = storeToRefs(userStore)
</script>

<template>
  <p>User: {{ getUserById(2) }}</p>
</template>
```

注意：如果这样使用，getter 不会缓存，它只会当作一个普通函数使用。一般不推荐这种用法，因为在组件中定义一个函数，可以实现同样的功能。

## Actions

### 访问 store 实例

与 getters 一样，actions 可以通过 this 访问当 store 的实例。不同的是，actions 可以是异步的。

```
// src/stores/user.js
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  state: () => ({ userData: null }),
  actions: {
    async registerUser(login, password) {
      try {
        this.userData = await api.post({ login, password })
      } catch (error) {
        return error
      }
    }
  }
})
```

### 访问其他 Store 的 action

要使用其他 Store 的 action，也可以直接在 action 内部使用：

```
// src/stores/setting.js
import { defineStore } from 'pinia'
import { useAuthStore } from './authStore'

export const useSettingStore = defineStore('setting', {
  state: () => ({ preferences: null }),
  actions: {
    async fetchUserPreferences(preferences) {
      const authStore = useAuthStore()
      if (authStore.isAuthenticated()) {
        this.preferences = await fetchPreferences()
      } else {
        throw new Error('User must be authenticated!')
      }
    }
  }
})
```

以上就是 Pinia 的详细用法，是不是比 Vuex 简单多了。除此之外，插件也是 Pinia 的一个亮点，个人觉得非常实用，下面我们就来重点介绍一下。

## Plugins

由于是底层 API，Pania Store 完全支持扩展。以下是可以扩展的功能列表： 

- 向 Store 添加新状态 
- 定义 Store 时添加新选项 
- 为 Store 添加新方法 
- 包装现有方法 
- 更改甚至取消操作 
- 实现本地存储等副作用 
- 仅适用于特定 Store

### 使用方法

Pinia 插件是一个函数，接受一个可选参数 context ，context 包含四个属性：app 实例、pinia 实例、当前 store 和选项对象。函数也可以返回一个对象，对象的属性和方法会分别添加到 state 和 actions 中。

```
export function myPiniaPlugin(context) {
  context.app // 使用 createApp() 创建的 app 实例（仅限 Vue 3）
  context.pinia // 使用 createPinia() 创建的 pinia
  context.store // 插件正在扩展的 store
  context.options // 传入 defineStore() 的选项对象（第二个参数）
  // ...
  return {
    hello: 'world', // 为 state 添加一个 hello 状态
    changeHello() { // 为 actions 添加一个 changeHello 方法
      this.hello = 'pinia'
    }
  }
}
```

然后使用 pinia.use() 将此函数传递给 pinia 就可以了：

```
// src/main.js
import { createPinia } from 'pinia'

const pinia = createPinia()
pinia.use(myPiniaPlugin)
```

### 向 Store 添加新状态

可以简单地通过返回一个对象来为每个 store 添加状态：

```
pinia.use(() => ({ hello: 'world' }))
```

也可以直接在 store 上设置属性来添加状态，为了使它可以在 devtools 中使用，还需要对 store.$state 进行设置：

```
import { ref, toRef } from 'vue'

pinia.use(({ store }) => {
  const hello = ref('word')
  store.$state.hello = hello
  store.hello = toRef(store.$state, 'hello')
})
```

也可以在 use 方法外面定义一个状态，共享全局的 ref 或 computed 
```
import { ref } from 'vue'

const globalSecret = ref('secret')
pinia.use(({ store }) => {
  // `secret` 在所有 store 之间共享
  store.$state.secret = globalSecret
  store.secret = globalSecret
}
```

### 定义 Store 时添加新选项

可以在定义 store 时添加新的选项，以便在插件中使用它们。例如，可以添加一个 debounce 选项，允许对所有操作进行去抖动：

```
// src/stores/search.js
import { defineStore } from 'pinia'

export const useSearchStore = defineStore('search', {
  actions: {
    searchContacts() {
      // ...
    },
    searchContent() {
      // ...
    }
  },

  debounce: {
    // 操作 searchContacts 防抖 300ms
    searchContacts: 300,
    // 操作 searchContent 防抖 500ms
    searchContent: 500
  }
})
```

然后使用插件读取该选项，包装并替换原始操作：

```
// src/main.js
import { createPinia } from 'pinia'
import { debounce } from 'lodash'

const pinia = createPinia()
pinia.use(({ options, store }) => {
  if (options.debounce) {
    // 我们正在用新的 action 覆盖原有的 action
    return Object.keys(options.debounce).reduce((debouncedActions, action) => {
      debouncedActions[action] = debounce(
        store[action],
        options.debounce[action]
      )
      return debouncedActions
    }, {})
  }
})
```

这样在组件中使用 actions 的方法就可以去抖动了，是不是很方便！

## 实现本地存储

相信大家使用 Vuex 都有这样的困惑，F5 刷新一下数据全没了。在我们眼里这很正常，但在测试同学眼里这就是一个 bug 。Vuex 中实现本地存储比较麻烦，需要把状态一个一个存储到本地，取数据时也要进行处理。而使用 Pinia ，一个插件就可以搞定。

这次我们就不自己写了，直接安装开源插件。

```
npm i pinia-plugin-persist
```

然后引入插件，并将此插件传递给 pinia ：

```
// src/main.js
import { createPinia } from 'pinia'
import piniaPluginPersist from 'pinia-plugin-persist'

const pinia = createPinia()
pinia.use(piniaPluginPersist)

```

接着在定义 store 时开启 persist 即可：

```
// src/stores/counter.js
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  state: () => ({ count: 1 }),
  // 开启数据缓存
  persist: {
    enabled: true
  }
})
```

这样，无论你用什么姿势刷新，数据都不会丢失啦！

默认情况下，会以 storeId 作为 key 值，把 state 中的所有状态存储在 sessionStorage 中。我们也可以通过 strategies 进行修改：

```
// 开启数据缓存
persist: {
  enabled: true，
  strategies: [
    {
      key: 'myCounter', // 存储的 key 值，默认为 storeId
      storage: localStorage, // 存储的位置，默认为 sessionStorage
      paths: ['name', 'age'], // 需要存储的 state 状态，默认存储所有的状态
    }
  ]
}
```