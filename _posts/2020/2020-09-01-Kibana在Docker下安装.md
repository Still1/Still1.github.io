---
title: Kibana在Docker下安装
tags: [Kibana]
---

## 下载Kibana镜像

```shell
docker pull kibana:7.17.9
```

## 复制默认配置

```shell
mkdir -p /opt/volume/kibana/config

docker run --name kibana --rm -d kibana:7.17.9
docker cp kibana:/usr/share/kibana/config /opt/volume/kibana

docker stop kibana
```

## 修改配置

`/opt/volume/kibana/config/kibana.yml`

```yml
server.host: "0.0.0.0"
server.shutdownTimeout: "5s"
elasticsearch.hosts: [ "http://192.168.88.161:9200" ]
monitoring.ui.container.elasticsearch.enabled: true
```

## 启动Kibana

```shell
docker run -p 5601:5601 --name kibana \
--restart always \
-v /etc/timezone:/etc/timezone:ro \
-v /etc/localtime:/etc/localtime:ro \
-v /opt/volume/kibana/config:/usr/share/kibana/config \
-d kibana:7.17.9
```

## Elasticsearch配置

`config/kibana.yml`

```yaml
server.port: 5601
elasticsearch.hosts： ["http://localhost:9200"]
kibana.index: ".kibana"
i18n.locale: "zh-CN"
```