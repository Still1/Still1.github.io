---
title: Redis在Docker下的安装
tags: 
  - Redis
  - Docker
modify_date: 2020-03-05
---

## 下载redis镜像

<!--more-->

```
docker pull redis:5.0
```

## 创建容器并启动redis

```
mkdir -p /mydata/redis/conf
touch /mydata/redis/conf/redis.conf

docker run -p 16333:6379 --name redis \
-v /mydata/redis/data:/data \
-v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
-d redis:5.0 redis-server /etc/redis/redis.conf
```

## Redis 持久化配置

修改`/mydata/redis/conf/redis.conf`

```
appendonly yes
```

## 设置redis认证密码

修改`/mydata/redis/conf/redis.conf`

```
requirepass yourpassword
```
