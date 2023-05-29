---
title: Elasticsearch在Docker下安装
tags: [Docker, Elasticsearch, 系统运维, 软件开发]
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
-e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
-e "discovery.type=single-node" \
--privileged \
-d elasticsearch:7.17.9
```

## 安装IK分词器

下载Elasticsearch对应版本的IK分词器的zip包

https://github.com/medcl/elasticsearch-analysis-ik/releases

在plugins文件夹下新建一个文件夹，将zip包的解压内容放到这个新建文件夹中，再重启ElasticSearch即可

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230522083552.png" width="800px" />

当找不到Elasticsearch对应版本的IK分词器时，可修改IK分词器插件的`plugin-descriptor.properties`文件中的对应版本

```properties
elasticsearch.version=7.17.9
```