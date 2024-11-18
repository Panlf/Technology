# Go协程

## goroutine简介

- 协程goroutine在go语言中属于轻量级的线程
- 在运行时由runtime管理
- 每个go程序至少启动一个goroutine，main函数也在goroutine上运行

## 启动goroutine
- 关键字 go
- 模式
    - go变量
    - go匿名函数
    - go函数

## 需要注意的地方
- main的goroutine结束了就意味着整个程序的运行已经结束了，换言之，go的执行是非阻塞的，不会等待
- 不同goroutine之间的代码次序并不能代表真正的执行顺序
- go后边的函数的返回值会被忽略，也就是说协程不能有返回值
- 没有父子goroutine的概念，所有的goroutine是平等的被调度和执行的
- go程序在执行时会单独为main创建一个goroutine，遇到其他go关键字再去创建其他的goroutine
- go没有暴露goroutine id给用户，所以不能在一个goroutine里面显式的操作另外一个goroutine