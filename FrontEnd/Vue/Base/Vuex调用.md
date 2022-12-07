# Vuex

## 如何使用全局state

直接使用`this.$store.state.xxx`

map辅助

```
computed:{
	...mapState(['xxx']),
	...mapState({'新名字':'xxx'})
}
```

## 如何使用modules中的state

直接使用 `this.$store.state.模块名.xxx;`

map辅助

```
computed:{
	...mapState('模块名',['xxx']),
	...mapState('模块名',{'新名字':'xxx'})
}
```

## 如何使用全局getters

直接使用`this.$store.getters.xxx`

map辅助

```
computed:{
	...mapGetters(['xxx']),
	...mapGetters({'新名字':'xxx'})
}
```

## 如何使用modules中的getters

直接使用 `this.$store.getters.模块名.xxx;`

map辅助

```
computed:{
	...mapGetters('模块名',['xxx']),
	...mapGetters('模块名',{'新名字':'xxx'})
}
```

## 如何使用全局mutations

直接使用`this.$store.commit('mutation名',参数)`

map辅助

```
methods:{
	...mapMutations(['mutation名']),
	...mapMutations({'新名字':'mutation名'})
}
```

## 如何使用modules中的mutations(namespace:true)

直接使用 `this.$store.commit('模块名/mutation名',参数)`

map辅助

```
methods:{
	...mapMutations('模块名',['mutation名']),
	...mapMutations('模块名',{'新名字':'mutation名'})
}
```

## 如何使用全局actions

直接使用`this.$store.dispatch('action名',参数)`

map辅助

```
methods:{
	...mapMutations(['action名']),
	...mapMutations({'新名字':'action名'})
}
```

## 如何使用modules中的actions(namespace:true)

直接使用 `this.$store.dispatch('模块名/action名',参数)`

map辅助

```
methods:{
	...mapMutations('模块名',['action名']),
	...mapMutations('模块名',{'新名字':'action名'})
}
```