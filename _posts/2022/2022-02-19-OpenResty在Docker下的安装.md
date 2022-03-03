---
title: OpenResty在Docker下的安装
tags: 
  - OpenResty
  - Docker
modify_date: 2022-02-19
---

## OpenResty安装

<!--more-->

```
docker pull openresty/openresty
docker run --name openResty -p 80:80 -d openresty/openresty

cd /mnt
mkdir -p ./openresty/{conf,html,logs}

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

```
docker exec openResty nginx -t
docker exec openResty nginx -s reload
```

