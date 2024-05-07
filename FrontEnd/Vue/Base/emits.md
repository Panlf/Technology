# Vue中的emits

## 介绍
Vue应用程序架构中的核心概念之一是组件之间的父子关系。父组件经常需要与其子组件进行交互，反之亦然！我们利用这个概念来创建复杂且交互性强的用户界面。虽然props使得数据从父组件流向子组件，但是“emits”使得数据从子组件流向父组件。

基本上，“emits”是Vue中的一个概念，允许子组件与其父组件进行通信。在Vue中使用emits时，您可以向父组件发出带有数据（可选）的自定义事件。父组件可以监听事件并相应地处理自己的“响应”。这是一种强大的机制，可以促进子组件和父组件之间的无缝通信！

## 为什么 emits 有用
Emits 提供了一种结构化和解耦的方式，使组件能够与其父组件进行交互。这样可以创建更易于维护和扩展的应用程序。通过利用 emits，我们可以创建可重用的子组件，而不会将它们与其父组件紧密耦合在一起，从而可以在各种上下文中使用。

Emits 在实现子组件与父组件之间的高度解耦方面起着至关重要的作用。当子组件向父组件发射事件时，它们不会直接操作父组件的状态或调用父组件的方法。相反，发射器提供了一个抽象层，允许父组件决定如何处理这些事件。我认为，这种关注点的分离有助于实现更易于维护和可扩展的架构！

## 组件通信
子组件
```
<template>
 <button @click="sendMessageToParent">Send Message to Parent</button>
</template>

<script setup>
import { defineEmits } from 'vue';

const emit = defineEmits(['messageToParent']);

const sendMessageToParent = () => {
const message = 'Hello from child!';
emit('messageToParent', message);
}
</script>
```

父组件
```
<template>
  <div>
    <child-component @message-to-parent="handleMessageFromChild" />
    <p>Message from child: {{ messageFromChild }}</p>
  </div>
</template>

<script setup>
import ChildComponent from './ChildComponent.vue';
import { ref } from 'vue';

const messageFromChild = ref('');

const handleMessageFromChild = (message) => {
  messageFromChild.value = message;
}
</script>
```

## 如何在Typescript中正确地使用类型推断
使用emits的一个“缺点”是，当你发出一个自定义事件时，你不一定知道子组件会发出什么。这种不确定性可能会导致数据类型和运行时错误的潜在问题。幸运的是，Vue 3的Composition API与TypeScript结合提供了一个非常强大的解决方案来解决这个问题。通过正确地为emits添加类型，你可以确保类型安全性，提高代码清晰度，并使你的Vue应用程序更易于维护。

让我们探索如何使用Vue 3的Composition API和script setup正确地使用TypeScript来输入emits。

子组件（使用TypeScript）
```
<template>
  <button @click="sendDataToParent">Send Data to Parent</button>
</template>

<script setup lang="ts">
import { defineEmits } from 'vue';

// Define the interface for the emitted object 
// (Ideally a shared interface for both components)
// This could event get exported, fot the parent component can import it
interface ItemData {
  id: number;
  name: string;
  quantity: number;
}

const emit = defineEmits<{
  // Define the emitted event and its payload type
  (event: 'dataToParent', payload: ItemData[]): void;
}>();

function sendDataToParent() {
  const payload: ItemData[] = [
    { id: 1, name: 'Item 1', quantity: 3 },
    { id: 2, name: 'Item 2', quantity: 5 },
    { id: 3, name: 'Item 3', quantity: 2 },
  ];

  emit('dataToParent', payload);
}
</script>
```

父组件：（使用TypeScript）
```
<template>
  <div>
    <test-child @data-to-parent="handleDataFromChild" />
    <div v-if="itemsFromChild.length">
      <p v-for="item in itemsFromChild" :key="item.id">Item: {{ item.name }} | Quantity: {{ item.quantity }}</p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import TestChild from '@/TestChild.vue';

// Define the interface for the emitted object 
// (Ideally a shared interface for both components)
interface ItemData {
  id: number;
  name: string;
  quantity: number;
}

const itemsFromChild = ref<ItemData[]>([]);

const handleDataFromChild = (payload: ItemData[]) => {
  itemsFromChild.value = payload;
};
</script>

```

通过利用TypeScript的强大功能，我们可以确保我们的组件之间的通信精确无误且类型安全。