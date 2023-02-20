---
title: 20230219 学习springboot 调用 elasticsearch
tags:
  - elasticsearch
  - sprint boot
---



## [Spring Boot Starter Data Elasticsearch](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-elasticsearch) 

 

#### pom.xml

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-elasticsearch</artifactId>
	<version>3.0.2</version>
</dependency>
 
```

#### application.yml

```
spring: 
  data: 
    elasticsearch: 
     cluster-name: es-cluster
     # Comma-separated list of cluster node addresses.
     cluster-nodes: 10.133.28.55:9300
     # Whether to enable Elasticsearch repositories.
     repositories:
       enabled: true   
   
   
```



#### 创建Book实体类

创建一个Book实体类，用于同步ElasticSearch数据库的索引信息。

```java
/**
 *Document:
 * indexName = "book":索引名称,类似于mysql中的数据库;
 * type = "doc":文档类型,类似于mysql中的表;
 * shards = 5:分片数量
 * replicas = 1:副本
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
@Document(indexName = "book", type = "doc", shards = 5, replicas = 1)
public class Book {

    @Id
    private String id;

    /**
     * 中文分词设置，前提是您的es已经安装了中文分词ik插件
     * 中文分词有两种形式：
     * ik_max_word：会将文本做最细粒度的拆分
     * ik_smart：会将文本做最粗粒度的拆分
     */
    @Field(type = FieldType.Text, analyzer = "ik_max_word",searchAnalyzer ="ik_max_word")
    private String title;

    private String author;

    private String postDate;

}
```

####  创建Repository仓库类

创建一个ElasticsearchRepository接口子类，利用该类进行数据库操作。

```java
public interface BookRepository extends ElasticsearchRepository<Book, String> {

    //Optional<Book> findById(String id);

    Page<Book> findByAuthor(String author, Pageable pageable);
    
}
```

#### 定义Service及其实现类

在service层封装实现几个数据库的增删改查方法。

##### BookService接口

定义BookService接口，封装对索引的CRUD方法。

```java
public interface BookService {

    Optional<Book> findById(String id);

    Book save(Book blog);

    void delete(Book blog);

    Optional<Book> findOne(String id);

    List<Book> findAll();

    Page<Book> findByAuthor(String author, PageRequest pageRequest);

}
```

##### BookServiceImpl实现类

实现上面封装的方法，对索引库进行操作。

```java
@Slf4j
@Service
public class BookServiceImpl implements BookService {

    @Autowired
    private BookRepository bookRepository;

    @Override
    public Optional<Book> findById(String id) {
        return bookRepository.findById(id);
    }

    @Override
    public Book save(Book blog) {
        return bookRepository.save(blog);
    }

    @Override
    public void delete(Book blog) {
        bookRepository.delete(blog);
    }

    @Override
    public Optional<Book> findOne(String id) {
        return bookRepository.findById(id);
    }

    @Override
    public List<Book> findAll() {
        return (List<Book>) bookRepository.findAll();
    }

    @Override
    public Page<Book> findByAuthor(String author, PageRequest pageRequest) {
        return bookRepository.findByAuthor(author, pageRequest);
    }

}
```

##### **创建BookController接口**

创建Controller，定义几个URL接口用于测试调用。

```java
@Slf4j
@RestController
@RequestMapping("/book")
public class ElasticController {

    @Autowired
    private BookService bookService;

    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;

    /**
     * 创建索引
     */
    @PostMapping("/create")
    public void createIndex() {
        elasticsearchTemplate.createIndex(Book.class);
    }

    @PostMapping("/save")
    public void Save() {
        Book book = new Book("100", "ElasticSearch第2种实现方式", "一一哥", "2020-04-13");
        log.warn(book.toString());
        bookService.save(book);
    }

    @GetMapping("/id/{id}")
    public Book getBookById(@PathVariable String id) {
        Optional<Book> opt = bookService.findById(id);
        Book book = opt.get();
        log.warn(book.toString());
        return book;
    }

}
```









#### 新增单条文档 用elaticsearchTemplate 做插入

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class ESTest {
    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;
    @Test
    public void insertItemDoc(){
        Item item = new Item(1001L,"XXX1","XXX1","XXX1");
        IndexQuery indexQuery = new IndexQueryBuilder().withObject(item).build();
        elasticsearchTemplate.index(indexQuery);
    }
}
 
```

#### 

# Elastic实战：spring-data-elasticsearch聚合查询指南

 elasticsearch支持各种类型的聚合查询，给我们做数据统计、数据分析时提供了强大的处理能力，但是在elastic、spring-data官方文档中都没有详细说明聚合查询在java client中如何实现。

[es官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/7.13/search-aggregations.html)

索引结构

```json
# 订单索引，一个订单下有多个商品
PUT order_test
{
  "mappings": {
    "properties": {
      // 订单状态 0未付款 1未发货 2运输中 3待签收 4已签收 
      "status": {
        "type": "integer"
      },
      // 订单编号
      "no": {
        "type": "keyword"
      },
      // 下单时间
      "create_time": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss"
      },
      // 订单金额
      "amount": {
        "type": "double"
      },
      // 创建人
      "creator":{
        "type": "keyword"
      },
      // 商品信息
      "product":{
        "type": "nested",
        "properties": {
          // 商品ID
          "id": {
            "type": "keyword"
          },
          // 商品名称
          "name":{
            "type": "keyword"
          },
          // 商品价格
          "price": {
            "type": "double"
          },
          // 商品数量
          "quantity": {
            "type": "integer"
          }
        }
      }
    }
  }
}


```



 测试数据

```json
POST order_test/_bulk
{"index":{}}
{"status":0,"no":"DD202205280001","create_time":"2022-05-01 12:00:00","amount":100.0,"creator":"张三","product":[{"id":"1","name":"苹果","price":20.0,"quantity":5}]}
{"index":{}}
{"status":0,"no":"DD202205280002","create_time":"2022-05-01 12:00:00","amount":100.0,"creator":"李四","product":[{"id":"2","name":"香蕉","price":20.0,"quantity":5}]}
{"index":{}}
{"status":1,"no":"DD202205280003","create_time":"2022-05-02 12:00:00","amount":100.0,"creator":"张三","product":[{"id":"2","name":"香蕉","price":20.0,"quantity":5}]}
{"index":{}}
{"status":2,"no":"DD202205280004","create_time":"2022-05-01 12:00:00","amount":150.0,"creator":"王二","product":[{"id":"1","name":"苹果","price":30.0,"quantity":5}]}
{"index":{}}
{"status":2,"no":"DD202205280005","create_time":"2022-05-03 12:00:00","amount":100.0,"creator":"55555","product":[{"id":"2","name":"香蕉","price":20.0,"quantity":5}]}
{"index":{}}
{"status":3,"no":"DD202205280006","create_time":"2022-05-04 12:00:00","amount":150.0,"creator":"李四","product":[{"id":"3","name":"榴莲","price":150.0,"quantity":1}]}
{"index":{}}
{"status":4,"no":"DD202205280007","create_time":"2022-05-04 12:00:00","amount":100.0,"creator":"张三","product":[{"id":"2","name":"香蕉","price":20.0,"quantity":5}]}
{"index":{}}
{"status":3,"no":"DD202205280008","create_time":"2022-05-01 12:00:00","amount":200.0,"creator":"王二","product":[{"id":"1","name":"苹果","price":40.0,"quantity":5}]}
{"index":{}}
{"status":4,"no":"DD202205280009","create_time":"2022-05-03 12:00:00","amount":100.0,"creator":"55555","product":[{"id":"2","name":"香蕉","price":20.0,"quantity":5}]}
 
```

# 分桶聚合 Bucket aggregations

## Terms aggregation

分桶聚合中最常用的就是terms聚合了，它可以按照指定字段将数据分组聚合，类似mysql中的group by

 

- 案例

> 要求统计各种状态的单数



- DSL

```bash
GET order_test/_search
{
  "size": 0,
  "aggs": {
    "status_bucket": {
      "terms": {
        "field": "status"
      }
    }
  }
}

{
  "took" : 356,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 9,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "status_bucket" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : 0,
          "doc_count" : 2
        },
        {
          "key" : 2,
          "doc_count" : 2
        },
        {
          "key" : 3,
          "doc_count" : 2
        },
        {
          "key" : 4,
          "doc_count" : 2
        },
        {
          "key" : 1,
          "doc_count" : 1
        }
      ]
    }
  }
}

```

 

- ava实现

```java
	public void termsAgg(){
        String aggName = "status_bucket";
        NativeSearchQueryBuilder queryBuilder =  new NativeSearchQueryBuilder();
        queryBuilder.withPageable(PageRequest.of(0,1));
        TermsAggregationBuilder termsAgg = AggregationBuilders.terms(aggName).field("status");
        queryBuilder.addAggregation(termsAgg);
        Aggregations aggregations = restTemplate.query(queryBuilder.build(), SearchResponse::getAggregations);
        Terms terms = aggregations.get(aggName);
        List<? extends Terms.Bucket> buckets = terms.getBuckets();
        HashMap<String,Long> statusRes = new HashMap<>();
        buckets.forEach(bucket -> {
            statusRes.put(bucket.getKeyAsString(),bucket.getDocCount());
        });
        System.out.println("---聚合结果---");
        System.out.println(statusRes);
    }
```

 

## Date histogram aggregation

日期分组聚合可以按照日期进行分组，常用到一些日期趋势统计中

- 案例

> 统计每天的下单量

- DSL

```java
GET order_test/_search
{
  "size": 0,
  "aggs": {
    "date": {
      "date_histogram": {
        "field": "create_time",
        "calendar_interval": "day",
        "format": "yyyy-MM-dd"
      }
    }
  }
}

{
  "took" : 25,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 9,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "date" : {
      "buckets" : [
        {
          "key_as_string" : "2022-05-01",
          "key" : 1651363200000,
          "doc_count" : 4
        },
        {
          "key_as_string" : "2022-05-02",
          "key" : 1651449600000,
          "doc_count" : 1
        },
        {
          "key_as_string" : "2022-05-03",
          "key" : 1651536000000,
          "doc_count" : 2
        },
        {
          "key_as_string" : "2022-05-04",
          "key" : 1651622400000,
          "doc_count" : 2
        }
      ]
    }
  }
}
```



- java

```java
	public void dateHistogramAgg(){
        String aggName = "date";
        DateHistogramAggregationBuilder dateHistogramAggregation = AggregationBuilders.dateHistogram(aggName).field("create_time")
                .calendarInterval(DateHistogramInterval.days(1)).format("yyyy-MM-dd");
        NativeSearchQueryBuilder queryBuilder =  new NativeSearchQueryBuilder();
        queryBuilder.withPageable(PageRequest.of(0,1)).addAggregation(dateHistogramAggregation);
        Aggregations aggregations = restTemplate.query(queryBuilder.build(), SearchResponse::getAggregations);
        ParsedDateHistogram terms = aggregations.get(aggName);
        List<? extends Histogram.Bucket> buckets = terms.getBuckets();
        HashMap<String,Long> resultMap = new HashMap<>();
        buckets.forEach(bucket -> {
            resultMap.put(bucket.getKeyAsString(),bucket.getDocCount());
        });
        System.out.println("---聚合结果---");
        System.out.println(resultMap);
    }
```

### 拓展：

这里大家会发现使用的是`ParsedDateHistogram`来承接结果，与上述的`Term`不一致，那么我们怎么知道什么时候该用哪个呢？实际上可以通过断点来判断

我们通过把断点截取到`restTemplate.query`的执行结果`aggregations`之后，会发现该aggregations中的元素已经标明了其类型为`ParsedDateHistogram`，所以大家只需要跟着用就可以了。



## Range aggregation

范围分组聚合可以帮助我们按照指定的数值范围进行分组

- 案例

> 统计订单金额在0～100，100～200，200+ 这几个区间的订单数量

- DSL

 

```json
GET order_test/_search
{
  "size": 0,
  "aggs": {
    "date_range": {
      "range": {
        "field": "amount",
        "ranges": [
          {
            "to": "100"
          },
          {
            "from": "100",
            "to": "200"
          },
          {
            "from": "200"
          }
        ]
      }
    }
  }
}



{
  "took" : 7,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 9,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "date_range" : {
      "buckets" : [
        {
          "key" : "*-100.0",
          "to" : 100.0,
          "doc_count" : 0
        },
        {
          "key" : "100.0-200.0",
          "from" : 100.0,
          "to" : 200.0,
          "doc_count" : 8
        },
        {
          "key" : "200.0-*",
          "from" : 200.0,
          "doc_count" : 1
        }
      ]
    }
  }
}

```

- java

```java
 	public void rangeAgg(){
        String aggName = "range";
        RangeAggregationBuilder agg = AggregationBuilders.range(aggName).field("amount").addUnboundedTo(100).addRange(100, 200).addUnboundedFrom(200);
        NativeSearchQueryBuilder queryBuilder =  new NativeSearchQueryBuilder();
        queryBuilder.withPageable(PageRequest.of(0,1)).addAggregation(agg);
        Aggregations aggregations = restTemplate.query(queryBuilder.build(), SearchResponse::getAggregations);
        ParsedRange terms = aggregations.get(aggName);
        List<? extends Range.Bucket> buckets = terms.getBuckets();
        HashMap<String,Long> resultMap = new HashMap<>();
        buckets.forEach(bucket -> {
            resultMap.put(bucket.getKeyAsString(),bucket.getDocCount());
        });
        System.out.println("---聚合结果---");
        System.out.println(resultMap);
    }

```



## Nested aggregation

nested聚合专用于json型子对象进行聚合，比如上述案例中product是json型数组，如果当我们想通过商品中的属性来聚合统计时就需要用到nested聚合，直接使用`product.name`来聚合其结果则不会是我们预期的，这主要与es针对数组的存储形式有关。

 

- 案例

> 统计每种货物的订单数

- DSL

```json
GET order_test/_search
{
  "size": 0,
  "aggs": {
    "product_nested": {
      "nested": {
        "path": "product"
      },
      "aggs": {
        "name_bucket": {
          "terms": {
            "field": "product.name"
          }
        }
      }
    }
  }
}

{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 9,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "product_nested" : {
      "doc_count" : 9,
      "name_bucket" : {
        "doc_count_error_upper_bound" : 0,
        "sum_other_doc_count" : 0,
        "buckets" : [
          {
            "key" : "香蕉",
            "doc_count" : 5
          },
          {
            "key" : "苹果",
            "doc_count" : 3
          },
          {
            "key" : "榴莲",
            "doc_count" : 1
          }
        ]
      }
    }
  }
}


```

- java：这里我们涉及到要设置一个嵌套聚合，可以通过`subAggregation`方法来定义子聚合

```java
 	public void nestedAgg(){
        String aggName = "product_nested";
        String termsAggName = "name_bucket";
        NestedAggregationBuilder aggregationBuilder = AggregationBuilders.nested(aggName, "product").subAggregation(AggregationBuilders.terms(termsAggName).field("product.name"));
        NativeSearchQueryBuilder queryBuilder =  new NativeSearchQueryBuilder()
                .withPageable(PageRequest.of(0,1))
                .addAggregation(aggregationBuilder);
        Aggregations aggregations = restTemplate.query(queryBuilder.build(), SearchResponse::getAggregations);
        ParsedNested nestedRes = aggregations.get(aggName);
        Terms terms = nestedRes.getAggregations().get(termsAggName);
        List<? extends Terms.Bucket> buckets = terms.getBuckets();
        HashMap<String,Long> resMap = new HashMap<>();
        buckets.forEach(bucket -> {
            resMap.put(bucket.getKeyAsString(),bucket.getDocCount());
        });
        System.out.println("---聚合结果---");
        System.out.println(resMap);
    }

```

# 数值聚合 Metrics aggregations

## 3.1 Sum aggregations

求和聚合是常用的聚合之一，经常与分组聚合配合使用，用来统计出各组下的合计

- 案例

> 求5月1日销售总额

 

- DSL：这里我们添加了一个query语句，用来限定聚合范围是5.1日的订单

```json
GET order_test/_search
{
  "query": {
    "range": {
      "create_time": {
        "format": "yyyy-MM-dd",
        "from": "2022-05-01",
        "to": "2022-05-01"
      }
    }
  }, 
  "size": 0,
  "aggs": {
    "sum_amount": {
      "sum": {
        "field": "amount"
      }
    }
  }
}

{
  "took" : 6,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "sum_amount" : {
      "value" : 550.0
    }
  }
}

```



- java

```java
public void sumAgg(){
        String aggName = "sumAmount";
        SumAggregationBuilder agg = AggregationBuilders.sum(aggName).field("amount");
        NativeSearchQueryBuilder queryBuilder =  new NativeSearchQueryBuilder()
                .withPageable(PageRequest.of(0,1))
                .withQuery(QueryBuilders.rangeQuery("create_time").format("yyyy-MM-dd").from("2022-05-01").to("2022-05-01"))
                .addAggregation(agg);
        Aggregations aggregations = restTemplate.query(queryBuilder.build(), SearchResponse::getAggregations);
        ParsedSum metric = aggregations.get(aggName);
        double value = metric.getValue();
        System.out.println("---聚合结果---");
        System.out.println(value);
    }
```

## Script aggregation

脚本聚合支持我们通过脚本语言来自定义聚合的数值，es中脚本默认的语言为painless。需要注意的是脚本语法非常影响性能，我们一般是尽量避免使用。同时es中还提供了专门的[脚本数值聚合 script metric aggregation](https://link.juejin.cn/?target=https%3A%2F%2Fwww.elastic.co%2Fguide%2Fen%2Felasticsearch%2Freference%2F7.13%2Fsearch-aggregations-metrics-scripted-metric-aggregation.html)，但因为不太常用，所以我们这里以更加常用的聚合脚本来讲解

- 案例

> 求所有货物平均单价

- DSL：注意这里不能直接用product.price。因为product是数组，里面可能包含多种货物，所以应该用订单总金额除以所有订单的货物数量

```json
GET order_test/_search
{
  "size": 0,
  "aggs": {
    "total_amount":{
      "sum": {
        "field": "amount"
      }
    },
    "total_quantity":{
      "sum": {
        "script": {
          "source": """
            int total = 0;
            for(int i=0; i<params._source['product'].size(); i++){
              if(params._source['product'][i]['quantity'] != null){
                total += params._source['product'][i]['quantity'];
              }
            }
            return total;
          """
        }
      }
    }
  }
}

{
  "took" : 30,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 9,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "total_amount" : {
      "value" : 1100.0
    },
    "total_quantity" : {
      "value" : 41.0
    }
  }
}


```



 可以看到这里，原本sum聚合中是field属性的，改成了script脚本来动态计算属性值，从而实现聚合。同理，script脚本不仅可以使用到sum聚合中，也可以用到其他metric聚合中。将结果total_amount除以total_quantity即可得到平均价格

- java

```java
	public void scriptAgg(){
        String totalAmountAggName = "total_amount";
        String totalQuantityAggName = "total_quantity";
        SumAggregationBuilder amountAgg = AggregationBuilders.sum(totalAmountAggName).field("amount");
        SumAggregationBuilder quantityAgg = AggregationBuilders.sum(totalQuantityAggName).script(
                new Script("int total = 0;\n" +
                        "            for(int i=0; i<params._source['product'].size(); i++){\n" +
                        "              if(params._source['product'][i]['quantity'] != null){\n" +
                        "                total += params._source['product'][i]['quantity'];\n" +
                        "              }\n" +
                        "            }\n" +
                        "            return total;"));

        NativeSearchQueryBuilder queryBuilder =  new NativeSearchQueryBuilder()
                .withPageable(PageRequest.of(0,1))
                .addAggregation(amountAgg).addAggregation(quantityAgg);
        Aggregations aggregations = restTemplate.query(queryBuilder.build(), SearchResponse::getAggregations);
        ParsedSum amountRes = aggregations.get(totalAmountAggName);
        ParsedSum quantityRes = aggregations.get(totalQuantityAggName);
        double avgPrice = amountRes.getValue()/quantityRes.getValue();
        System.out.println("---聚合结果---");
        System.out.println(avgPrice);
    }

```

 
