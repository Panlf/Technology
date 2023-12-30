# Go的Context
`Go` 在`1.7`引入了`context`包，目的是为了在不同的 `goroutine` 之间或跨 `API` 边界传递超时、取消信号和其他请求范围内的值（与该请求相关的值。这些值可能包括用户身份信息、请求处理日志、跟踪信息等等）。
在 `Go` 的日常开发中，`Context` 上下文对象无处不在，无论是处理网络请求、数据库操作还是调用 `RPC` 等场景下，都会使用到 `Context`。
## Context 接口
`context` 包在提供了一个用于跨 `API` 边界传递超时、取消信号和其他请求范围值的通用数据结构。它定义了一个名为 `Context` 的接口，该接口包含一些方法，用于在多个 `Goroutine` 和函数之间传递请求范围内的信息。
以下是 `Context` 接口的定义：
```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)

    Done() <-chan struct{}

    Err() error

    Value(key any) any
}
```
## Context的核心方法
`Context` 接口中有四个核心方法：

- `Deadline()`
- `Done()`
- `Err()`
- `Value()`
### Deadline()
`Deadline()` (`deadline time.Time, ok bool`) 方法返回 `Context` 的截止时间，表示在这个时间点之后，`Context` 会被自动取消。如果 `Context` 没有设置截止时间，该方法返回一个零值 `time.Time` 和一个布尔值 `false`。
```go
deadline, ok := ctx.Deadline()
if ok {
    // Context 有截止时间
} else {
    // Context 没有截止时间
}
```
### Done()
`Done()`方法返回一个只读通道，当`Context`被取消时，该通道会被关闭。你可以通过监听这个通道来检测`Context` 是否被取消。如果`Context`永不取消，则返回`nil`。
```go
select {
case <-ctx.Done():
    // Context 已取消
default:
    // Context 尚未取消
}
```
### Err()
`Err() `方法返回一个 `error` 值，表示 `Context` 被取消时产生的错误。如果 `Context` 尚未取消，该方法返回 `nil`。
```go
if err := ctx.Err(); err != nil {
    // Context 已取消，处理错误
}
```
### Value()
`Value(key any) any `方法返回与 `Context` 关联的键值对，一般用于在 `Goroutine` 之间传递请求范围内的信息。如果没有关联的值，则返回`nil`。
```go
value := ctx.Value(key)
if value != nil {
    // 存在关联的值
}
```
## Context的创建方式
### context.Background()
`context.Background()` 函数返回一个非 `nil` 的空 `Context`，它没有携带任何的值，也没有取消和超时信号。通常作为根 `Context` 使用。
```go
ctx := context.Background()
```
### context.TODO()
`context.TODO()` 函数返回一个非 `nil` 的空 `Context`，它没有携带任何的值，也没有取消和超时信号。虽然它的返回结果和 `context.Background()` 函数一样，但是它们的使用场景是不一样的，如果不确定使用哪个上下文时，可以使用 `context.TODO()`。
```go
ctx := context.TODO()
```
### context.WithValue()
`context.WithValue(parent Context, key, val any) `函数接收一个父 `Context` 和一个键值对 `key`、`val`，返回一个新的子 `Context`，并在其中添加一个 `key-value` 数据对。
```go
ctx := context.WithValue(parentCtx, "username", "陈明勇")
```
### context.WithCancel()
`context.WithCancel(parent Context) (ctx Context, cancel CancelFunc) `函数接收一个父 `Context`，返回一个新的子 `Context` 和一个取消函数，当取消函数被调用时，子 `Context` 会被取消，同时会向子 `Context` 关联的 `Done() `通道发送取消信号，届时其衍生的子孙 `Context` 都会被取消。这个函数适用于手动取消操作的场景。
```go
ctx, cancelFunc := context.WithCancel(parentCtx)  
defer cancelFunc()
```
### context.WithCancelCause() 
`context.WithCancelCause(parent Context) (ctx Context, cancel CancelCauseFunc) `函数是`Go 1.20`版本才新增的，其功能类似于`context.WithCancel()`，但是它可以设置额外的取消原因，也就是`error`信息，返回的`cancel` 函数被调用时，需传入一个`error`参数。
```go
ctx, cancelFunc := context.WithCancelCause(parentCtx)
defer cancelFunc(errors.New("原因"))
```
### context.Cause()
`context.Cause(c Context) error`函数用于返回取消`Context`的原因，即错误值`error`。如果是通过`context.WithCancelCause()`函数返回的取消函数`cancelFunc(myErr)`进行的取消操作，我们可以获取到 myErr 的值。否则，我们将得到与`c.Err()`相同的返回值。如果`Context`尚未被取消，将返回`nil`。
```go
err := context.Cause(ctx)
```
### context.WithDeadline()
`context.WithDeadline(parent Context, d time.Time) (Context, CancelFunc)` 函数接收一个父 `Context` 和一个截止时间作为参数，返回一个新的子 `Context`。当截止时间到达时，子 `Context` 其衍生的子孙 `Context` 会被自动取消。这个函数适用于需要在特定时间点取消操作的场景。
```go
deadline := time.Now().Add(time.Second * 2)
ctx, cancelFunc := context.WithTimeout(parentCtx, deadline)
defer cancelFunc()
```
### context.WithTimeout()
`context.WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)` 函数和`context.WithDeadline()`函数的功能是一样的，其底层会调用`WithDeadline()`函数，只不过其第二个参数接收的是一个超时时间，而不是截止时间。这个函数适用于需要在一段时间后取消操作的场景。
```
ctx, cancelFunc := context.WithTimeout(parentCtx, time.Second * 2)
defer cancelFunc()
```
## Context的使用场景
### 传递共享数据
编写中间件函数，用于向 `HTTP` 处理链中添加处理请求 `ID` 的功能。
```
type key int

const (
   requestIDKey key = iota
)

func WithRequestId(next http.Handler) http.Handler {
   return http.HandlerFunc(func(rw http.ResponseWriter, req *http.Request) {
      // 从请求中提取请求ID和用户信息
      requestID := req.Header.Get("X-Request-ID")

      // 创建子 context，并添加一个请求 Id 的信息
      ctx := context.WithValue(req.Context(), requestIDKey, requestID)

      // 创建一个新的请求，设置新 ctx
      req = req.WithContext(ctx)

      // 将带有请求 ID 的上下文传递给下一个处理器
      next.ServeHTTP(rw, req)
   })
}
```
首先，我们从请求的头部中提取请求 `ID`。然后使用 `context.WithValue `创建一个子上下文，并将请求 `ID` 作为键值对存储在子上下文中。接着，我们创建一个新的请求对象，并将子上下文设置为新请求的上下文。最后，我们将带有请求 `ID` 的上下文传递给下一个处理器。这样，通过使用 `WithRequestId` 中间件函数，我们可以在处理请求的过程中方便地获取和使用请求 `ID`，例如在 日志记录、跟踪和调试等方面。
### 传递取消信号，结束任务
启动一个工作协程，接收到取消信号就停止工作。
```
package main

import (
   "context"
   "fmt"
   "time"
)

func main() {
   ctx, cancelFunc := context.WithCancel(context.Background())
   go Working(ctx)

   time.Sleep(3 * time.Second)
   cancelFunc()

   // 等待一段时间，以确保工作协程接收到取消信号并退出
   time.Sleep(1 * time.Second)
}

func Working(ctx context.Context) {
   for {
      select {
      case <-ctx.Done():
         fmt.Println("下班啦...")
         return
      default:
         fmt.Println("陈明勇正在工作中...")
      }
   }
}
```
执行结果
```
······
······
陈明勇正在工作中...
陈明勇正在工作中...
陈明勇正在工作中...
陈明勇正在工作中...
陈明勇正在工作中...
下班啦...
```
在上面的示例中，我们创建了一个 `Working` 函数，它会不断执行工作任务。我们使用 `context.WithCancel` 创建了一个上下文 `ctx` 和一个取消函数 `cancelFunc`。然后，启动了一个工作协程，并将上下文传递给它。
在主函数中，需要等待一段时间（3 秒）模拟业务逻辑的执行。然后，调用取消函数 `cancelFunc`，通知工作协程停止工作。工作协程在每次循环中都会检查上下文的状态，一旦接收到取消信号，就会退出循环。
最后，等待一段时间（1 秒），以确保工作协程接收到取消信号并退出。
### 超时控制
模拟耗时操作，超时控制。
```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    // 使用 WithTimeout 创建一个带有超时的上下文对象
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()

    // 在另一个 goroutine 中执行耗时操作
    go func() {
        // 模拟一个耗时的操作，例如数据库查询
        time.Sleep(5 * time.Second)
        cancel()
    }()

    select {
        case <-ctx.Done():
        fmt.Println("操作已超时")
        case <-time.After(10 * time.Second):
        fmt.Println("操作完成")
    }
}
```
执行结果
```go
操作已超时
```
在上面的例子中，首先使用 `context.WithTimeout()` 创建了一个带有 3 秒超时的上下文对象 `ctx`, `cancel := context.WithTimeout(ctx, 3*time.Second)`。接下来，在一个新的 `goroutine `中执行一个模拟的耗时操作，例如等待 5 秒钟。当耗时操作完成后，调用 `cancel() `方法来取消超时上下文。
最后，在主 `goroutine` 中使用 `select` 语句等待超时上下文的完成信号。如果在 3 秒内耗时操作完成，那么会输出 "操作完成"。如果超过了 3 秒仍未完成，超时上下文的 `Done()` 通道会被关闭，输出 "操作已超时"。
## 使用Context的一些规则
使用 `Context` 上下文，应该遵循以下规则，以保持包之间的接口一致，并使静态分析工具能够检查上下文传播

- 不要在结构类型中加入 `Context `参数，而是将它显式地传递给需要它的每个函数，并且它应该是第一个参数，通常命名为 `ctx`
- 即使函数允许，也不要传递 `nil Context`。如果不确定要使用哪个 `Context`，建议使用 `context.TODO()`。
- 仅将 `Context` 的值用于传输进程和 `api` 的请求作用域数据，不能用于向函数传递可选参数。
