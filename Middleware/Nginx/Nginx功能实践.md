# Nginx功能实践

## 限流
### 限流算法
#### 令牌桶算法
算法思想
- 令牌以固定速率产生，并缓存到令牌桶中；
- 令牌桶放满时，多余的令牌被丢弃；
- 请求要消耗等比例的令牌才能被处理；
- 令牌不够时，请求被缓存。

#### 漏桶算法
算法思想
- 水（请求）从上方倒入水桶，从水桶下方流出（被处理）；
- 来不及流出的水存在水桶中（缓冲），以固定速率流出；
- 水桶满后水溢出（丢弃）。

这个算法的核心是：缓存请求、匀速处理、多余的请求直接丢弃。

相比漏桶算法，令牌桶算法不同之处在于它不但有一只“桶”，还有个队列，这个桶是用来存放令牌的，队列才是用来存放请求的。

从作用上来说，漏桶和令牌桶算法最明显的区别就是是否允许突发流量(burst)的处理，漏桶算法能够强行限制数据的实时传输（处理）速率，对突发流量不做额外处理；而令牌桶算法能够在限制数据的平均传输速率的同时允许某种程度的突发传输。

### Nginx限流
Nginx官方版本限制IP的连接和并发分别有两个模块：
- limit_req_zone 用来限制单位时间内的请求数，即速率限制,采用的漏桶算法 "leaky bucket"。
- limit_req_conn 用来限制同一时间连接数，即并发限制。

#### limit_req_zone 参数配置
```
Syntax: limit_req zone=name [burst=number] [nodelay];
Default: —
Context: http, server, location

limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
```

- $binary_remote_addr 表示通过remote_addr这个标识来做限制，“binary_”的目的是缩写内存占用量，是限制同一客户端ip地址。
- zone=one:10m表示生成一个大小为10M，名字为one的内存区域，用来存储访问的频次信息。
- rate=1r/s表示允许相同标识的客户端的访问频次，这里限制的是每秒1次，还可以有比如30r/m的。limit_req zone=one burst=5 nodelay;
- zone=one 设置使用哪个配置区域来做限制，与上面limit_req_zone 里的name对应。
- burst=5，重点说明一下这个配置，burst爆发的意思，这个配置的意思是设置一个大小为5的缓冲区当有大量请求（爆发）过来时，超过了访问频次限制的请求可以先放到这个缓冲区内。
- nodelay，如果设置，超过访问频次而且缓冲区也满了的时候就会直接返回503，如果没有设置，则所有请求会等待排队。

案例
```
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
    server {
        location /search/ {
            limit_req zone=one burst=5 nodelay;
        }
}
```
其他参数
```
Syntax: limit_req_log_level info | notice | warn | error;
Default:
limit_req_log_level error;
Context: http, server, location
```
当服务器由于limit被限速或缓存时，配置写入日志。延迟的记录比拒绝的记录低一个级别。例子：limit_req_log_level notice延迟的的基本是info。
```
Syntax: limit_req_status code;
Default:
limit_req_status 503;
Context: http, server, location
```
设置拒绝请求的返回值。值只能设置 400 到 599 之间。

#### ngx_http_limit_conn_module 参数配置
这个模块用来限制单个IP的请求数。并非所有的连接都被计数。只有在服务器处理了请求并且已经读取了整个请求头时，连接才被计数。

一次只允许每个IP地址一个连接。
```
Syntax: limit_conn zone number;
Default: —
Context: http, server, location

limit_conn_zone $binary_remote_addr zone=addr:10m;

server {
    location /download/ {
    limit_conn addr 1;
}
```

可以配置多个limit_conn指令。例如，以上配置将限制每个客户端IP连接到服务器的数量，同时限制连接到虚拟服务器的总数。
```
limit_conn_zone $binary_remote_addr zone=perip:10m;
limit_conn_zone $server_name zone=perserver:10m;

server {
    ...
    limit_conn perip 10;
    limit_conn perserver 100;
}
```
在这里，客户端IP地址作为关键。请注意，不是`$remote_addr`，而是使用`$ binary_remote_addr`变量。`$remote_addr`变量的大小可以从7到15个字节不等。存储的状态在32位平台上占用32或64字节的内存，在64位平台上总是占用64字节。对于IPv4地址，`$ binary_remote_addr`变量的大小始终为4个字节，对于IPv6地址则为16个字节。存储状态在32位平台上始终占用32或64个字节，在64位平台上占用64个字节。一个兆字节的区域可以保持大约32000个32字节的状态或大约16000个64字节的状态。如果区域存储耗尽，服务器会将错误返回给所有其他请求。

```
Syntax: limit_conn_zone key zone=name:size;
Default: —
Context: http

limit_conn_zone $binary_remote_addr zone=addr:10m;
```
当服务器限制连接数时，设置所需的日志记录级别。

```
Syntax: limit_conn_log_level info | notice | warn | error;
Default:
limit_conn_log_level error;
Context: http, server, location
```

设置拒绝请求的返回值。
```
Syntax: limit_conn_status code;
Default:
limit_conn_status 503;
Context: http, server, location
```

### 实战
#### 限制访问速率
```
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=2r/s;
server {
    location / {
        limit_req zone=mylimit;
    }
}
```
我们使用单个IP在10ms内发并发送了6个请求，只有1个成功，剩下的5个都被拒绝。我们设置的速度是2r/s，为什么只有1个成功呢，是不是Nginx限制错了？当然不是，是因为Nginx的限流统计是基于毫秒的，我们设置的速度是2r/s，转换一下就是500ms内单个IP只允许通过1个请求，从501ms开始才允许通过第二个请求。

####  burst缓存处理
```
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=2r/s;
server {
    location / {
        limit_req zone=mylimit burst=4;
    }
}
```

我们加入了burst=4，意思是每个key(此处是每个IP)最多允许4个突发请求的到来。如果单个IP在10ms内发送6个请求，结果会怎样呢？

相比实例一成功数增加了4个，这个我们设置的burst数目是一致的。具体处理流程是：1个请求被立即处理，4个请求被放到burst队列里，另外一个请求被拒绝。通过burst参数，我们使得Nginx限流具备了缓存处理突发流量的能力。

但是请注意：burst的作用是让多余的请求可以先放到队列里，慢慢处理。如果不加nodelay参数，队列里的请求不会立即处理，而是按照rate设置的速度，以毫秒级精确的速度慢慢处理。

**延续实例二的配置，我们加入nodelay选项**
```
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=2r/s;
server {
    location / {
        limit_req zone=mylimit burst=4 nodelay;
    }
}
```
跟实例二相比，请求成功率没变化，但是总体耗时变短了。这怎么解释呢？实例二中，有4个请求被放到burst队列当中，工作进程每隔500ms(rate=2r/s)取一个请求进行处理，最后一个请求要排队2s才会被处理；实例三中，请求放入队列跟实例二是一样的，但不同的是，队列中的请求同时具有了被处理的资格，所以实例三中的5个请求可以说是同时开始被处理的，花费时间自然变短了。

但是请注意，虽然设置burst和nodelay能够降低突发请求的处理时间，但是长期来看并不会提高吞吐量的上限，长期吞吐量的上限是由rate决定的，因为nodelay只能保证burst的请求被立即处理，但Nginx会限制队列元素释放的速度，就像是限制了令牌桶中令牌产生的速度。

#### 自定义返回值
```
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=2r/s;
server {
    location / {
        limit_req zone=mylimit burst=4 nodelay;
        limit_req_status 598;
    }
}
```


## 负载均衡
所谓的负载均衡就是把很多请求进行分流，将他们分配到不同的服务器去处理。比如我有3个服务器，分别为A、B、C，然后使用Nginx进行负载均衡，使用轮询策略，此时如果收到了9个请求，那么会均匀的将这9个请求分发给A、B、C服务器，每一个服务器处理3个请求，这样的话我们可以利用多台机器集群的特性减少单个服务器的压力。

Nginx通过反向代理可以实现服务的负载均衡，避免了服务器单节点故障，把请求按照一定的策略转发到不同的服务器上，达到负载的效果。

### 负载均衡策略
#### 轮询
Round Robin: 对所有的请求进行轮询发送请求，默认的分配方式。

将请求按顺序轮流地分配到后端服务器上，它均衡地对待后端的每一台服务器，而不关心服务器实际的连接数和当前的系统负载。

```
upstream server1 {
   server www.test.com;
   server www.test1.com;
}
```

#### 加权轮询
不同的后端服务器可能机器的配置和当前系统的负载并不相同，因此它们的抗压能力也不相同。

给配置高、负载低的机器配置更高的权重，让其处理更多的请；而配置低、负载高的机器，给其分配较低的权重，降低其系统负载，加权轮询能很好地处理这一问题，并将请求顺序且按照权重分配到后端。
```
upstream server2 {
    server ip:prot weight=1;
    server ip:prot weight=2;
}
```

#### 随机
通过系统的随机算法，根据后端服务器的列表大小值来随机选取其中的一台服务器进行访问。

每个请求将被传递到随机选择的服务器。如果指定了两个参数，首先，NGINX根据服务器权重随机选择两个服务器，然后使用指定的方法选择其中一个。

- least_conn ：活动连接的最少数量
- least_time=header (NGINX Plus)：从服务器接收响应标头的最短平均时间 ($upstream_header_time)。
- least_time=last_byte (NGINX Plus) ：从服务器接收完整响应的最短平均时间（$upstream_response_time）。

```
upstream sever3 { 
    random two least_time=last_byte; 
    server www.test.com;
    server www.test1.com;
}
```

#### 源地址哈希法
根据获取客户端的IP地址，通过哈希函数计算得到一个数值，用该数值对服务器列表的大小进行取模运算，得到的结果便是客户端要访问服务器的序号。

采用源地址哈希法进行负载均衡，同一IP地址的客户端，当后端服务器列表不变时，它每次都会映射到同一台后端服务器进行访问。
```
upstream sever4 {
     ip_hash;
     server www.test.com;
     server www.test1.com;
}
```
#### 最小连接数法
由于后端服务器的配置不尽相同，对于请求的处理有快有慢，最小连接数法根据后端服务器当前的连接情况，动态地选取其中当前积压连接数最少的一台服务器来处理当前的请求，尽可能地提高后端服务的利用效率，将负责合理地分流到每一台服务器。

```
upstream sever5 {
    least_time header;
    server www.test.com;
    server www.test1.com;
}
```
- header ：从服务器接收第一个字节的时间。
- last_byte：从服务器接收完整响应的时间。
- last_byte inflight：从服务器接收完整响应的时间。

完整案例
```
events {
    worker_connections  1024;
}

error_log nginx-error.log info;
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

        upstream myserver{
        server 127.0.0.1:8085;
        server 127.0.0.1:8086;
    }

    server {
        listen       80;
        server_name  127.0.0.1;


        location / {
            root   html;
            proxy_pass http://myserver;
            proxy_connect_timeout 3s;
            proxy_read_timeout 5s;
            proxy_send_timeout 3s;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

## 静态代理

Nginx擅长处理静态文件，是非常好的图片、文件服务器。把所有的静态资源的放到nginx上，可以使应用动静分离，性能更好。

## 黑白名单
### 黑名单
```
location / {
    deny ip;
    allow ip;
    deny all;
}
```

### 白名单
```
geo $limit {
    ip 0;
}

map $limit $limit_key {
    1 $binary_remote_addr;
    0 "";
}
```

## 缓存
### 浏览器缓存
静态资源缓存用expire
```
location ~ .*\.(?:jpg|jpeg|gif|png|ico|cur|gz|svg|svgz|mp4|ogg|ogv|webm) $ {
    expire 7d;
}

location ~ .*\.(?:js|css)$ {
    expires 7d;
}
```

### 代理层缓存
```
proxy_cache_path /data/cache/nginx/ levels=1:2 keys_zone=cache:512m inactive = 1d max_size=8g;

location / {
    location ~ \.(html|html)?$ {
        proxy_cache cache;
        proxy_cache_key $uri$is_args$args; //以此变量值做Hash，作为key
        add_header X-Cache $upstream_cache_status;
        proxy_cache_valid 200 10m;
        proxy_cache_valid any 1m;
        proxy_pass http://real_server;
        proxy_redirect off;
    }
    
    location ~ .*\.(gif|jpg|jpeg|bmp|png|ico|txt|js|css)$ {
        root /data/webapps/edc;
        expires 3d;
        add_header Static Nginx-Proxy;
    }
}
```
