# Vite基础

> [Vite官网](https://vitejs.cn/)

## 创建项目
```
> npm init vite@latest
> Project name vite-project
> Select a framework Vue
> Select a variant JavaScript
> cd vite-project
> npm install
> npm run dev
```
## 配置
vite.config.js
```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from "path"

// https://vitejs.dev/config/
export default defineConfig({
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "src"),
      "comps": path.resolve(__dirname, "src/components"),
      "apis": path.resolve(__dirname, "src/apis"),
      "views": path.resolve(__dirname, "src/views"),
      "utils": path.resolve(__dirname, "src/utils"),
      "routes": path.resolve(__dirname, "src/routes"),
      "styles": path.resolve(__dirname, "src/styles"),
    }
  },
  plugins: [vue()],
})
```
## 子父组件传值
```javascript
<template>
   <button @click="onclick">emit</button>
</template>
<script setup>
//1.组件导入
import { ref } from 'vue'

//2.定义属性
defineProps({
  msg: String,
})

//3.定义事件
const emit = defineEmits(['myClick'])
const onclick = () => {
   emit('myClick')
}
 

const count = ref(0)
</script>
```
```javascript

<template>
	<HelloWorld msg="Vite + Vue" @myClick="onMyClick" />
</template>
<script setup>
import HelloWorld from 'comps/HelloWorld.vue'
const onMyClick = ()=>{
  alert("alert hello world my click")
}
</script>
```
## vuex和vue-router
```javascript
npm install vue-router
npm install vuex@next
```
router/index.js
```javascript
import {createRouter,createWebHashHistory} from 'vue-router'

const router = createRouter({
    history: createWebHashHistory(),
    routes:[
        {path:'/',component:()=>import('views/Home.vue')}
    ]
})


export default router
```
store/index.js
```javascript
import { createStore } from 'vuex'

export default createStore({
    state:{
        counter:10
    },
    mutations:{
        add(state){
            state.counter++
        }
    }
})
```
views/Home.vue
```javascript
<template>
    <div>
        <hello-world msg="HomeHahahah"></hello-world>
    </div>
</template>

<script setup>
import HelloWorld from 'comps/HelloWorld.vue'


</script>

```
main.js
```javascript
import { createApp } from 'vue'
import './style.css'
import App from './App.vue'
import router from './router'
import store from './store'

createApp(App).use(router).use(store).mount('#app')

```
App.vue
```javascript

<template>
  <router-view />
</template>
```
HelloWorld.vue
```javascript
<template>
  <h1>{{ msg }}</h1>
  <p @click="$store.commit('add')">{{ $store.state.counter }}</p>
</template>
```

## 创建Vue3项目入门指南
创建命令
```powershell
npm init vite@latest

yarn create vite

pnpm create vite

cd [项目名]
pnpm install
pnpm run dev
```
```json

{
  "files.autoSave": "off",
  "git.autofetch": true,
  "files.associations": {
    "*.cjson": "jsonc",
    "*.wxss": "css",
    "*.wxs": "javascript",
    "*.wpy": "javascriptreact",
    "*.py": "python"
  },
  "emmet.includeLanguages": {
    "wxml": "html"
  },
  "minapp-vscode.disableAutoConfig": true,
  "git.confirmSync": false,
  "search.actionsPosition": "right",
  "search.exclude": {
    "**/dist": true
  },
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "editor.suggestSelection": "first",
  "files.exclude": {
    "**/node_modules": true,
    "*/node_modules": true
  },
  "sync.gist": "686f9b0e863088a613cdc96e5bc81c55",
  "[less]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[vue]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "beautify.language": {
    "js": {
      "type": ["javascript", "json", "jsonc"],
      "filename": [".jshintrc", ".jsbeautifyrc"]
    },
    "css": ["css", "less", "scss"],
    "html": ["htm", "html"]
  },
  "editor.tabSize": 2,
  "sync.autoUpload": true,
  "sync.forceUpload": false,
  "sync.forceDownload": false,
  "sync.autoDownload": true,
  "beautify.config": "",
  "prettier.trailingComma": "none",
  "prettier.arrowParens": "avoid",
  "editor.fontSize": 13,
  // "workbench.colorTheme": "Visual Studio Dark",
  "editor.accessibilitySupport": "on",
  "diffEditor.ignoreTrimWhitespace": false,
  "editor.quickSuggestions": {
    "strings": true
  },
  "editor.rulers": [],
  "[html]": {
    "editor.defaultFormatter": "vscode.html-language-features"
  },
  "extensions.closeExtensionDetailsOnViewChange": true,
  "[javascriptreact]": {
    "editor.defaultFormatter": "svipas.prettier-plus"
  },
  "[jsonc]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[json]": {
    "editor.defaultFormatter": "vscode.json-language-features"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "eslint.format.enable": true,
  "editor.formatOnSave": true,
  "prettier.singleQuote": false,
  "prettier.semi": true
}
```
```javascript

import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import path from "path";
...

// https://vitejs.dev/config/
export default defineConfig({
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "src")
    }
  },
  plugins: [vue()]
  ...
});
```
```json

{
  "compilerOptions": {
    "target": "es5",
    "module": "esnext",
    "baseUrl": "./",
    "jsx": "preserve",
    "moduleResolution": "node",
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "paths": {
      "@/*": ["./src/*"]
    },
    "lib": ["esnext", "dom", "dom.iterable", "scripthost"]
  },
  "exclude": ["node_modules"]
}
```
代理
```json

...

export default defineConfig({
  resolve: {
    ...
  },
  plugins: [
    ...
  ],
  base: "/", // 打包路径
  server: {
    host: "0.0.0.0",
    port: 3300, 
    open: true, 
    cors: true, 
    https: true
    proxy: {  //设置代理
      "/api": {
        target: "https://xxx.xxx.com",
        changeOrigin: true,
        rewrite: path => path.replace(/^\/api/, "")
      }
    }
  },
  ...
});
```
