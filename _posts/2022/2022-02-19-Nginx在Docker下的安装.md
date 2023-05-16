---
title: Nginx在Docker下的安装
tags: [Docker, Nginx, 系统运维, 软件开发]
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

## 配置文件

修改`conf/conf.d/default.conf`

简单例子

```
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;
   
    location /api/ {
        proxy_pass http://language-trainer:8080/;
    }    

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

## 更新配置文件

```shell
docker exec openResty nginx -t
docker exec openResty nginx -s reload
```