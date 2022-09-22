---
title: MySQL在Docker下的安装
tags: 
  - MySQL
  - Docker
modify_date: 2020-03-05
---

## 下载MySQL镜像

<!--more-->

```
docker pull mysql:5.7
```

## 创建容器并启动MySQL

```shell
docker run -p 3306:3306 --name mysql \
-v /opt/volume/mysql/log:/var/log/mysql \
-v /opt/volume/mysql/data:/var/lib/mysql \
-v /opt/volume/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=mysqlrootroot \
-d mysql:5.7
```

## 配置默认字符集

修改`my.cnf`

```
[mysqld]
init_connect='SET collation_connection = utf8mb4_unicode_ci'
init_connect='SET NAMES utf8mb4'
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
skip-character-set-client-handshake
skip-name-resolve

[client]
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4
```
