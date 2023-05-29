---
title: Redis在Docker下的安装
tags: [软件开发, 系统运维, Docker, Redis]
---

## 下载redis镜像

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
-d redis:5.0 redis-server --appendonly yes --requirepass "redisredisroot"
```

## Redis 持久化配置

修改`/opt/volume/redis/conf/redis.conf`

```
appendonly yes
```

## 设置redis认证密码

修改`/opt/volume/redis/conf/redis.conf`

```
requirepass yourpassword
```