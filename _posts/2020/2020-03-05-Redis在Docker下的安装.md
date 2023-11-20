---
title: Redis在Docker下的安装
tags: [软件开发, 系统运维, Docker, Redis]
---

## 下载Redis镜像

```shell
docker pull redis:5.0
```

## 创建容器并启动Redis

```shell
mkdir -p /opt/volume/redis/conf
touch /opt/volume/redis/conf/redis.conf

docker run -p 6379:6379 --name redis \
--restart always \
-v /etc/timezone:/etc/timezone:ro \
-v /etc/localtime:/etc/localtime:ro \
-v /opt/volume/redis/data:/data \
-v /opt/volume/redis/conf/redis.conf:/etc/redis/redis.conf \
-d redis:5.0 redis-server /etc/redis/redis.conf
```

## Redis配置

### 默认配置

默认配置下载地址

[https://redis.io/docs/management/config/](https://redis.io/docs/management/config/)

### 重点配置修改

修改`/opt/volume/redis/conf/redis.conf`

```
# 允许所有远程连接
bind 0.0.0.0

# 开启持久化
appendonly yes

# 设置认证密码
requirepass redisredisroot
```