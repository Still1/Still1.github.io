---
title: Elasticsearch基本使用
tags: [Elasticsearch]
---

## Elasticsearch与关系型数据库概念类比

* Index -> Database
* Type -> Table (7以上版本概念删除)
* Document -> Row
* Field -> Column

## Elasticsearch基础操作

### 创建Index

```
PUT http://localhost:9200/shopping
```

### 获取Index信息

```
GET http://localhost:9200/shopping
GET http://localhost:9200/_cat/indices?v
```

### 删除Index

```
DELETE http://localhost:9200/shopping
```

### 创建Document

```
POST http://localhost:9200/shopping/_doc
POST http://localhost:9200/shopping/_doc/1001
PUT http://localhost:9200/shopping/_doc/1001
PUT http://localhost:9200/shopping/_create/1001
```

```json
{
    "title": "小米手机",
    "category": "小米",
    "images": "http://www.guolixueyuan.com/xm.jpg",
    "price": 3999.00
}
```

 ### 查询Document

```
GET http://localhost:9200/shopping/_doc/1001
GET http://localhost:9200/shopping/_search
```

### 全量修改Document

```
PUT http://localhost:9200/shopping/_doc/1001
```

```json
{
    "title": "小米手机",
    "category": "小米",
    "images": "http://www.guolixueyuan.com/xm.jpg",
    "price": 4999.00
}
```

### 局部修改Document

```
POST http://localhost:9200/shopping/_update/1001
```

```json
{
    "doc": {
		"title": "华为手机"
    }
}
```

```
POST http://localhost:9200/shopping/_update/1001?if_seq_no=1&if_primary_term=1
```

### 删除Document

```
DELETE http://localhost:9200/shopping/_doc/1001
```

## Elasticsearch查询

### 条件查询

```
GET http://localhost:9200/shopping/_search?q=category:小米
```

```
GET http://localhost:9200/shopping/_search
```

```JSON
{
    "query": {
        "match": {
            "category": "小米"
        }
    }
}
```

`must`，`should`,  `filter`

```json
{
    "query": {
		"bool": {
            "must": [
                {
                    "match_phrase": {
                        "category": "小米"
                    }
                }，
                {
                	"match": {
                		"price": 1999.00
                	}
                }
            ],
    		"filter": {
    			"range": {
    				"price": {
    					"gt": 5000
					}
				}
			}
        }
    }
}
```

### 全量查询

```
GET http://localhost:9200/shopping/_search
```

```json
{
    "query": {
        "match_all": {}
    }
}
```

### 分页查询


```json
{
    "query": {
        "match_all": {}
    },
    "from": 0,
    "size": 2
}
```

### 指定查询Fields

```json
{
    "query": {
        "match_all": {}
    },
    "_source": ["title"]
}
```

### 排序

```json
{
    "query": {
        "match_all": {}
    },
    "sort": {
        "price": {
            "order": "desc"
        }
    }
}
```

### 高亮

```json
{
    "query": {
        "match_all": {}
    },
    "highlight": {
        "fields": {
            "category": {}
        }
    }
}
```

### 聚合

```json
{
    "aggs": {
        "price_group": {
            "terms" {
            	"field": "price
	        }
        }
    }
}
```

```json
{
    "aggs": {
        "price_avg": {
            "avg": {
                "field": "price"
            }
        }
    }
}
```

## Elasticsearch映射

### 创建映射

```
PUT http://localhost:9200/user/_mapping
```

```json
{
    "properties": {
        "name": {
            "type": "text",
            "index": true
        },
        "sex": {
            "type": "keyword",
            "index": true
        },
        "tel": {
            "type": "keyword",
            "index": false
        }
    }
}
```

### 查询映射

```
GET http://localhost:9200/user/_mapping
```