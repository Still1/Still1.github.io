---
title: MySQL在Docker下的安装
tags: [Docker, MySQL, 系统运维, 软件开发]
---

## 下载MySQL镜像

```
docker pull mysql:5.7
```

## 创建容器并启动MySQL

先把默认配置文件复制到挂载数据卷里面

```shell
mkdir -p /opt/volume/mysql

docker run --name mysql --rm -e MYSQL_ROOT_PASSWORD=mysqlrootroot -d mysql:5.7
docker cp mysql:/etc/mysql /opt/volume/mysql
mv /opt/volume/mysql/mysql /opt/volume/mysql/conf

docker stop mysql

docker run -p 3306:3306 --name mysql \
--restart always \
--network language-trainer \
-v /etc/timezone:/etc/timezone:ro \
-v /etc/localtime:/etc/localtime:ro \
-v /opt/volume/mysql/log:/var/log/mysql \
-v /opt/volume/mysql/data:/var/lib/mysql \
-v /opt/volume/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=mysqlrootroot \
-d mysql:5.7
```

## 配置默认字符集

新增配置文件`/opt/volume/mysql/conf/conf.d/my.cnf`

```
[mysqld]
init_connect='SET collation_connection = utf8mb4_unicode_ci'
init_connect='SET NAMES utf8mb4'
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
skip-character-set-client-handshake

[client]
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4
```

重启MySQL

```shell
docker restart mysql
```

创建一个数据库，验证字符集是否修改成功

```sql
show variables like 'character%';
```

验证以下变量值是否都为`utf8mb4`

* `character_set_client`
* `character_set_connection`
* `character_set_database`
* `character_set_results`
* `character_set_server`

验证`character_set_system`变量值是否为`utf8`

> [MySQL字符集最佳实践](https://blog.oliverclio.com/2022/07/12/MySQL%E5%AD%97%E7%AC%A6%E9%9B%86%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5.html)