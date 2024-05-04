---
title: Elasticsearch客户端Java High Level REST Client使用
tags: [Elasticsearch]
---

## 前言

Java High Level REST Client在Elasticsearch 7.15版本后官方不再建议使用，在Elasticsearch 8.x版本中兼容模式下可以使用Java High Level REST Client的7.17版本

Elasticsearch 7.15版本以后，官方推荐使用Java API Client

## 基础API

### 客户端连接

```java
RestHighLevelClient esClient = new RestHighLevelClient(RestClient.builder(new HttpHost("localhost", 9200, "http")));
esClient.close();
```

### 创建Index

```java
CreateIndexRequest request = new CreateIndexRequest("user");
CreateIndexResponse response = esClient.indices().create(request, RequestOptions.DEFAULT);
response.isAcknowledged();
```

### 获取Index信息

```java
GetIndexRequest request = new GetIndexRequest("user");
GetIndexResponse response = esClient.indices().get(request, RequestOptions.DEFAULT);
response.getAliases();
response.getMappings();
response.getSettings();
```

### 删除Index

```java
DeleteIndexRequest request = new DeleteIndexRequest("user");
AcknowledgedResponse response = esClient.indices().delete(request, RequestOptions.DEFAULT);
response.isAcknowledged();
```

### 创建Document

```java
IndexRequest request = new IndexRequest();
request.index("user").id("1001");

User user = new User();
user.setName("zhangsan");
user.setAge(30);

ObjectMapper mapper = new ObjectMapper();
String json = mapper.writeValueAsString(user);
request.source(json, XContentType.JSON);

IndexResponse response = esClient.index(request, RequestOptions.DEFAULT);
response.getResult();
```

### 查询Document

```json
GetRequest request = new GetRequest();
request.index("user").id("1001");
GetRespnse response = esClient.get(request, RequestOptions.DEFAULT);

response.getSourceAsString();
```

### 局部修改Document

```json
UpdateRequest request = new UpdateRequest();
request.index("user").id("1001");
request.doc(XContentType.JSON, "name", "lisi");

UpdateResponse response = esClient.update(request, RequestOptions.DEFAULT);
```

### 删除Document

```json
DeleteRequest request = new DeleteRequest();
request.index("user").id("1001");

DeleteResonse response = esClient.delete(request, RequestOptions.DEFAULT);
```

### 批量操作

```java
BulkRequest request = new BulkRequest();
request.add(indexRequest);
request.add(deleteRequest);

BulkResponse response = esClient.bulk(request, RequestOptions.DEFAULT)；
response.getTook();
response.getItems();
```

## 查询API

### 条件查询

```java
SearchRequest request = new SearchRequest();
request.indices("user");
SearchSourceBuilder builder = new SearchSourceBuilder();
builder.query(QueryBuilders.termQuery("age", 30))
request.source(builder);
SearchResponse response = esClient.search(request, RequestOptions.DEFAULT);

response.getTook();
SearchHits hits = response.getHits();
hits.getTotalHits();

for(SearchHit hit : hits) {
    hit.getSourceAsString();
}
```

```java
SearchRequest request = new SearchRequest();
request.indices("user");
SearchSourceBuilder builder = new SearchSourceBuilder();
BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
boolQueryBuilder.must(QueryBuilders.matchQuery("age", 30));
boolQueryBuilder.should(QueryBuilders.matchQuery("sex", "男"));
boolQueryBuilder.should(QueryBuilders.matchQuery("sex", "女"));
```

```java
SearchRequest request = new SearchRequest();
request.indices("user");
SearchSourceBuilder builder = new SearchSourceBuilder();
RangeQueryBuilder rangeQueryBuilder = QueryBuilders.rangeQuery("age");
rangeQueryBuilder.gte(30);
rangeQueryBuilder.lte(40);
```

```java
SearchRequest request = new SearchRequest();
request.indices("user");
SearchSourceBuilder builder = new SearchSourceBuilder();
FuzzyQueryBuilder fuzzyQueryBuilder = QueryBuilders.fuzzyQuery("name", "wangwu").fuzziness(Fuzziness.ONE);
builder.query(fuzzyQueryBuilder)；
```

### 全量查询

```java
SearchRequest request = new SearchRequest();
request.indices("user");
SearchSourceBuilder builder = new SearchSourceBuilder();
builder.query(QueryBuilders.matchAllQuery())；
request.source(builder);
SearchResponse response = esClient.search(request, RequestOptions.DEFAULT);
```

### 分页查询

```java
SearchSourceBuilder builder = new SearchSourceBuilder();
builder.query(QueryBuilders.matchAllQuery());
builder.from(0);
builder.size(2);
request.source(builder);
SearchResponse response = esClient.search(request, RequestOptions.DEFAULT);
```

### 指定查询Fields

```java
SearchSourceBuilder builder = new SearchSourceBuilder();
builder.query(QueryBuilders.matchAllQuery());
String[] excludes = {};
String[] includes = {"name"};
builder.fetchSource(includes, excludes);
request.source(builder);
SearchResponse response = esClient.search(request, RequestOptions.DEFAULT);
```

### 排序

```java
SearchSourceBuilder builder = new SearchSourceBuilder();
builder.query(QueryBuilders.matchAllQuery());
builder.sort("age", SortOrder.DESC);
```

### 高亮

```java
SearchRequest request = new SearchRequest();
request.indices("user");
SearchSourceBuilder builder = new SearchSourceBuilder();
builder.query(QueryBuilders.termQuery("age", 30));
HighlightBuilder highlightBuilder = new HighlightBuilder();
highlightBuilder.field("name");
builder.hightlighter(highlightBuilder);
```

### 聚合

```java
SearchRequest request = new SearchRequest();
request.indices("user");
SearchSourceBuilder builder = new SearchSourceBuilder();
AggregationBuilder aggregationBuilder = AggregationBuilders.max("maxAge").field("age");
builder.aggregation(aggregationBuilder);
```

```java
SearchRequest request = new SearchRequest();
request.indices("user");
SearchSourceBuilder builder = new SearchSourceBuilder();
AggregationBuilder aggregationBuilder = AggregationBuilders.terms("ageGroup").field("age");
builder.aggregation(aggregationBuilder);
```

## Spring Data 集成

### 配置

```properties
elasticsearch.host=127.0.0.1
elasticsearch.port=9200
```

```java
@ConfigurationProperties(prefix = "elasticsearch")
@Configuration
@Data
public class ElasticSearchConfig extends AbstractElasticsearchConfiguraion {
    private String host;
    private Integer port;
    @Override
    public RestHighLevelClient elasticsearchClient() {
        RestClientBuilder builder = RestClient.builder(new HttpHost(host, port));
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(builder);
        return restHighLevelClient;
    }
}
```

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Document(indexName= "shopping", shards = 3, replicas = 1)
public class Product {
    @Id
    private Long id;
    @Field(type = FieldType.Text)
    private String title;
    @Field(type = FieldType.Keyword)
    private String category;
    @Field(type = FieldType.Double)
    private Double price;
    @Field(type = FieldType.Keyword, index = false)
    private String images;
}
```

```java
@Repository
public interface ProductDao extends ElasticsearchRepository<Product, Long> {}
```

### 操作Document

```java
productDao.save(product);
Product product = productDao.findById(1L).get();
Iterable<Product> products = productDao.findAll();
productDao.delete(product);
productDao.saveAll(products);

Sort sort = Sort.by(Sort.Direction.DESC, "id");
int from = 0;
int size = 5;
PageRequest pageRequest = PageRequest.of(from, size, sort);
Page<Product> productPage = productDao.findAll(pageRequest);
productPage.getContent();
```

### 查询Document

```java
TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("title", "小米");
productDao.search(termQueryBuilder);
```