---
title: HAProxy在Docker下安装
tags: [系统运维, HAProxy, 集群系统, Docker]
---

## 准备配置文件

```shell
mkdir -p /opt/volume/haproxy/conf
```

本例子实现MySQL两从服务器的反向代理

`/opt/volume/haproxy/conf/haproxy.cfg`
```
defaults
    mode tcp
    log global
    option tcplog
    option dontlognull
    option http-server-close
    option redispatch
    retries 3
    timeout http-request 10s
    timeout queue 1m
    timeout connect 10s
    timeout client 1m
    timeout server 1m
    timeout http-keep-alive 10s
    timeout check 10s
    maxconn 3000
    
frontend mysql
    bind 0.0.0.0:3306
    mode tcp
    log global
    default_backend mysql_server
    
backend mysql_server
    balance roundrobin
    server mysql1 192.168.88.172:3306 check inter 5s rise 2 fall 3
    server mysql2 192.168.88.173:3306 check inter 5s rise 2 fall 3
    
listen stats
    mode http
    bind 0.0.0.0:1080
    stats enable
    stats hide-version
    stats uri /haproxyamdin?stats
    stats realm Haproxy\ Statistics
    stats auth admin:admin
    stats admin if TRUE
    
```

注意配置文件的最后必须有空行，否则启动时读取配置文件会报错
{:.warning}

## 创建容器并启动

```shell
docker run -p 1080:1080 -p 3306:3306  --name haproxy \
--restart always \
-v /etc/timezone:/etc/timezone:ro \
-v /etc/localtime:/etc/localtime:ro \
-v /opt/volume/haproxy/conf/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg \
-d haproxy
```

## 访问管理页面

通过以下地址访问，其中URI、用户名、密码可在配置文件中指定

[http://192.168.88.221:1080/haproxyamdin?stats](http://192.168.88.221:1080/haproxyamdin?stats)