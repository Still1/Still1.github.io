---
title: 解决Docker 启动的MySQL容器无法连接 ERROR 1045的问题
tags: [Docker, MySQL, 问题解决]
---

## 问题描述

Docker 启动
```shell
docker run -p 3306:3306 --name mysql \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql:5.7
```

输入以上密码root

![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405180053.png)

无法登陆

## 问题原因

直接原因实际上为密码输入错误
由于-v参数 容器内的MySQL配置会被宿主机的配置覆盖
而由于之前曾经用同一命令运行过容器，但只有密码参数不同，所以已经在宿主机生成了原来密码的配置，导致密码还是之前配置的，而非root

## 解决方法

先删掉宿主机上的配置数据，再次运行此启动命令