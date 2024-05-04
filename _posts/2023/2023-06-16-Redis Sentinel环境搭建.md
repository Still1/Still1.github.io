---
title: Redis Sentinel环境搭建
tags: [Redis]
---

## 前言

在搭建Redis sentinel环境之前，先基于Docker安装至少三台Redis服务器

> [Redis在Docker下的安装](https://blog.oliverclio.com/2020/03/05/Redis%E5%9C%A8Docker%E4%B8%8B%E7%9A%84%E5%AE%89%E8%A3%85.html)

## Redis从机配置

修改`/opt/volume/redis/conf/redis.conf`

```
# 从机配置主机地址
replicaof 192.168.88.151 6379

# 从机配置主机认证密码（开启sentinel的情况下，主机也需要配置，因为主机可能转为从机）
masterauth redisredisroot
```

## 检查主从配置

```shell
docker exec -it redis /bin/bash
redis-cli
auth redisredisroot
info replication
```

一主二从主机的输出结果

```
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.88.153,port=6379,state=online,offset=196,lag=1
slave1:ip=192.168.88.152,port=6379,state=online,offset=196,lag=1
```

## 开启Sentinel

### Sentinel配置文件

```
# 允许所有远程连接
bind 0.0.0.0

# 对外访问的IP
sentinel announce-ip "192.168.88.153"

# 对外访问的端口号
sentinel announce-port 26379

# redis主机地址配置，以及达到客观下线的票数要求
sentinel monitor mymaster 192.168.88.151 6379 2

# redis主机和从机的认证密码
sentinel auth-pass mymaster redisredisroot

# redis主观下线的失效时间
sentinel down-after-milliseconds mymaster 30000

# 主备切换时，最多可同时同步的从机数量
sentinel parallel-syncs mymaster 1
```

### 启动sentinel

sentinel也需要满足高可用的需求，最好部署3台或以上，且与redis服务器分离

```shell
mkdir -p /opt/volume/sentinel/conf

docker run -p 26379:26379 --name sentinel \
--restart always \
-v /etc/timezone:/etc/timezone:ro \
-v /etc/localtime:/etc/localtime:ro \
-v /opt/volume/sentinel/data:/data \
-v /opt/volume/sentinel/conf/sentinel.conf:/etc/redis/sentinel.conf \
-d redis:5.0 redis-sentinel /etc/redis/sentinel.conf
```

## Spring Data Redis连接Sentinel

```yml
redis:  
  sentinel:  
    master: mymaster  
    nodes: 192.168.88.151:26379,192.168.88.152:26379,192.168.88.153:26379  
  timeout: 10000  
  database: 0  
  lettuce:  
    pool:  
      enabled: true  
  password: redisredisroot
```