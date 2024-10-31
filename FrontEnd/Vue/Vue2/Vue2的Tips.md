# Vue2的Tips

## Vue语法

### 事件绑定

#### 指令v-on

Vue使用v-on指令监听DOM事件，开发者可以将事件代码通过v-on指令绑定到DOM节点上。Vue也为v-on提供了一种简写形式@
```
<button v-on:click="console.log('A Vue App')">打印消息</button> 

<button @click="console.log('A Vue App')">打印消息</button> 
```

实际上，写法的不同只是为了避免混肴和冲突，事件绑定的主要作用在于降低学习和开发的成本
- 解决了不同浏览器之间的兼容问题
- 提供了语法糖-事件绑定修饰符

### 常见修饰符

|名称|可用版本|可用事件|说明|
|--|--|--|--|
|.stop|所有|任意|当事件触发时，阻止事件冒泡|
|.prevent|所有|任意|当事件触发时，阻止元素默认行为|
|.capture|所有|任意|当事件触发时，阻止事件捕获|
|.self|所有|任意|限制事件仅作用于节点自身|
|.once|2.1.4以上|任意|事件被触发一次后即解除监听|
|.passive|2.3.0以上|滚动|移动端，限制事件永不调用preventDefault()方法|


```
<form @submit.prevent="handleSubmit">
    <button type="submit">提交</button>
</form>
```
点击提交后，页面没有被重载。

### 按键修饰符

|别名修饰符|键值修饰符|对应按键|
|--|--|--|
|.delete|.8/.46|回格/删除|
|.tab|.9|制表|
|.enter|.13|回车|
|.esc|.27|退出|
|.space|.32|空格|
|.left|.37|左|
|.up|.38|上|
|.right|.39|右|
|.down|.40|下|

```
<input type="text" @keyup.13="console.log($event)">


<input type="text" @keyup.enter="console.log($event)">
```

