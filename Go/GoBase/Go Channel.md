# Channel

## 基础用法

**构造**
```go
# 无缓冲
ch := make(chan int)


# 有缓冲
ch := make(chan int,10)
```

**读**
```go
val := <- ch

<- ch

val,ok := <- ch
```

**写**
```go

var data int
ch <- data
```

**关闭**
```go
close(ch)
```