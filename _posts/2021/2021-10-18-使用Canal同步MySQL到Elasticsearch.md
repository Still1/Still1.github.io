---
title: 使用Canal同步MySQL到Elasticsearch
tags: [软件开发, Java, Canal, RabbitMQ, Elasticsearch, MySQL]
---

## MySQL配置

开启MySQL的binlog写入，并指定binlog为ROW模式

`my.cnf`

```
[mysqld]
log-bin=mysql-bin
binlog-format=ROW
server_id=1
```

添加Canal使用的用户

```sql
CREATE USER canal IDENTIFIED BY 'canal'; 
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
FLUSH PRIVILEGES;
```

## Canal配置

安装Canal

先把默认配置信息复制出来

```shell
mkdir -p /opt/volume/canal

docker run --name canal --rm -d canal/canal-server
docker cp canal:/home/admin/canal-server/conf /opt/volume/canal

docker stop canal
```

配置文件

`/opt/volume/canal/example/instance.properties`

```
canal.instance.master.address=192.168.88.171:3306

# binlog日志名
canal.instance.master.journal.name=
# binlog起始偏移量
canal.instance.master.position=
# binlog起始时间戳
canal.instance.master.timestamp=

canal.instance.dbUsername=canal
canal.instance.dbPassword=canal

# 监听所有表，指定库写法为db\..*
canal.instance.filter.regex=.*\\..*

# 路由键
canal.mq.topic=example
```

查看binlog信息

```sql
show master status;
```

`/opt/volume/canal/canal.properties`

```
# 关联到rabbitMQ配置
canal.serverMode = rabbitMQ

rabbitmq.host =192.168.88.141
rabbitmq.virtual.host = /
rabbitmq.exchange = canalExchange
rabbitmq.username = root
rabbitmq.password = rabbitmqroot
rabbitmq.deliveryMode = fanout
```

启动Canal

```shell
docker run -p 11111:11111 --name canal \
--restart always \
-v /etc/timezone:/etc/timezone:ro \
-v /etc/localtime:/etc/localtime:ro \
-v /opt/volume/canal/conf:/home/admin/canal-server/conf \
-d canal/canal-server
```

## RabbitMQ 配置

创建名为canalExchage的交换机

![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230402040600.png)

创建名为canalQueue的队列

![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230402040638.png)

绑定交换机和队列

![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230402040756.png)

## 测试队列消息

在数据库里执行一些语句，查看对应队列已有新的消息

![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230402040940.png)

## 消费队列消息

Maven依赖

```xml
<dependency>  
   <groupId>org.springframework.boot</groupId>  
   <artifactId>spring-boot-starter-amqp</artifactId>  
</dependency>
```

RabbitMQ配置

```yml
spring:  
  rabbitmq:  
    host: 192.168.88.141  
    port: 5672  
    username: root  
    password: rabbitmqroot  
    virtual-host: /  
    listener:  
      simple:  
        acknowledge-mode: manual
```

消费队列消息，将数据同步到Elasticsearch

```xml
<dependency>  
   <groupId>co.elastic.clients</groupId>  
   <artifactId>elasticsearch-java</artifactId>  
</dependency>
```

```java
@Component  
public class CanalMessageConsumer {  
  
    private static final String MOVIES_INDEX_NAME = "movies";  
  
    @SneakyThrows  
    @RabbitListener(queues = "canalQueue")  
    public void movieItemsConsumer(@Payload String message, @Header(value = AmqpHeaders.DELIVERY_TAG) long deliveryTag, Channel channel) {  
        JSONObject jsonObject = JSON.parseObject(message);  
        Boolean isDdl = jsonObject.getBoolean("isDdl");  
        if(!isDdl) {  
            String type = jsonObject.getString("type");  
            if ("INSERT".equals(type) || "UPDATE".equals(type)) {  
                RestClient restClient = RestClient.builder(new HttpHost("192.168.88.161", 9200)).build();  
                ObjectMapper objectMapper = new ObjectMapper();  
                objectMapper.setPropertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE);  
                objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);  
                ElasticsearchTransport elasticsearchTransport = new RestClientTransport(restClient,
                    new JacksonJsonpMapper(objectMapper));  
                ElasticsearchClient elasticsearchClient = new ElasticsearchClient(elasticsearchTransport);  
  
                String table = jsonObject.getString("table");  
                if ("t_movie".equals(table)) {  
                    JSONArray dataArray = jsonObject.getJSONArray("data");  
                    for (int i = 0; i < dataArray.size(); i++) {  
                        Movie movie = dataArray.getObject(i, Movie.class);  
                        Integer id = movie.getId();  
                        SearchResponse<Movie> searchResponse = elasticsearchClient
                            .search(builder -> builder.index(MOVIES_INDEX_NAME)  
                                .query(queryBuilder -> queryBuilder
                                    .constantScore(constantScoreQueryBuilder -> constantScoreQueryBuilder  
                                        .filter(filterQueryBuilder -> filterQueryBuilder
                                            .term(termQueryBuilder -> termQueryBuilder
                                                .field("id").value(id))))), Movie.class);  
                        HitsMetadata<Movie> hits = searchResponse.hits();  
                        TotalHits totalHits = hits.total();  
                        long value = 0;  
                        if (totalHits != null) {  
                            value = totalHits.value();  
                        }  
                        if (value == 0) {  
                            Snowflake snowflake = IdUtil.getSnowflake();  
                            elasticsearchClient.create(builder -> builder
                                .index(MOVIES_INDEX_NAME).id(snowflake.nextIdStr()).document(movie));  
                        } else {  
                            List<Hit<Movie>> hitList = hits.hits();  
                            for(Hit<Movie> hit: hitList) {  
                                Movie source = hit.source();  
                                if (source != null) {  
                                    movie.setCelebrityId(source.getCelebrityId());  
                                    movie.setCelebrityName(source.getCelebrityName());  
                                    movie.setMovieCelebrityType(source.getMovieCelebrityType());  
                                    movie.setCelebrityChineseName(source.getCelebrityChineseName());  
                                    String hitId = hit.id();  
                                    elasticsearchClient.update(builder -> builder
                                        .index(MOVIES_INDEX_NAME).id(hitId).doc(movie), Movie.class);  
                                }  
                            }  
                        }  
                    }  
                }  
                elasticsearchClient.shutdown();  
            }  
        }  
        channel.basicAck(deliveryTag, true);  
    }  
}
```