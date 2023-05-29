---
title: Zookeeper在Docker下安装
tags: [Docker, Zookeeper, 系统运维, 软件开发]
---

## Zookeeper安装

集群模式启动

```shell
docker run -p 2181:2181 --name zookeeper \
--restart always \
-v /opt/volume/zookeeper/data:/data \
-e ZOO_MY_ID=1 \
-e ZOO_SERVERS='server.1=192.168.88.131:2888:3888;2181 server.2=192.168.88.132:2888:3888;2181 server.3=192.168.88.133:2888:3888;2181' \
-d zookeeper:3.8.0
```

单机模式启动

```shell
docker run -p 2181:2181 --name zookeeper \
--restart always \
-v /opt/volume/zookeeper/data:/data \
-d zookeeper:3.8.0
```