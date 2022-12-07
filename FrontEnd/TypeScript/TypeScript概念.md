# TypeScript概念

## TypeScript是什么

- 以JavaScript为基础构建的语言
- 一个JavaScript的超集
- 可以在任何支持JavaScript的平台中执行
- TypeScript扩展了JavaScript，并添加了类型
- TS不能被JS解析器直接执行

## TypeScript增加了什么

- 类型
- 支持ES的新特性
- 添加ES不具备的新特性
- 丰富的配置选项

## TypeScript的开发环境

- 安装Node.js
- 使用npm全局安装TypeScript
  - npm i -g typescript
- 检查是否安装成功 tsc -V
- 创建一个ts文件
- 使用tsc对ts文件进行编译
  - 执行命令 tsc xxx.ts

## 运行ts文件

为了直接运行ts文件，安装npm install -g ts-node

这样可以直接运行`ts-node xxx.ts`

## 内置对象

JavaScript有很多内置对象，它们可以直接在TypeScript中当作定义好了的类型。

1、ECMAScript的内置对象

- Boolean
- Number
- String
- Date
- RegExp
- Error

2、BOM和DOM的内置对象

- Window
- Document
- HTMLElement
- DocumentFragment
- Event
- NodeList

