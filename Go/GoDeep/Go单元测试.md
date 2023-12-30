# Go单元测试

`golang`提供了标准库`testing`用来支持测试
```
import "testing"
```
## 支持测试种类
| 类型 | 格式 | 作用 |
| --- | --- | --- |
| 单元测试 | 函数名前缀为Test | 测试程序的一些逻辑行为是否准确 |
| 基准（压力）测试 | 函数名前缀为Benchmark | 测试函数的性能 |
| 示例测试 | 函数名前缀为Example | 为文档提供示例文档 |
| 模糊（随机）测试 | 函数名前缀为Fuzz | 生成一个随机测试用例去覆盖人为测不到的各种复杂场景 |

## 测试格式
文件名一般`xxx_test.go`结尾。

- 测试用例名称一般命名为`Test`加上待测试的方法名
- 测试用的参数有且只有一个，在这里是`t *testing.T`
- 基准测试（benchmark）的参数是`*testing.B`，TestMain的参数是`*testing.M`类型

运行`go test`，该`package`下所有的测试用例都会被执行。
`go test -v`显示每个用例的测试结果，另外`-cover`参数可以查看覆盖率。
如果只想运行其中的一个用例，例如`TestAdd`，可以用`-run`参数指定，该参数支持通配符`*`，和部分正则表达式，例如`^`、`$`。
单元测试框架提供的日志方法

| 方法 | 备注 |
| --- | --- |
| Log | 打印日志，同时结束测试 |
| Logf | 格式化打印日志，同时结束测试 |
| Error | 打印错误日志，同时结束测试 |
| Errorf | 格式化打印错误日志，同时结束测试 |
| Fatal | 打印致命日志，同时结束测试 |
| Fatalf | 格式化打印致命日志，同时结束测试 |

## 子测试
子测试是Go语言内置支持的，可以在某个测试用例中，根据测试环境使用`t.run`创建不同的子测试用例
```go
func TestMul(t *testing.T){
    t.Run("pos",func(t *testing.T){
        if Mul(2,3) != 6 {
            t.Fatal("fail")    
        }
    })
}
```
## 帮助函数
对一些重复的逻辑，抽取出来作为公共的帮助函数（helpers），可以增加测试代码的可读性和可维护性。借助帮助函数，可以让测试用例的主逻辑看起来更清晰。

## setup和teardown
如果在同一个测试文件中，每一个测试用例运行前后的逻辑是相同的，一般会在`setup`和`teardown`函数中。例如执行前需要实例化待测试的对象，如果这个对象比较复杂，很适合将这一部分逻辑提取出来；执行后，可能会做一些资源回收类的工作，例如关闭网络连接，释放文件等。
```go
func setup(){
    fmt.Println("start")
}

func teardown(){
    fmt.Println("over")
}
```
## 网络测试
### TCP/HTTP
假设需要测试某个API接口的handler能够正常工作，例如`helloHandler`
```go
func helloHandler(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("hello world"))
}

```
那我们可以创建真实的网络连接进行测试
```go
// test code
import (
	"io/ioutil"
	"net"
	"net/http"
	"testing"
)

func handleError(t *testing.T, err error) {
	t.Helper()
	if err != nil {
		t.Fatal("failed", err)
	}
}

func TestConn(t *testing.T) {
	ln, err := net.Listen("tcp", "127.0.0.1:0")
	handleError(t, err)
	defer ln.Close()

	http.HandleFunc("/hello", helloHandler)
	go http.Serve(ln, nil)

	resp, err := http.Get("http://" + ln.Addr().String() + "/hello")
	handleError(t, err)

	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	handleError(t, err)

	if string(body) != "hello world" {
		t.Fatal("expected hello world, but got", string(body))
	}
}

```

- `net.Listen("tcp","127.0.0.1:0")`：监听一个未被占用的端口，并返回Listener
- 调用`http.Serve(ln,nil)`启动http服务
- 使用`http.GET`发起一个Get请求，检查返回值是否正确
- 尽量不对`http`和`net`库使用`mock`，这样可以覆盖较为真实的场景
### httptest
针对http开发的场景，使用标准库`net/http/httptest`进行测试更为高效
```go
// test code
import (
	"io/ioutil"
	"net/http"
	"net/http/httptest"
	"testing"
)

func TestConn(t *testing.T) {
	req := httptest.NewRequest("GET", "http://example.com/foo", nil)
	w := httptest.NewRecorder()
	helloHandler(w, req)
	bytes, _ := ioutil.ReadAll(w.Result().Body)

	if string(bytes) != "hello world" {
		t.Fatal("expected hello world, but got", string(bytes))
	}
}

```
使用`httptest`模拟请求对象（req）和响应对象（w），达到相同的目的。
