# Nginx

Nginx是一款高性能的HTTP和反向代理服务器（异步非阻塞的高性能IO模型）。Nginx以高效的epoll、kqueue、eventport作为网络IO模型，在高并发场景下，Nginx能够轻松支持5w并发连接数的响应，并且消耗的服务器内存、CPU等系统资源却很低，运行非常稳定。

想让Nginx支持5万并发，甚至百万级的并发，需要做一些优化
- 服务器内存、CPU硬件支持
- 磁盘使用SSD
- 安装光纤网口
- 使用linux系统，优化内核参数，对TCP连接的设置
- 优化nginx.conf中并发相关的参数
```
worker_processes 16;
worker_connections 50000;
```

## Nginx工作流程架构

### master主进程

- 启动时检查nginx.conf是否正确，语法是否错误
- 根据配置文件的参数创建且监控worker进程的数量和状态
- 监听socket，接收client发起的请求，然后worker竞争抢夺链接，获胜的可以处理且响应请求
- 接收运维发送的管理nginx进程的信号，并且将信号通知到worker进程
- 如果运维发送reload命令，则读取新配置文件，创建新的worker进程，结束旧的worker进程

### worker工作进程原理

- 实际处理client网络请求的是worker
- master根据nginx.conf决定worker的数量
- 有client用户请求到达时，worker之间进程竞争，获胜者和client建立连接且处理用户请求
- 接收用户请求后，若需要代理转发给后端，则后端处理完毕后接收处理结果，再响应给用户
- 接收并处理master发送的进程信号，如启动、重启、重载、停止。

### Nginx处理Http请求

请求报文 --> 逐行解析：请求行 --> 逐行解析：请求头 --> 处理请求（1、匹配虚拟主机 2、location 匹配路径 3、记录日志 access.log） --> 数据压缩 gzip --> 响应报文

## Nginx命令

```
nginx -t # 检测nginx.conf语法
# stop quit reopen reload
nginx -s reload # 重新读取nginx.conf
nginx -s stop # 停止nginx  kill -15 nginx 

nginx # 默认是直接运行 前提是当前机器没运行nginx

nginx -c nginx.conf的路径 # 指定用哪个配置文件
```

`nginx -s reload`是给master进程发信号，重新读取配置信息，导致worker重新生成，因此worker-pid发生了变化。但是master进程id不带变化的。


```
# 现在可以用systemctl去管理nginx了

systemctl start nginx

systemctl status nginx

systemctl reload nginx  # worker变化 master不变

systemctl restart nginx # 整个nginx进程变化
```

## Nginx配置文件

```
http{} 允许内部嵌套多个server{}

server{} 内部允许有多个location{}


http{} 标签用于解决用户的请求与响应整体功能

server {} 用于响应具体一个网站（域名）

location {} 用于匹配网站具体的URL路径
```

## Nginx虚拟主机

### 单虚拟主机

只需要在http{}区域中，设置一个server{}标签即可。

```

server {
    listen 80;
    server_name ip/域名;
    charset utf-8;
    location / {
        root /../../;
        index index.html;
    }

}

```

`etc/nginx/mime.types`只有这个文件中定义的文件类型，Nginx会进行解析，其他文件类型Nginx默认会下载。

### 多IP虚拟主机

Linux操作系统都能够支持给网卡绑定多个IP，可以使得一块网卡上运行多个基于IP的虚拟主机

```
# 临时添加一个IP

ip addr add 10.0.0.88/24 dev eth0


# 查看
ip addr show eth0
```

配置Nginx
```
server {
    listen 10.0.0.88:80;
    server_name _;

    location / {

        root /data/80/;
        index index.html;
    }

}
```

### 多端口的虚拟主机

```
server {
    listen 10.0.0.88:80;
    server_name _;

    location / {

        root /data/80/;
        index index.html;
    }
}

server {
    listen 10.0.0.88:81;
    server_name _;

    location / {

        root /data/81/;
        index index.html;
    }
}

```

## Nginx核心功能

```
user www;

http {
   数据传输性能相关的参数 

   includ /etc/nginx/conf.d/*.conf;
} 定义全局的一些关于http请求响应处理的参数

server {}  区域中主要定义 单个的网站的处理  

    网站的根目录，静态数据存放的地方

server {
    端口
    域名
    URL处理
} 网站1

server {} 网站2

server {} 网站3


```

## Nginx日志

### 日志格式

```
log_format main '$remote_addr - $remote_user [$time_local] "$request"'
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent" "$http_x_forwarder_for"';


access_log logs/access.log main;

参数解释
$remote_addr 记录访问网站的客户端IP地址
$remote_user 记录远程客户端用户名称
$time_local 记录访问时间与时区
$request 记录用户的http请求起始行信息（请求方法、http协议）
$status 记录http状态码，即请求返回的状态 ，例如200、404、502等
$body_bytes_sent 记录服务器发送给客户端的响应body字节数
$http_referer 记录此次请求是从哪个链接访问过来的，可以根据referer进行防盗链设置
$http_user_agent 记录客户端访问信息，如浏览器、手机客户端等
$http_x_forwarder_for 当前端有代理服务器时，设置Web节点记录客户端地址的配置，
    此参数生效的前提是代理服务器上也进行相关的x_forwarded_for设置

备注：
    $remote_addr 可能拿到是反向代理IP地址
    $http_x_forwarder_for  可以获取客户端真实IP地址
    
    更多变量
    https://nginx.org/en/docs/


    log_format ; 设定日志的格式
    access_log ; 设定是否开启日志，日志存储路径   可以写在 http {}  或者 server {}

    http {
        log_format ; 设定日志的格式
        access_log ; 设定是否开启日志，日志存储路径

    }

    http {
        server {
            # log_format ; 设定日志的格式 不允许写在这里
            access_log ; 设定是否开启日志，日志存储路径
        }

    }
```

### 关闭日志

```
#access_log logs/access.log main;
access_log off;
```

### 错误日志

语法

```
error_log file level;

日志级别 debug|info|notice|warn|error|crit|alert|emerg

级别越高，日志记录越少，生产常用模式是warn|error|crit级别

日志记录，会给服务器增加大量的IO消耗，按需修改


error_log /var/log/log-error.log error;
```

## 错误页面

语法

```
#error_page 响应状态码  相对路径的html文件 / url

error_page 404 /404.html
error_page 500 502 503 504 /50x.html
```

