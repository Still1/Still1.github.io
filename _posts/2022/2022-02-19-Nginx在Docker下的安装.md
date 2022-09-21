---
title: Nginx在Docker下的安装
tags: 
  - Nginx
  - Docker
modify_date: 2022-02-19
---

## Nginx安装

Nginx挂载数据卷，配置文件夹和页面文件夹不会自动复制到宿主机，日志文件夹会自动复制。

需要先把配置文件手动复制到挂载数据卷里面，否则无法正常启动。页面文件可以根据需要决定是否手动复制。

<!--more-->

```shell
mkdir -p /opt/volumn/nginx

docker run --name nginx --rm -d nginx
docker cp nginx:/etc/nginx /opt/volumn/nginx
mv /opt/volumn/nginx/nginx /opt/volumn/nginx/conf

# 复制页面文件夹内容，可选
docker cp nginx:/usr/share/nginx/html /opt/volumn/nginx

docker stop nginx

docker run -p 80:80 --name nginx \
--restart always \
-v /opt/volumn/nginx/conf:/etc/nginx \
-v /opt/volumn/nginx/html:/usr/share/nginx/html \
-v /opt/volumn/nginx/log:/var/log/nginx \
-d nginx
```

## OpenResty安装

```shell
mkdir -p /mnt/openresty/{conf,html,logs}

docker pull openresty/openresty
docker run --name openResty -p 80:80 -d openresty/openresty

docker cp openResty:/usr/local/openresty/nginx/conf/nginx.conf /mnt/openresty/conf/nginx.conf
docker cp openResty:/etc/nginx/conf.d/default.conf /mnt/openresty/conf/default.conf

docker stop openResty
docker rm openResty

docker run --name openResty -p 80:80 -d \
--restart always \
-v /mnt/openresty/conf/nginx.conf:/usr/local/openresty/nginx/conf/nginx.conf \
-v /mnt/openresty/conf/default.conf:/etc/nginx/conf.d/default.conf \
-v /mnt/openresty/html:/usr/local/openresty/nginx/html \
-v /mnt/openresty/logs:/usr/local/openresty/nginx/logs \
-v /etc/localtime:/etc/localtime \
openresty/openresty
```

## 更新配置文件

```shell
docker exec openResty nginx -t
docker exec openResty nginx -s reload
```

