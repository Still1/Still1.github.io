---
title: Elasticsearch部署配置
tags: [Elasticsearch]
---

## Linux系统配置

`/etc/security/limits.conf`
```
# 每个进程可以打开的文件数限制
es soft nofile 65536
es hard nofile 65536
```

`/etc/security/limits.d/20-nproc.conf`
```
# 每个进程可以打开的文件数限制
es soft nofile 65536
es hard nofile 65536
```

`/etc/sysctl.conf`
```
# 一个进程可以拥有的VMA（虚拟内存区域）数量，默认65536
vm.max_map_count=655360
```

重新加载配置
```shell
sysctl -p
```

## JVM配置

最大堆内存不超过系统总内存的一半，例如系统总内存为64G

`config/jvm.options`
```
-Xms 31g
-Xmx 31g
```

## 集群配置

`config/elasticsearch.yml`
```yaml
# 集群名称
cluster.name:my-application
# 节点名称
node.name: node-1001
# 主机名称
network.host: localhost
network.host: 0.0.0.0
# 端口号
http.port: 1001
# TCP监听
transport.tcp.port: 9301
# 节点是否可以为master
node.master: true
# 节点是否可以为data
node.data: true
# 允许跨域
http.cors.enabled: true
http.cors.allow-origin: "*"
# 最大内容长度
http.max_content_length: 200mb
# 发现集群
discovery.seed_hosts: ["localhost:9301","localhost:9302"]
discovery.zen.fd.ping_timeout: 1m
discovery.zen.fd.ping_retries: 5
# 集群初始主节点
cluster.initial_master_nodes: ["node-1"]

gateway.recover_after_nodes: 2
network.tcp.keep_alive: true
network.tcp.no_delay: true
# 节点之间传输数据是否压缩
transport.tcp.compress: true

# 集群内同时启动的数据任务个数，默认为2
cluster.routing.allocation.cluster_concurrent_rebalance: 16
# 添加或删除节点及负载均衡时并发恢复的线程个数，默认为4
cluster.routing.allocation.node_concurrent_recoveries: 16
# 初始化数据恢复时，并发恢复线程的个数，默认为4
cluster.routing.allocation.node_initial_primaries_recoveries: 16
```

查看集群状态

```
GET http://localhost:1001/_cluster/health
GET http://localhost:1001/_cat/nodes
```

## 分片与副本配置

创建Index时指定

```
PUT http://localhost:1001/users
```

```json
{
    "setting": {
        "number_of_shards": 3,
        "number_of_replicas": 1
    }
}
```

 修改Index只能修改副本数

```
PUT http://localhost:1001/users/_settings
```

```json
{
	"number_of_replicas": 2
}
```