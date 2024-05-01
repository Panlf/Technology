# Vue3 新语法

## Vue3 里 script 的三种写法

### 最基本的 Vue2 写法

```vue
<template>
    <div>{{ count }}</div>
    <button @click="onClick">
        增加 1
    </button>
</template>
<script>
export default {
    data() {
        return {
            count: 1,
        };
    },
    methods: {
        onClick() {
            this.count += 1;
        },
    },
}
</script>
```

### setup() 属性

```vue
<template>
    <div>{{ count }}</div>
    <button @click="onClick">
        增加 1
    </button>
</template>
<script>
import { ref } from 'vue';
export default {
    // 注意这部分
    setup() {
        let count = ref(1);
        const onClick = () => {
            count.value += 1;
        };
        return {
            count,
            onClick,
        };
    },
}
</script>
```

### `<script setup>`
```vue
<template>
    <div>{{ count }}</div>
    <button @click="onClick">
        增加 1
    </button>
</template>
<script setup>
import { ref } from 'vue';
const count = ref(1);
const onClick = () => {
    count.value += 1;
};
</script>
```

## 如何使用 `<script setup>` 编写组件

### data - 唯一需要注意的地方

以前在 `data` 中创建的属性，现在全都用 `ref()` 声明。

在 `template` 中直接用，在 `script` 中记得加 `.value` 。

**写法对比**
```vue
// Vue2 的写法
<template>
    <div>{{ count }}</div>
    <button @click="onClick">
        增加 1
    </button>
</template>
<script>
export default {
    data() {
        return {
            count: 1,
        };
    },
    methods: {
        onClick() {
            this.count += 1;
        },
    },
}
</script>
```

```vue
// Vue3 的写法
<template>
    <div>{{ count }}</div>
    <button @click="onClick">
        增加 1
    </button>
</template>
<script setup>
import { ref } from 'vue';
// 用这种方式声明
const count = ref(1);
const onClick = () => {
    // 使用的时候记得 .value
    count.value += 1;
};
</script>
```

#### 注意事项 — 组合式 api 的心智负担

a、`ref`和 `reactive`

`Vue3` 里，还提供了一个叫做 `reactive` 的 `api`。

但是我的建议是，你不需要关心它。绝大多数场景下，`ref` 都够用了。

b、什么时候用 `ref()` 包裹，什么时候不用。

要不要用`ref`，就看你的这个变量的值改变了以后，页面要不要跟着变。

当然，你可以完全不需要关心这一点，跟过去写 `data` 一样就行。

只不过这样做，你在使用的时候，需要一直 `.value`。

c、不要解构使用

在使用时，不要像下面这样去写，会丢失响应性。

也就是会出现更新了值，但是页面没有更新的情况

```vue
// Vue3 的写法
<template>
 <div>{{ count }}</div>
 <button @click="onClick">
 增加 1
 </button>
</template>
<script setup>
import { ref } from 'vue';
const count = ref(1);
const onClick = () => {
 // 不要这样写！！
 const { value } = count;
 value += 1;
};
</script>
```

### methods
声明事件方法，我们只需要在 `script` 标签里，创建一个方法对象即可。

剩下的在 `Vue2` 里是怎么写的，`Vue3` 是同样的写法。

```vue
// Vue2 的写法
<template>
    <div @click="onClick">
        这是一个div
    </div>
</template>
<script>
export default {
    methods: {
        onClick() {
            console.log('clicked')
        },
    },
}
</script>
```

```vue
// Vue3 的写法
<template>
    <div @click="onClick">
        这是一个div
    </div>
</template>
<script setup>
// 注意这部分
const onClick = () => {
    console.log('clicked')
}
</script>
```

### props

声明 `props` 我们可以用 `defineProps()`

```vue
// Vue2 的写法
<template>
    <div>{{ foo }}</div>
</template>
<script>
export default {
    props: {
        foo: String,
    },
    created() {
        console.log(this.foo);
    },
}
</script>
```

```vue
// Vue3 的写法
<template>
 <div>{{ foo }}</div>
</template>
<script setup>
// 注意这里
const props = defineProps({
 foo: String
})
// 在 script 标签里使用
console.log(props.foo)
</script>
```

#### 注意事项 — 组合式 api 的心智负担

使用 props 时，同样注意不要使用解构的方式。
```vue
<script setup>
const props = defineProps({
 foo: String
})
 // 不要这样写
const { foo } = props;
console.log(foo)
</script>
```

### emits 事件
与 `props` 相同，声明 `emits` 我们可以用 `defineEmits()`

```vue
// Vue2 的写法
<template>
    <div @click="onClick">
        这是一个div
    </div>
</template>
<script>
export default {
    emits: ['click'], // 注意这里
    methods: {
        onClick() {
            this.$emit('click'); // 注意这里
        },
    },

}
</script>
```

```vue
// Vue3 的写法
<template>
 <div @click="onClick">
 这是一个div
 </div>
</template>
<script setup>
// 注意这里
const emit = defineEmits(['click']);
const onClick = () => {
 emit('click') // 注意这里
}
</script>
```

### computed
```vue
// Vue2 的写法
<template>
    <div>
        <span>{{ value }}</span>
        <span>{{ reversedValue }}</span>
    </div>
</template>
<script>
export default {
    data() {
        return {
            value: 'this is a value',
        };
    },
    computed: {
        reversedValue() {
            return value
                .split('').reverse().join('');
        },
    },
}
</script>
```

```vue
// Vue3 的写法
<template>
 <div>
 <span>{{ value }}</span>
 <span>{{ reversedValue }}</span>
 </div>
</template>
<script setup>
import {ref, computed} from 'vue'
const value = ref('this is a value')
// 注意这里
const reversedValue = computed(() => {
 // 使用 ref 需要 .value
 return value.value
 .split('').reverse().join('');
})
</script>
```

### watch
这一部分，我们需要注意一下了，`Vue3` 中，`watch` 有两种写法。一种是直接使用 `watch`，还有一种是使用 `watchEffect`。

两种写法的区别是：

- `watch` 需要你明确指定依赖的变量，才能做到监听效果。
- `watchEffect` 会根据你使用的变量，自动的实现监听效果。

#### 直接使用 watch
```vue
// Vue2 的写法
<template>
    <div>{{ count }}</div>
    <div>{{ anotherCount }}</div>
    <button @click="onClick">
        增加 1
    </button>
</template>
<script>
export default {
    data() {
        return {
            count: 1,
            anotherCount: 0,
        };
    },
    methods: {
        onClick() {
            this.count += 1;
        },
    },
    watch: {
        count(newValue) {
            this.anotherCount = newValue - 1;
        },
    },
}
</script>
```

```vue
// Vue3 的写法
<template>
 <div>{{ count }}</div>
 <div>{{ anotherCount }}</div>
 <button @click="onClick">
 增加 1
 </button>
</template>
<script setup>
import { ref, watch } from 'vue';
const count = ref(1);
const onClick = () => {
 count.value += 1;
};
const anotherCount = ref(0);
// 注意这里
// 需要在这里，
// 明确指定依赖的是 count 这个变量
watch(count, (newValue) => {
 anotherCount.value = newValue - 1;
})
</script>
```

#### 使用 watchEffect
```vue
// Vue2 的写法
<template>
    <div>{{ count }}</div>
    <div>{{ anotherCount }}</div>
    <button @click="onClick">
        增加 1
    </button>
</template>
<script>
export default {
    data() {
        return {
            count: 1,
            anotherCount: 0,
        };
    },
    methods: {
        onClick() {
            this.count += 1;
        },
    },
    watch: {
        count(newValue) {
            this.anotherCount = newValue - 1;
        },
    },
}
</script>
```

```vue
// Vue3 的写法
<template>
 <div>{{ count }}</div>
 <div>{{ anotherCount }}</div>
 <button @click="onClick">
 增加 1
 </button>
</template>
<script setup>
import { ref, watchEffect } from 'vue';
const count = ref(1);
const onClick = () => {
 count.value += 1;
};
const anotherCount = ref(0);
// 注意这里
watchEffect(() => {
 // 会自动根据 count.value 的变化，
 // 触发下面的操作
 anotherCount.value = count.value - 1;
})
</script>
```

### 生命周期
`Vue3` 里，除了将两个 `destroy` 相关的钩子，改成了 `unmount`，剩下的需要注意的，就是在` <script setup>` 中，不能使用 `beforeCreate` 和 `created` 两个钩子。

```vue
// 选项式 api 写法
<template>
 <div></div>
</template>
<script>
export default {
 beforeCreate() {},
 created() {},
  
 beforeMount() {},
 mounted() {},
  
 beforeUpdate() {},
 updated() {},
  
 // Vue2 里叫 beforeDestroy
 beforeUnmount() {},
 // Vue2 里叫 destroyed
 unmounted() {},
  
 // 其他钩子不常用，所以不列了。
}
</script>
```

```vue
// 组合式 api 写法
<template>
 <div></div>
</template>
<script setup>
  import {
   onBeforeMount,
   onMounted,
   onBeforeUpdate,
   onUpdated,
   onBeforeUnmount,
   onUnmounted,
  } from 'vue'
  onBeforeMount(() => {})
  onMounted(() => {})
  onBeforeUpdate(() => {})
  onUpdated(() => {})
  onBeforeUnmount(() => {})
  onUnmounted(() => {})
</script>
```