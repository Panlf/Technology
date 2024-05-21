# HttpClient优化

## 优化思路
- 池化
- 长连接
- httpclient和httpget复用
- 合理的配置参数（最大并发请求数，各种超时时间，重试次数）
- 异步

## 分析
### httpclient反复创建开销

httpclient是一个线程安全的类，没有必要由每个线程在每次使用时创建，全局保留一个即可。

### 反复创建tcp连接的开销
tcp的三次握手与四次挥手两大裹脚布过程，对于高频次的请求来说，消耗实在太大。试想如果每次请求我们需要花费5ms用于协商过程，那么对于qps为100的单系统，1秒钟我们就要花500ms用于握手和挥手，所以改成keep alive方式以实现连接复用！

### 重复缓存entity的开销

原本的逻辑里，使用了如下代码：
```
HttpEntity entity = httpResponse.getEntity();

String response = EntityUtils.toString(entity);
```
这里我们相当于额外复制了一份content到一个字符串里，而原本的httpResponse仍然保留了一份content，需要被consume掉，在高并发且content非常大的情况下，会消耗大量内存。并且，我们需要显式的关闭连接。

## 实现
### 定义一个keep alive strategy

在本业务场景里，我们相当于有少数固定客户端，长时间极高频次的访问服务器，启用keep-alive非常合适。

```
ConnectionKeepAliveStrategy myStrategy = new ConnectionKeepAliveStrategy() {
    @Override
    public long getKeepAliveDuration(HttpResponse response, HttpContext context) {
        HeaderElementIterator it = new BasicHeaderElementIterator
            (response.headerIterator(HTTP.CONN_KEEP_ALIVE));
        while (it.hasNext()) {
            HeaderElement he = it.nextElement();
            String param = he.getName();
            String value = he.getValue();
            if (value != null && param.equalsIgnoreCase
               ("timeout")) {
                return Long.parseLong(value) * 1000;
            }
        }
        return 60 * 1000;//如果没有约定，则默认定义时长为60s
    }
};
```

### 配置一个PoolingHttpClientConnectionManager

```
PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager();
connectionManager.setMaxTotal(500);
connectionManager.setDefaultMaxPerRoute(50);//例如默认每路由最高50并发，具体依据业务来定
```

也可以针对每个路由设置并发数。

### 生成httpclient
```
httpClient = HttpClients.custom()
     .setConnectionManager(connectionManager)
     .setKeepAliveStrategy(kaStrategy)
     .setDefaultRequestConfig(RequestConfig.custom().setStaleConnectionCheckEnabled(true).build())
     .build();
```

注意：使用setStaleConnectionCheckEnabled方法来逐出已被关闭的链接不被推荐。更好的方式是手动启用一个线程，定时运行closeExpiredConnections 和closeIdleConnections方法，如下所示。

```
public static class IdleConnectionMonitorThread extends Thread {
    
    private final HttpClientConnectionManager connMgr;
    private volatile boolean shutdown;
    
    public IdleConnectionMonitorThread(HttpClientConnectionManager connMgr) {
        super();
        this.connMgr = connMgr;
    }
 
    @Override
    public void run() {
        try {
            while (!shutdown) {
                synchronized (this) {
                    wait(5000);
                    // Close expired connections
                    connMgr.closeExpiredConnections();
                    // Optionally, close connections
                    // that have been idle longer than 30 sec
                    connMgr.closeIdleConnections(30, TimeUnit.SECONDS);
                }
            }
        } catch (InterruptedException ex) {
            // terminate
        }
    }
    
    public void shutdown() {
        shutdown = true;
        synchronized (this) {
            notifyAll();
        }
    }
    
}
```

### 使用httpclient执行method时降低开销
一种可行的获取内容的方式类似于，把entity里的东西复制一份：
```
res = EntityUtils.toString(response.getEntity(),"UTF-8");
EntityUtils.consume(response1.getEntity());
```

但是，更推荐的方式是定义一个ResponseHandler，方便你我他，不再自己catch异常和关闭流。在此我们可以看一下相关的源码：
```
public <T> T execute(final HttpHost target, final HttpRequest request,
        final ResponseHandler<? extends T> responseHandler, final HttpContext context)
        throws IOException, ClientProtocolException {
    Args.notNull(responseHandler, "Response handler");

    final HttpResponse response = execute(target, request, context);

    final T result;
    try {
        result = responseHandler.handleResponse(response);
    } catch (final Exception t) {
        final HttpEntity entity = response.getEntity();
        try {
            EntityUtils.consume(entity);
        } catch (final Exception t2) {
            // Log this exception. The original exception is more
            // important and will be thrown to the caller.
            this.log.warn("Error consuming content after an exception.", t2);
        }
        if (t instanceof RuntimeException) {
            throw (RuntimeException) t;
        }
        if (t instanceof IOException) {
            throw (IOException) t;
        }
        throw new UndeclaredThrowableException(t);
    }

    // Handling the response was successful. Ensure that the content has
    // been fully consumed.
    final HttpEntity entity = response.getEntity();
    EntityUtils.consume(entity);//看这里看这里
    return result;
}
```

可以看到，如果我们使用resultHandler执行execute方法，会最终自动调用consume方法，而这个consume方法如下所示：
```
public static void consume(final HttpEntity entity) throws IOException {
    if (entity == null) {
        return;
    }
    if (entity.isStreaming()) {
        final InputStream instream = entity.getContent();
        if (instream != null) {
            instream.close();
        }
    }
}
```

###  httpclient的一些超时配置
CONNECTION_TIMEOUT是连接超时时间，SO_TIMEOUT是socket超时时间，这两者是不同的。连接超时时间是发起请求前的等待时间；socket超时时间是等待数据的超时时间。

```
HttpParams params = new BasicHttpParams();
//设置连接超时时间
Integer CONNECTION_TIMEOUT = 2 * 1000; //设置请求超时2秒钟 根据业务调整
Integer SO_TIMEOUT = 2 * 1000; //设置等待数据超时时间2秒钟 根据业务调整
 
//定义了当从ClientConnectionManager中检索ManagedClientConnection实例时使用的毫秒级的超时时间
//这个参数期望得到一个java.lang.Long类型的值。如果这个参数没有被设置，默认等于CONNECTION_TIMEOUT，因此一定要设置。
Long CONN_MANAGER_TIMEOUT = 500L; //在httpclient4.2.3中我记得它被改成了一个对象导致直接用long会报错，后来又改回来了
 
params.setIntParameter(CoreConnectionPNames.CONNECTION_TIMEOUT, CONNECTION_TIMEOUT);
params.setIntParameter(CoreConnectionPNames.SO_TIMEOUT, SO_TIMEOUT);
params.setLongParameter(ClientPNames.CONN_MANAGER_TIMEOUT, CONN_MANAGER_TIMEOUT);
//在提交请求之前 测试连接是否可用
params.setBooleanParameter(CoreConnectionPNames.STALE_CONNECTION_CHECK, true);
 
//另外设置http client的重试次数，默认是3次；当前是禁用掉（如果项目量不到，这个默认即可）
httpClient.setHttpRequestRetryHandler(new DefaultHttpRequestRetryHandler(0, false));
```

### 如果配置了nginx的话，nginx也要设置面向两端的keep-alive

nginx默认和client端打开长连接而和server端使用短链接。

## 整体代码

引入Jar
```
<!-- httpclient -->
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.6</version>
</dependency>
```
具体代码
```
//Basic认证
private static final CredentialsProvider credsProvider = new BasicCredentialsProvider();
//httpClient
private static final CloseableHttpClient httpclient;
//httpGet方法
private static final HttpGet httpget;
//
private static final RequestConfig reqestConfig;
//响应处理器
private static final ResponseHandler<String> responseHandler;
//jackson解析工具
private static final ObjectMapper mapper = new ObjectMapper();

static {
    System.setProperty("http.maxConnections","50");
    System.setProperty("http.keepAlive", "true");
    //设置basic校验
    credsProvider.setCredentials(
            new AuthScope(AuthScope.ANY_HOST, AuthScope.ANY_PORT, AuthScope.ANY_REALM),
            new UsernamePasswordCredentials("", ""));
    //创建http客户端
    httpclient = HttpClients.custom()
            .useSystemProperties()
            .setRetryHandler(new DefaultHttpRequestRetryHandler(3,true))
            .setDefaultCredentialsProvider(credsProvider)
            .build();
    //初始化httpGet
    httpget = new HttpGet();
    //初始化HTTP请求配置
    reqestConfig = RequestConfig.custom()
            .setContentCompressionEnabled(true)
            .setSocketTimeout(100)
            .setAuthenticationEnabled(true)
            .setConnectionRequestTimeout(100)
            .setConnectTimeout(100).build();
    httpget.setConfig(reqestConfig);
    //初始化response解析器
    responseHandler = new BasicResponseHandler();
}
/*
 * 功能：返回响应
 * @author zhangdaquan
 * @param [url]
 * @return org.apache.http.client.methods.CloseableHttpResponse
 * @exception
 */
public static String getResponse(String url) throws IOException {
    HttpGet get = new HttpGet(url);
    String response = httpclient.execute(get,responseHandler);
    return response;
}
 
/*
 * 功能：发送http请求，并用net.sf.json工具解析
 * @author zhangdaquan
 * @param [url]
 * @return org.json.JSONObject
 * @exception
 */
public static JSONObject getUrl(String url) throws Exception{
    try {
        httpget.setURI(URI.create(url));
        String response = httpclient.execute(httpget,responseHandler);
        JSONObject json = JSONObject.fromObject(response);
        return json;
    } catch (IOException e) {
        e.printStackTrace();
    }
    return null;
}
```