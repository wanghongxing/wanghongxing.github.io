---
title: 20230218 ElasticSearch 学习
date: 2023-02-19 10:38:10
tags:
  - elasticsearch
---





**ES主要解决问题：**
1）检索相关数据； 
2）返回统计结果； 
3）速度要快。

**ES特点和优势**
1）分布式实时文件存储，可将每一个字段存入索引，使其可以被检索到。 
2）实时分析的分布式搜索引擎。 
分布式：索引分拆成多个分片，每个分片可有零个或多个副本。集群中的每个数据节点都可承载一个或多个分片，并且协调和处理各种操作； 
负载再平衡和路由在大多数情况下自动完成。 
3）可以扩展到上百台服务器，处理PB级别的结构化或非结构化数据。也可以运行在单台PC上（已测试） 
4）支持插件机制，分词插件、同步插件、Hadoop插件、可视化插件等。



### ElasticSearch核心概念

- Cluster集群：一个集群由一个唯一的名字标识，默认为“elasticsearch”。
- Node 节点：存储集群的数据，参与集群的索引和搜索功能。
- Shard 分片：在创建一个索引时可以指定分成多少个分片来存储
- Index 索引: 一个索引是一个文档的集合（等同于solr中的集合）。
- Document 文档：被索引的一条数据，索引的基本信息单元，以JSON格式来表示。（就是数据）
- type类型：在一个索引中，你可以定义一种或多种类型。（本身就是个鸡肋，实际很少用到，6x后取消该概念，所以可以忽略）。
- mapping：映射像关系数据库中的表结构，每一个索引都有一个映射，它定义了索引中的每一个字段类型，以及一个索引范围内的设置。一个映射可以事先被定义，或者在第一次存储文档的时候自动识别。
- field：一个文档中包含零个或者多个字段，字段可以是一个简单的值（例如字符串、整数、日期），也可以是一个数组或对象的嵌套结构。字段类似于关系数据库中的表中的列。每个字段都对应一个字段类型，例如整数、字符串、对象等。字段还可以指定如何分析该字段的值。

#### Mapping简介

mapping 是用来定义文档及其字段的存储方式、索引方式的手段，例如利用mapping 来定义以下内容：

- 哪些字段需要被定义为全文检索类型
- 哪些字段包含number、date类型等
- 格式化时间格式
- 自定义规则，用于控制动态添加字段的映射

#### Mapping Type

每个索引都拥有唯一的 mapping type，用来决定文档将如何被索引。mapping type由下面两部分组成

Meta-fields
元字段用于自定义如何处理文档的相关元数据。 元字段的示例包括文档的_index，_type，_id和_source字段。

Fields or properties
映射类型包含与文档相关的字段或属性的列表。

#### 字段类型

- 一种简单的数据类型，例如text、keyword、double、boolean、long、date、ip类型。
- 也可以是一种分层的json对象（支持属性嵌套）。
- 也可以是一些不常用的特殊类型，例如geo_point、geo_shape、completion

针对同一字段支持多种字段类型可以更好地满足我们的搜索需求，例如一个string类型的字段可以设置为text来支持全文检索，与此同时也可以让这个字段拥有keyword类型来做排序和聚合，另外我们也可以为字段单独配置分词方式，例如"analyzer": "ik_max_word"。



##### text 类型

text类型的字段用来做全文检索，例如邮件的主题、淘宝京东中商品的描述等。这种字段在被索引存储前先进行分词，存储的是分词后的结果，而不是完整的字段。text字段不适合做排序和聚合。如果是一些结构化字段，分词后无意义的字段建议使用keyword类型，例如邮箱地址、主机名、商品标签等。

##### keyword 类型

keyword用于索引结构化内容（例如电子邮件地址，主机名，状态代码，邮政编码或标签）的字段，这些字段被拆分后不具有意义，所以在es中应索引完整的字段，而不是分词后的结果。

通常用于过滤（例如在博客中根据发布状态来查询所有已发布文章），排序和聚合。keyword只能按照字段精确搜索，例如根据文章id查询文章详情。如果想根据本字段进行全文检索相关词汇，可以使用text类型。

##### **date类型**

支持排序，且可以通过format字段对时间格式进行格式化。

json中没有时间类型，所以在es在规定可以是以下的形式：
一段格式化的字符串，例如"2015-01-01"或者"2015/01/01 12:10:30"
一段long类型的数字，指距某个时间的毫秒数，例如1420070400001
一段integer类型的数字，指距某个时间的秒数

##### **object类型**

mapping中不用特意指定field为object类型，因为这是它的默认类型。

json类型天生具有层级的概念，文档内部还可以包含object类型进行嵌套。

##### **nest类型**

nest类型是一种特殊的object类型，它允许object可以以数组形式被索引，而且数组中的某一项都可以被独立检索。

而且es中没有内部类的概念，而是通过简单的列表来实现nest效果，例如下列结构的文档：

```awk
PUT my_index/_doc/1
{
  "group" : "fans",
  "user" : [ 
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
}
```

上面格式的对象会被按照下列格式进行索引，因此会发现一个user中的两个属性值不再匹配，alice和white失去了联系

```java
{
  "group" :        "fans",
  "user.first" : [ "alice", "john" ],
  "user.last" :  [ "smith", "white" ]
}
```



##### **range类型**

*这个没搞明白干啥用？？？*

支持以下范围类型：

| 类型          | 范围                                 |
| ------------- | ------------------------------------ |
| integer_range | -2的31次 到 2的31次-1.               |
| float_range   | 32位单精度浮点数                     |
| long_range    | -2的63次 到 2的63次-1.               |
| double_range  | 64位双精度浮点数                     |
| date_range    | unsigned 64-bit integer milliseconds |
| ip_range      | ipv4和ipv6或者两者的混合             |

使用范例为：

```ada
PUT range_index
{
  "settings": {
    "number_of_shards": 2
  },
  "mappings": {
    "properties": {
      "age_range": {
        "type": "integer_range"
      },
      "time_frame": {
        "type": "date_range", 
        "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
      }
    }
  }
}

PUT range_index/_doc/1?refresh
{
  "age_range" : { 
    "gte" : 10,
    "lte" : 20
  },
  "time_frame" : { 
    "gte" : "2015-10-31 12:00:00", 
    "lte" : "2015-11-01"
  }
}
```



### 查看es相关信息

```bash
GET _cat/nodes     # 查看所有节点
GET _cat/health    # 查看es健康状况
GET _cat/master    # 查看主节点
GET _cat/indices   # 查看所有索引
GET _cat/indices?v # 带参数v, 更详细,显示title
GET _cluster/health # 查看集群健康信息
```

查看健康状态

```
$ curl  localhost:9200/_cat/health?v&pretty
epoch      timestamp cluster        status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1676774613 02:43:33  docker-cluster green           3         3     18   9    0    0        0             0                  -                100.0%
```

### 索引操作

```bash
PUT customer # 创建索引 使用PUT
DELETE customer # 删除索引 使用DELETE
GET customer # 查看单个索引 使用GET
```





**查看指定索引（my_index）的mapping和setting的相关信息**

GET /my_index?pretty

```json
$ curl localhost:9200/.tasks?pretty
{
  ".tasks" : {
    "aliases" : { },
    "mappings" : {
      "dynamic" : "strict",
      "_meta" : {
        "version" : "7.17.9"
      },
      "properties" : {
        "completed" : {
          "type" : "boolean"
        },
        "error" : {
          "type" : "object",
          "enabled" : false
        },
        "response" : {
          "type" : "object",
          "enabled" : false
        },
        "task" : {
          "properties" : {
            "action" : {
              "type" : "keyword"
            },
            "cancellable" : {
              "type" : "boolean"
            },
            "description" : {
              "type" : "text"
            },
            "headers" : {
              "type" : "object",
              "enabled" : false
            },
            "id" : {
              "type" : "long"
            },
            "node" : {
              "type" : "keyword"
            },
            "parent_task_id" : {
              "type" : "keyword"
            },
            "running_time_in_nanos" : {
              "type" : "long"
            },
            "start_time_in_millis" : {
              "type" : "long"
            },
            "status" : {
              "type" : "object",
              "enabled" : false
            },
            "type" : {
              "type" : "keyword"
            }
          }
        }
      }
    },
    "settings" : {
      "index" : {
        "routing" : {
          "allocation" : {
            "include" : {
              "_tier_preference" : "data_content"
            }
          }
        },
        "number_of_shards" : "1",
        "auto_expand_replicas" : "0-1",
        "provided_name" : ".tasks",
        "creation_date" : "1676774460157",
        "priority" : "2147483647",
        "number_of_replicas" : "1",
        "uuid" : "QYIany0aS-SX9fTy67pyKg",
        "version" : {
          "created" : "7170999"
        }
      }
    }
  }
}
```

**查看所有的index**

GET /_cat/indices?v&pretty

```
$ curl localhost:9200/_cat/indices?v
health status index                           uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .geoip_databases                aAgf55NkQV68krH2BmJaUA   1   1         41            0     79.3mb         39.6mb
green  open   .monitoring-es-7-2023.02.19     OwvlV2oUQJWe1f1S1tJaAg   1   1       3604            0     10.5mb          5.2mb
green  open   .apm-custom-link                0JPcMxKnQGOuoeQcV0D7wg   1   1          0            0       452b           226b
green  open   .kibana_7.17.9_001              rJo3F1WRQ7GDRAQaYHwFIw   1   1         19            1      9.4mb          4.7mb
green  open   .apm-agent-configuration        ZI2WKexKTYCTmpfoyfDSjA   1   1          0            0       452b           226b
green  open   .kibana_task_manager_7.17.9_001 LzRJwt89TVa0QiR4Gnbk7Q   1   1         17          111    902.5kb        372.9kb
green  open   .monitoring-kibana-7-2023.02.19 2Wdu56YJTBOm7draPIjp4w   1   1        414            0    821.4kb        323.9kb
green  open   .tasks                          QYIany0aS-SX9fTy67pyKg   1   1          6            0       70kb           35kb
```

**创建一个分片数3，没有复制分片名为my_index的索引**

PUT my_index { "settings": { "number_of_shards": 1, "number_of_replicas": 0 }, "mappings": { "properties": { "name": { "type": "text" }, "age": { "type": "integer" }, "mobile": { "type": "keyword" }, "context": { "type": "text" } } } }

**修改索引的设置**

PUT /my_index/_settings { "number_of_replicas": 1 }

**删除索引**

DELETE /my_index

**删除多个索引**

DELETE /index_one,index_two

DELETE /index_*

DELETE /_all



### **文档操作**

```bash
# 添加一个文档：
# 使用PUT添加，不指定id会报错，因为添加认为你准确知道添加哪个id的数据。
# 使用POST添加，可以不指定id，会自动生成，因此相同数据会认为是多条数据，因为id不同
# 重复发送会增加版本号，并覆盖原文档
PUT /customer/_doc/1
{
  "name":"john"
}
 
# 使用_create添加文档
# 准确表示在创建文档，只能创建一次，PUT POST都可以，多次创建会失败
PUT /customer/_create/1
{
  "name":"john"
}
## 总结：_create用于创建，只能创建一次，PUT、POST都可以。_doc表示文档，PUT必须指定id才能添加，POST会随机生成id。如果索引没有会自动创建
 
# 删除一个文档
DELETE customer/_doc/5
 
# 查看一个文档
GET customer/_doc/6
# 结果如下，带_的都是元信息
{
  "_index" : "customer",    # 在哪个索引
  "_type" : "_doc",         # 在哪个类型
  "_id" : "6",              # id
  "_version" : 1,           # 版本号
  "_seq_no" : 27,           # 序列号,用于并发控制，每次更新就会加1，用来做乐观锁
  "_primary_term" : 1,      # 同上，主分片重新分配，如重启就会变化
  "found" : true,
  "_source" : {             # 实际存储的数据
    "name" : "john"
  }
}
 
# 使用乐观锁修改文档，更新时携带?if_seq_no=0&if_primary_term=1 当满足条件时修改数据，否则不修改
PUT /customer/_doc/6?if_seq_no=27&if_primary_term=1 # PUT POST + _doc是覆盖，修改使用POST + _update
{
  "name2":"alice"
}
 
# 更新一个文档
# 对比原数据，没有变化则不更新，noop，如果使用_doc直接创建新文档，会直接覆盖
# _update只允许使用POST，"doc"表示文档，是内置字段，原数据会增加age字段
POST customer/_update/6/
{
  "doc":{
    "age":18
  }
}
# 可以直接添加一个新文档，直接覆盖，根据实际情况确定
# 总结：大并发偶尔更新的使用_update更新，重新计算分配规则，大并发更新较多的直接覆盖
```



#### **创建文档**

POST /my_index/_doc/f2daf45df25sa5fd4d58 { "name":"李强", "age":28 }

注：id可加可不加，不加的话es自动生成该文档的id

#### **修改文档**

两种写法

POST /my_index/_doc/f2daf45df25sa5fd4d58 { "name":"李强", "age":21 }

POST /my_index/_update/f2daf45df25sa5fd4d58 { "doc":{ "name":"李强", "age":28 } }

注：id必填。第二种_update请求体中需要将字段包type类型（doc）中。type类型名字可以在创建索引时指定，默认_doc。es7版本虽然已废弃，但还在使用

#### **删除文档**

DELETE /my_index/_doc/{id}



## 进阶检索

### Search API

- es支持的两种基本方式：
  - 通过REST request URI 发送搜索参数（URI + 请求参数）
  - 通过REST request body 来发送（URI + 请求体）

```bash
# 通过request uri检索 q=* 等价于query match_all
GET bank/_search?q=*&sort=account_number:desc
 
# 通过request body检索，请求体为Query DSL
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "account_number": {    # 也可以简写为"account_number": "desc"
        "order": "desc"
      }
    }
  ]
}
# 默认返回10条数据，在hits.hits数组里，_source里是存储的数据
```



#### **查询文档**

##### **match query**

match query 用于搜索单个字段，首先会针对查询语句进行解析，主要是对查询语句进行分词，分词后查询语句的任何一个词项被匹配，文档就会被搜到，默认情况下相当于对分词后词项进行 or 匹配操作。

GET ip:/my_index/_search { "query": { "match": { "name": { "query": "" } } } }

如果想查询匹配到全部关键词的文档，可以用 and 操作符连接，如下：

GET my_index/_search { "query": { "match": { "name": { "query": "", "operator": "and" } } } }

##### **multi_match**

multi_match 是 match 的升级，用于搜索多个字段。查询语句为“”。field可以定义字段及权重

GET my_index/_search { "query": { "multi_match": { "query": "", "fields": ["context^3", "name"] } } }

##### **term query**

term 查询用来查找指定字段中包含给定单词的文档，term 查询不被解析，只有查询词和文档中的词精确匹配才会被搜索到

GET my_index/_search { "query": { "term": { "mobile": “" } } }

##### **terms query**

terms 查询是 term 查询的升级，可以用来查询文档中包含多个词的文档。相当于sql中的in

{ "query": { "terms": { "mobile": ["", ""] } } }



##### **range query**

即范围查询，用于匹配在某一范围内的数值型、日期类型或者字符串型字段的文档

range 查询支持的参数有以下几种：

gt 大于，查询范围的最小值，也就是下界，但是不包含临界值。

gte 大于等于，和 gt 的区别在于包含临界值。

lt 小于，查询范围的最大值，也就是上界，但是不包含临界值。

lte 小于等于，和 lt 的区别在于包含临界值。

例如：查询年龄23到25

GET bookes/_search { "query": { "range": { "age": { "gt": 23, "lte": 25 } } } }

##### **bool query**

bool 查询可以把任意多个简单查询组合在一起，使用 must、should、must_not、filter 选项来表示简单查询之间的逻辑，它们的含义如下：

must 文档必须匹配 must 选项下的查询条件，相当于逻辑运算的 AND，且参与文档相关度的评分。

should 文档可以匹配 should 选项下的查询条件也可以不匹配，相当于逻辑运算的 OR，且参与文档相关度的评分。

must_not 与 must 相反，匹配该选项下的查询条件的文档不会被返回；需要注意的是，must_not 语句不会影响评分，它的作用只是将不相关的文档排除。

filter 和 must 一样，匹配 filter 选项下的查询条件的文档才会被返回，但是 filter 不评分，只起到过滤功能

例子：查询名字中包含”强“。且年龄不大于25

GET my_index/_search { "query": {
"bool": { "must_not": { "range": { "age": { "lte": 25 } } }, "must": { "match": { "name": "强" } } } } }



##### **聚合查询**

terms agges 类似于group by

例:查询数据中每个年龄的值及每个年龄数量

GET my_index/_search { "size": 0, "aggs": { "job_info": { "terms": { "field": "job" } } } }







### ElasticSearch 常用的查询过滤语句

**查询与过滤的区别？**
查询会计算文档匹配程度的_score，过滤则不会，所以过滤效率会更高

### Filter DSL

#### **term 过滤**

term主要用于精确匹配哪些值，比如数字，日期，布尔值或 not_analyzed 的字符串(未经分析的文本数据类型)：

```json
{ "term": { "age":    26           }}
{ "term": { "date":   "2014-09-01" }}
{ "term": { "public": true         }}
{ "term": { "tag":    "full_text"  }}
```

完整的例子， hostname 字段完全匹配成 saaap.wangpos.com 的数据：

```json
{
  "query": {
    "term": {
      "hostname": "saaap.wangpos.com"
    }
  }
}
```

#### **terms 过滤**

terms 跟 term 有点类似，但 terms 允许指定多个匹配条件。 如果某个字段指定了多个值，那么文档需要一起去做匹配：

```prolog
{
    "terms": {
        "tag": [ "search", "full_text", "nosql" ]
        }
}
```

完整的例子，所有http的状态是 302 、304 的， 由于ES中状态是数字类型的字段，所有这里我们可以直接这么写。：

```autohotkey
{
  "query": {
    "terms": {
      "status": [
        304,
        302
      ]
    }
  }
}
```

#### **range 过滤**

range过滤允许我们按照指定范围查找一批数据：

```json
{
    "range": {
        "age": {
            "gte":  20,
            "lt":   30
        }
    }
}
```

范围操作符包含：

- gt :: 大于
- gte:: 大于等于
- lt :: 小于
- lte:: 小于等于

一个完整的例子， 请求页面耗时大于1秒的数据，upstream_response_time 是 nginx 日志中的耗时，ES中是数字类型。

```json
{
  "query": {
    "range": {
      "upstream_response_time": {
        "gt": 1
      }
    }
  }
}
```



#### **exists 和 missing 过滤**

exists 和 missing 过滤可以用于查找文档中是否包含指定字段或没有某个字段，类似于SQL语句中的IS_NULL条件.

```json
{
    "exists":   {
        "field":    "title"
    }
}
```

这两个过滤只是针对已经查出一批数据来，但是想区分出某个字段是否存在的时候使用。



#### **bool 过滤**

bool 过滤可以用来合并多个过滤条件查询结果的布尔逻辑，它包含一下操作符：

- must :: 多个查询条件的完全匹配,相当于 and。
- must_not :: 多个查询条件的相反匹配，相当于 not。
- should :: 至少有一个查询条件匹配, 相当于 or。

这些参数可以分别继承一个过滤条件或者一个过滤条件的数组：

```json
{
    "bool": {
        "must":     { "term": { "folder": "inbox" }},
        "must_not": { "term": { "tag":    "spam"  }},
        "should": [
                    { "term": { "starred": true   }},
                    { "term": { "unread":  true   }}
        ]
    }
}
```

### Query DSL

- 基本语法

```bash
# ES提供的可以查询的json风格的DSL(Domain Specific language 领域特定语言)，称为Query DSL.该查询语言非常全面，并且刚开始的时候有些复杂，真正学好它的方法是从一些基础示例开始的。
# 典型结构
{
    query_name:{    # 根操作，要做什么，对于某个字段也是同样的结构
        argument:value,
        argument:value,...
    }
}
```

- 基本使用

```bash
GET bank/_search
query # 定义如何查询
    match_all       # 代表查询所有 等价于xxx/_search?q=*
    match           # 【模糊匹配】分词匹配，全文检索按照评分进行排序，keyword表示匹配字段整个内容，全文检索使用match，非text使用term，text类型才有keyword子属性
    term            # 【精确匹配】比如整数值，文本字段使用match，文本字段进行了分析，使用term检索非常困难
    match_phrase    # 【短语匹配】不进行分词，对整个短语进行匹配
    multi_match     # 【多个字段匹配】在某些字段里面匹配
    bool     # 【复合查询】，可以组合多个查询
        must     # 【必须满足】这几个查询都可以使用range指定范围lte,gte,lt,gt，贡献相关性得分
        must_not # 【必须不满足】
        should   # 【应该满足】
        filter   # 【过滤】不贡献相关性得分
sort    # 【排序】，会在前字段相等时，后字段内部排序，跟数组
from    # 【从第几条开始】偏移量
size    # 【限定返回结果数量】完成分页，默认返回10，设置为0不显示数据
_source # 【返回部分字段】指定返回哪些字段
aggs    # 【对匹配结果聚合】
    terms    # 【字段频次】size指定频次结果的前几个
    avg      # 【字段平均值】
```



#### **match_all 查询**

可以查询到所有文档，是没有查询条件下的默认语句。

```json
GET my_index/_search
{
    "query": {
      "match_all": {
      }
    }
}
```

此查询常用于合并过滤条件。 比如说你需要检索所有的邮箱,所有的文档相关性都是相同的，所以得到的_score为1.

#### **match 查询**

match查询是一个标准查询，不管你需要全文本查询还是精确查询基本上都要用到它。

如果你使用 match 查询一个全文本字段，它会在真正查询之前用分析器先分析match一下查询字符：

```json
{
    "match": {
        "tweet": "About Search"
    }
}
```

如果用match下指定了一个确切值，在遇到数字，日期，布尔值或者not_analyzed 的字符串时，它将为你搜索你给定的值：

```json
{ "match": { "age":    26           }}
{ "match": { "date":   "2014-09-01" }}
{ "match": { "public": true         }}
{ "match": { "tag":    "full_text"  }}
```

提示： 做精确匹配搜索时，你最好用过滤语句，因为过滤语句可以缓存数据。

match查询只能就指定某个确切字段某个确切的值进行搜索，而你要做的就是为它指定正确的字段名以避免语法错误。

#### **multi_match 查询**

multi_match查询允许你做match查询的基础上同时搜索多个字段，在多个字段中同时查一个：

```prolog
{
    "multi_match": {
        "query":    "full text search",
        "fields":   [ "title", "body" ]
    }
}
```

#### **bool 查询**

bool 查询与 bool 过滤相似，用于合并多个查询子句。不同的是，bool 过滤可以直接给出是否匹配成功， 而bool 查询要计算每一个查询子句的 _score （相关性分值）。

- must:: 查询指定文档一定要被包含。
- must_not:: 查询指定文档一定不要被包含。
- should:: 查询指定文档，有则可以为文档相关性加分。

以下查询将会找到 title 字段中包含 "how to make millions"，并且 "tag" 字段没有被标为 spam。 如果有标识为 "starred" 或者发布日期为2014年之前，那么这些匹配的文档将比同类网站等级高：

```prolog
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }},
            { "range": { "date": { "gte": "2014-01-01" }}}
        ]
    }
}
```

提示： 如果bool 查询下没有must子句，那至少应该有一个should子句。但是 如果有must子句，那么没有should子句也可以进行查询。



#### **短语匹配(Phrase Matching)**

当你需要寻找邻近的几个单词时，你会使用match_phrase查询：

```awk
GET /my_index/my_type/_search
{
    "query": {
        "match_phrase": {
            "title": "quick brown fox"
        }
    }
}
```

和match查询类似，match_phrase查询首先解析查询字符串来产生一个词条列表。然后会搜索所有的词条，但只保留含有了所有搜索词条的文档，并且词条的位置要邻接。一个针对短语quick fox的查询不会匹配。
我们的任何文档，因为没有文档含有邻接在一起的quick和box词条。
match_phrase查询也可以写成类型为phrase的match查询：

```json
"match": {
    "title": {
        "query": "quick brown fox",
        "type":  "phrase"
    }
}
```





# Elasticsearch优化

硬件：

- 使用ssd硬盘

分片策略：

- 合理设置分片，一个分片数据过多，它的查询效率会降低，分片在创建索引时就确定了，不可更改（查询数据会直接定位到在哪个分片上），副本数可以随时更改
  - 分片也不能过多，一个分片就是一个Lucene索引，会消耗一定的句柄、内存、cpu，每一个搜索请求都要命中每一个分片，如果分片处于不同的节点还好，如果多个分片都在同一个节点上，那么就会竞争使用相同的资源
  - 用于计算相关度的词频统计信息是基于分片的，如果分片过多，每个分片都只有很少的数据会导致很低的相关度
  - 一般分片数量不超过节点数量的三倍。主分片、副本和节点最大数可参考：`节点数<=主分片*(副本数-1)`
- 推迟分片分配

路由选择：

- 带路由搜索，直接去指定节点查询数据，而不用查询所有节点数据





