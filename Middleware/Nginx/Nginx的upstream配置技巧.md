# Nginx的upstream配置技巧

## 介绍
### 介绍

`Nginx`是一个反向代理软件，大部分的网站都采用Nginx作为网站/平台的服务器软件。`Nginx`除了可以直接作为web服务器使用外，更多的情况是通过反向代理将请求转发给上游服务器。配置上游服务器可以使用`upstream`进行设置，通过`upstream`可以实现服务的负载均衡规则，可以提高服务器的高可用性。

## 语法
### 基本语法

`upstream`的基本语法如下，一个`upstream`需要设置一个名称，这个名称可以在`server`里面当作`proxy`主机使用。

```
upstream default {
    server  php-fpm-tfphp:9000;
}
```

一个`upstream`可以设置多个`server`，通常情况下`Nginx`会轮询每一个`server`，从而达到最基本的负载循环效果。

```
upstream default {
    server  tflinux_php-fpm-tfphp_1:9000;
    server  tflinux_php-fpm-tfphp_2:9000;
}
```

### max_fails

`max_fails`是最多出错数量，可以为每一个`server`设置一个`max_fails`，如果请求`server`发生了错误则`max_fails`会加一，如果请求`server`错误次数达到了`max_fails`后，`Nginx`会标记这个`server`为故障状态，后面就不会再去请求它了。

默认情况下，`max_fails`的次数是1次。
```
upstream default {
    server  tflinux_php-fpm-tfphp_1:9000 max_fails=5;
    server  tflinux_php-fpm-tfphp_2:9000 max_fails=3;
}
```

### fail_timeout
`fail_timeout`是故障等待超时时间，前面说过了`max_fails`是请求`server`错误次数，如果达到了`max_fails`次数之后`server`会被标记为故障状态，那么多长时间会重新尝试呢？这个`fail_timeout`就是这个时间了，在达到`max_fails`次数之后`server`进入故障状态，而后在`fail_timeout`时间之后会被重新标记为正常状态。

默认情况下，`fail_timeout`的时间是10秒。

```
upstream default {
    server  tflinux_php-fpm-tfphp_1:9000 max_fails=5 fail_timeout=100;
    server  tflinux_php-fpm-tfphp_2:9000 max_fails=3 fail_timeout=60;
}
```

### proxy_connect_timeout
这个`proxy_connect_timeout`是连接超时时间，如果连接不到就会报错了。
```
proxy_connect_timeout 3s;
```

### proxy_next_upstream_tries
这个`proxy_next_upstream_tries`是一个`upstream`反向代理的重试次数，简单说就是如果请求`server`出错的次数达到了`proxy_next_upstream_tries`的次数的话，即使没有达到`max_fails`的次数，即使后面还有没有尝试过的`server`，都不会再继续尝试了，而是直接报错。

```
proxy_next_upstream_tries 3;
```

### proxy_next_upstream_timeout
这个`proxy_next_upstream_timeout`是一个`upstream`反向代理的故障等待时间，简单说就是无论`upstream`内部如何进行重试，所有花费的时间加在一起达到了`proxy_next_upstream_timeout`时间的话，就会直接报错，不会再继续尝试了。
```
proxy_next_upstream_timeout 60s;
```

### backup

这个是备用服务器参数，可以为一个`upstream`设置一个`backup`的`server`，在生产`server`全部都出问题之后，可以自动切换到备用`server`上，为回复服务争取时间。

`backup`的`server`不同于其他`server`，平时是不承载请求的，所以它应该是比较空闲的状态，应急再合适不过了~~

```
upstream default {
    server  tflinux_php-fpm-tfphp_1:9000 max_fails=5 fail_timeout=100;
    server  tflinux_php-fpm-tfphp_2:9000 max_fails=3 fail_timeout=60;
    server  tflinux_php-fpm-tfphp_3:9000 backup;
}
```

## 示例
```
proxy_connect_timeout 3s;
proxy_next_upstream_timeout 60s;
proxy_next_upstream_tries 3;

upstream default {
    server  tflinux_php-fpm-tfphp_1:9000 max_fails=5 fail_timeout=100;
    server  tflinux_php-fpm-tfphp_2:9000 max_fails=3 fail_timeout=60;
    server  tflinux_php-fpm-tfphp_3:9000 backup;
}
```