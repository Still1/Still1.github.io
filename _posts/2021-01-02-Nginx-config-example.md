---
title: Nginx 反向代理、动静分离、负载均衡简单配置例子
tags: 
  - Nginx
modify_date: 2021-01-02

---

## 前言

本文将介绍如何对Nginx进行反向代理、动静分离、负载均衡的简单配置。所有配置均通过修改Nginx安装目录下的conf文件夹里面的`nginx.conf`文件。

<!--more-->

## 配置过程

### 反向代理

修改http -> server -> location

```xml
location / {
    proxy_pass http://localhost:10050;
}
```

访问URL：

`http://localhost/doubucket/items/hello`Nginx会把请求转移到 `http://localhost:10050/doubucket/items/hello`，从而实现反向代理。

### 动静分离

修改http -> server -> location

```xml
location /static/ {
    root   html;
}

location / {
    proxy_pass http://localhost:10050;
}
```

访问URL：

`http://localhost/static/test.html` 则直接访问Nginx安装目录下 `html/static/test.html`

`http://localhost/doubucket/items/hello` 则同样通过反向代理访问 `http://localhost:10050/doubucket/items/hello`

这样，静态资源Nginx服务器直接返回，动态资源通过反向代理访问其它服务器获得，实现动静分离。

### 负载均衡

修改http

```xml
upstream doubucket {
    server localhost:10050;
    server localhost:10051;
}
```

修改http -> server -> location

```xml
location / {
    proxy_pass http://doubucket;
}
```

访问URL：

`http://localhost/doubucket/items/hello` 则同样通过反向代理访问 

`http://localhost:10050/doubucket/items/hello`或者 `http://localhost:10051/doubucket/items/hello`

通过设置多个上游服务器，实现简单的负载均衡。

## 总结

至此，Nginx的反向代理、动静分离、负载均衡简单配置完成了，附上完整`nginx.conf`文件内容

```xml

worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    upstream doubucket {
        server localhost:10050;
        server localhost:10051;
    }

    sendfile        on;

    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;

        location /static/ {
            root   html;
        }

        location / {
            proxy_pass http://doubucket;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}

```



## Reference

* [Nginx documentation](https://nginx.org/en/docs/)