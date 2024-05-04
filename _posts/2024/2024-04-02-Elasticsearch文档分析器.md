---
title: Elasticsearch文档分析器
tags: [Elasticsearch] 
---

## 分析器测试

根据使用的文档分析器和测试的文本，给出分词结果

```
GET http://localhost:9200/_analyze
```

```json
{
    "analyzer": "standard",
    "text": "Text to analyze"
}
```

标准分词器`standard`，适用于英文分词，不适用于中文，对中文只进行单字切分
IK分词器`ik_max_word`，对中文进行细粒度的切分，会对结果再继续尝试切分，直到不能切分为止。例如“中华人民共和国国歌”将切分为“中华人民共和国”、“中华人民”、“中华”、“华人”、“人民共和国”、“人民”、“人”、“民”、“共和国”、“共和”、“和”、“国国”、“国歌”
IK分词器`ik_smart`，对中文进行粗粒度的切分，。例如“中华人民共和国国歌”将切分为“中华人民共和国”、“国歌”

## IK分词器安装

下载对应版本的IK分词器
https://release.infinilabs.com/
https://github.com/infinilabs/analysis-ik/releases

在Elasticsearch的插件文件夹`plugins`下新建一个`ik`文件夹，将下载下来的zip放到里面并解压，重启Elasticsearch后即可使用

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230522083552.png" width="800px" />

当找不到Elasticsearch对应版本的IK分词器时，可修改IK分词器插件的`plugin-descriptor.properties`文件中的对应版本

```properties
elasticsearch.version=7.17.9
```

## IK分词器扩展词汇

修改`plugins/ik/config/IKAnalyzer.cfg.xml`

```xml
<properties>
    <entry key="ext_dict">custom.dic</entry>
</properties>
```

创建`plugins/ik/config/custom.dic`，增加新词

## IK分词器配置使用

全局配置，修改`config/elasticsearch.yml`，增加一行

```yml
index.analysis.analyzer.default.type: ik
```

通过mapping指定

```
PUT http://localhost:9200/user/_mapping
```

```json
{
    "properties": {
        "name": {
            "type": "text",
            "index": true,
            "analyzer": "ik_max_word",
            "search_analyzer": "ik_max_word"
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