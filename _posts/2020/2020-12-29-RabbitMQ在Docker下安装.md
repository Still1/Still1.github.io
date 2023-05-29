---
title: RabbitMQ在Docker下安装
tags: [Docker, RabbitMQ, 系统运维, 软件开发]
---

## RabbitMQ安装

```shell
docker run -p 15672:15672 -p 5672:5672 --name rabbitmq \
--restart always \
-v /etc/timezone:/etc/timezone:ro \
-v /etc/localtime:/etc/localtime:ro \
-d rabbitmq:3.8-management
```

## 配置root用户

```shell
docker exec -it rabbitmq /bin/bash

rabbitmqctl add_user root rabbitmqroot
rabbitmqctl set_permissions -p / root ".*" ".*" ".*"
rabbitmqctl set_user_tags root administrator

# 查看用户是否增加成功
rabbitmqctl list_users
```