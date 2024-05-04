---
title: Elasticsearch在Docker下安装
tags: [Elasticsearch]
---

## Elasticsearch安装

单机模式启动

先设置数据卷权限

```shell
mkdir -p /opt/volume/elasticsearch/data
chmod -R 777 /opt/volume/elasticsearch/data/
```

```shell
docker run -p 9200:9200 -p 9300:9300 --name elasticsearch \
--restart always \
-v /etc/timezone:/etc/timezone:ro \
-v /etc/localtime:/etc/localtime:ro \
-v /opt/volume/elasticsearch/data:/usr/share/elasticsearch/data \
-v /opt/volume/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-v /opt/volume/elasticsearch/config:/usr/share/elasticsearch/config \
-e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
-e "discovery.type=single-node" \
--privileged \
-d elasticsearch:7.17.9
```