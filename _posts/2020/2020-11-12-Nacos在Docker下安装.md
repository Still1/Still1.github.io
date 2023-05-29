---
title: Nacos在Docker下安装
tags: [Docker, Nacos, 系统运维, 软件开发]
---

## 准备数据库

执行初始化脚本

[https://github.com/alibaba/nacos/blob/master/distribution/conf/mysql-schema.sql](https://github.com/alibaba/nacos/blob/master/distribution/conf/mysql-schema.sql)

## 集群模式启动

Nacos版本2.2.0，外部数据库使用MySQL 5.7

```shell
docker run -p 8848:8848 -p 9848:9848 --name nacos \
--restart always \
-v /etc/timezone:/etc/timezone:ro \
-v /etc/localtime:/etc/localtime:ro \
--env MODE=cluster \
--env NACOS_AUTH_ENABLE=true \
--env NACOS_SERVERS=192.168.88.121,192.168.88.122,192.168.88.123 \
--env NACOS_SERVER_IP=192.168.88.121 \
--env SPRING_DATASOURCE_PLATFORM=mysql \
--env MYSQL_SERVICE_HOST=192.168.88.124 \
--env MYSQL_SERVICE_PORT=3306 \
--env MYSQL_SERVICE_DB_NAME=nacos \
--env MYSQL_SERVICE_USER=root \
--env MYSQL_SERVICE_PASSWORD=mysqlrootroot \
--env MYSQL_DATABASE_NUM=1 \
--env MYSQL_SERVICE_DB_PARAM='characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false&serverTimezone=GMT%2B8' \
-d nacos/nacos-server:v2.2.0
```

NACOS_AUTH_ENABLE=true 开启权限验证
{:.warning}

NACOS_AUTH_TOKEN 自定义密钥
{:.warning}


## 单机模式启动

```shell
docker run -p 8848:8848 -p 9848:9848 --name nacos \
--restart always \
-v /etc/timezone:/etc/timezone:ro \
-v /etc/localtime:/etc/localtime:ro \
--env MODE=standalone \
--env NACOS_AUTH_ENABLE=true \
--env SPRING_DATASOURCE_PLATFORM=mysql \
--env MYSQL_SERVICE_HOST=192.168.88.124 \
--env MYSQL_SERVICE_PORT=3306 \
--env MYSQL_SERVICE_DB_NAME=nacos \
--env MYSQL_SERVICE_USER=root \
--env MYSQL_SERVICE_PASSWORD=mysqlrootroot \
--env MYSQL_DATABASE_NUM=1 \
--env MYSQL_SERVICE_DB_PARAM='characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false&serverTimezone=GMT%2B8' \
-d nacos/nacos-server:v2.2.0
```

## 参考文档

* [https://github.com/nacos-group/nacos-docker](https://github.com/nacos-group/nacos-docker)
* [https://nacos.io/zh-cn/docs/auth.html](https://nacos.io/zh-cn/docs/auth.html)
* [https://github.com/nacos-group/nacos-docker/tree/master/example](https://github.com/nacos-group/nacos-docker/tree/master/example)