---
title: Seata在Docker下安装
tags: [Seata]
---

## 运行临时容器并复制默认配置

```shell
mkdir -p /opt/volume/seata

docker run -d -p 8091:8091 -p 7091:7091 --name seata --rm seataio/seata-server:1.7.0
docker cp seata:/seata-server/resources /opt/volume/seata/conf

docker stop seata
```

## 启动容器

```shell
docker run --name seata \
--restart always \
-p 8091:8091 \
-p 7091:7091 \
-v /opt/volume/seata/conf:/seata-server/resources \
-v /etc/timezone:/etc/timezone:ro \
-v /etc/localtime:/etc/localtime:ro \
-d seataio/seata-server:1.7.0
```