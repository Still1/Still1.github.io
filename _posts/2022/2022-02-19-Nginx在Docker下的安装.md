---
title: Nginx在Docker下的安装
tags: [Nginx]
---

## Nginx安装

Nginx挂载数据卷，配置文件夹和页面文件夹不会自动复制到宿主机，日志文件夹会自动复制。

需要先把配置文件手动复制到挂载数据卷里面，否则无法正常启动。页面文件可以根据需要决定是否手动复制。

```shell
mkdir -p /opt/volume/nginx

docker run --name nginx --rm -d nginx
docker cp nginx:/etc/nginx /opt/volume/nginx
mv /opt/volume/nginx/nginx /opt/volume/nginx/conf

# 复制页面文件夹内容，可选
docker cp nginx:/usr/share/nginx/html /opt/volume/nginx

docker stop nginx

docker run -p 80:80 --name nginx \
--restart always \
--network language-trainer \
-v /etc/timezone:/etc/timezone:ro \
-v /etc/localtime:/etc/localtime:ro \
-v /opt/volume/nginx/conf:/etc/nginx \
-v /opt/volume/nginx/html:/usr/share/nginx/html \
-v /opt/volume/nginx/log:/var/log/nginx \
-d nginx
```