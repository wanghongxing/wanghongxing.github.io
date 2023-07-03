install es doris run macbook 



 



参照 https://doris.apache.org/zh-CN/docs/install/construct-docker/construct-docker-image 构建docker 镜像。

然后使用docker-compose启动

```
version: '3'
services:
  docker-fe:
    image: "apache-doris:1.1.5-fe-arm"
    container_name: "doris-fe"
    hostname: "fe"
    environment:
      - FE_SERVERS=fe1:172.20.80.2:9010
      - FE_ID=1
    ports:
      - 8030:8030
      - 9030:9030
    volumes:
      - ./fe/doris-meta:/opt/apache-doris/fe/doris-meta
      - ./fe/conf:/opt/apache-doris/fe/conf
      - ./fe/log:/opt/apache-doris/fe/log
    networks:
      default:
        ipv4_address: 172.20.80.2
  docker-be:
    image: "apache-doris:1.1.5-be-arm"
    container_name: "doris-be"
    hostname: "be"
    depends_on:
      - docker-fe
    environment:
      - FE_SERVERS=fe1:172.20.80.2:9010
      - BE_ADDR=172.20.80.3:9050
    ports:
      - 8040:8040
    volumes:
      - ./be/storage:/opt/apache-doris/be/storage
      - ./be/conf:/opt/apache-doris/be/conf
      - ./be/script:/docker-entrypoint-initdb.d
      - ./be/log:/opt/apache-doris/be/log
      - ./docker-build/be/resource/init_be.sh:/opt/apache-doris/be/bin/init_be.sh
      - ./be/start_be.sh:/opt/apache-doris/be/bin/start_be.sh
    networks:
      default:
        ipv4_address: 172.20.80.3
networks:
  default:
    external: true
    name: doris_es
```

启动前先创建一个外部网络，方便后续跟别的docker容器间互通。

```
docker network create --subnet=172.20.80.0/16 doris_es
```

启动后通过 http://localhost:8030/home 访问，用户名root，没有密码。



命令行通过：`mysql -uroot -P9030 -h127.0.0.1` 就可以访问doris。

执行命令 `show frontends\G;` 可以查看前端状态。

执行 `SHOW BACKENDS\G`可以查看后端状态



SET PASSWORD FOR 'root' = PASSWORD('whxheart');



创建数据库和测试表

```sql
create database demo;
use demo;

CREATE TABLE IF NOT EXISTS demo.example_tbl
(
    `user_id` LARGEINT NOT NULL COMMENT "用户id",
    `date` DATE NOT NULL COMMENT "数据灌入日期时间",
    `city` VARCHAR(20) COMMENT "用户所在城市",
    `age` SMALLINT COMMENT "用户年龄",
    `sex` TINYINT COMMENT "用户性别",
    `last_visit_date` DATETIME REPLACE DEFAULT "1970-01-01 00:00:00" COMMENT "用户最后一次访问时间",
    `cost` BIGINT SUM DEFAULT "0" COMMENT "用户总消费",
    `max_dwell_time` INT MAX DEFAULT "0" COMMENT "用户最大停留时间",
    `min_dwell_time` INT MIN DEFAULT "99999" COMMENT "用户最小停留时间"
)
AGGREGATE KEY(`user_id`, `date`, `city`, `age`, `sex`)
DISTRIBUTED BY HASH(`user_id`) BUCKETS 1
PROPERTIES (
    "replication_allocation" = "tag.location.default: 1"
);

```

准备数据源

```
10000,2017-10-01,北京,20,0,2017-10-01 06:00:00,20,10,10
10000,2017-10-01,北京,20,0,2017-10-01 07:00:00,15,2,2
10001,2017-10-01,北京,30,1,2017-10-01 17:05:45,2,22,22
10002,2017-10-02,上海,20,1,2017-10-02 12:59:12,200,5,5
10003,2017-10-02,广州,32,0,2017-10-02 11:20:00,30,11,11
10004,2017-10-01,深圳,35,0,2017-10-01 10:00:15,100,3,3
10004,2017-10-03,深圳,35,0,2017-10-03 10:20:22,11,6,6
```

 将上面的数据保存在一个test.csv文件中。

1. 导入数据

这里我们通过Stream load 方式将上面保存到文件中的数据导入到我们刚才创建的表里。

```text
curl  --location-trusted -u root: -T ~/Desktop/demo.csv -H "column_separator:," http://127.0.0.1:8030/api/demo/example_tbl/_stream_load
```



- -T test.csv : 这里是我们刚才保存的数据文件，如果路径不一样，请指定完整路径
- -u root : 这里是用户名密码，我们使用默认用户root，密码是空
- 127.0.0.1:8030 : 分别是 fe 的 ip 和 http_port



执行后提示

```
* Issue another request to this URL: 'http://root%40default_cluster:@172.20.80.3:8040/api/demo/example_tbl/_stream_load'
```

修改命令



```text
# curl  --location-trusted -u root: -T ~/Desktop/demo.csv -H "column_separator:," \
http://127.0.0.1:8040/api/demo/example_tbl/_stream_load

{
    "TxnId": 1,
    "Label": "8e722bc1-055c-4525-ad30-721cf25452f7",
    "TwoPhaseCommit": "false",
    "Status": "Success",
    "Message": "OK",
    "NumberTotalRows": 7,
    "NumberLoadedRows": 7,
    "NumberFilteredRows": 0,
    "NumberUnselectedRows": 0,
    "LoadBytes": 398,
    "LoadTimeMs": 127,
    "BeginTxnTimeMs": 22,
    "StreamLoadPutTimeMs": 49,
    "ReadDataTimeMs": 0,
    "WriteDataTimeMs": 23,
    "CommitAndPublishTimeMs": 30
}
```



执行sql

```

select * from example_tbl;
select * from example_tbl where city='上海';
select city, sum(cost) as total_cost from example_tbl group by city;

```

## 基本概念

在 Doris 中，数据以表（Table）的形式进行逻辑上的描述。 一张表包括行（Row）和列（Column）。Row 即用户的一行数据。Column 用于描述一行数据中不同的字段。

Column 可以分为两大类：Key 和 Value。从业务角度看，Key 和 Value 可以分别对应维度列和指标列。

Doris 的数据模型主要分为3类:

- Aggregate
- Unique
- Duplicate

## Aggregate 模型[](https://doris.apache.org/zh-CN/docs/data-table/data-model#aggregate-模型)

我们以实际的例子来说明什么是聚合模型，以及如何正确的使用聚合模型。

### 示例1：导入数据聚合 

假设业务有如下数据表模式：

| ColumnName      | Type        | AggregationType | Comment              |
| --------------- | ----------- | --------------- | -------------------- |
| user_id         | LARGEINT    |                 | 用户id               |
| date            | DATE        |                 | 数据灌入日期         |
| city            | VARCHAR(20) |                 | 用户所在城市         |
| age             | SMALLINT    |                 | 用户年龄             |
| sex             | TINYINT     |                 | 用户性别             |
| last_visit_date | DATETIME    | REPLACE         | 用户最后一次访问时间 |
| cost            | BIGINT      | SUM             | 用户总消费           |
| max_dwell_time  | INT         | MAX             | 用户最大停留时间     |
| min_dwell_time  | INT         | MIN             | 用户最小停留时间     |

如果转换成建表语句则如下（省略建表语句中的 Partition 和 Distribution 信息）

```sql
CREATE TABLE IF NOT EXISTS example_db.example_tbl
(
    `user_id` LARGEINT NOT NULL COMMENT "用户id",
    `date` DATE NOT NULL COMMENT "数据灌入日期时间",
    `city` VARCHAR(20) COMMENT "用户所在城市",
    `age` SMALLINT COMMENT "用户年龄",
    `sex` TINYINT COMMENT "用户性别",
    `last_visit_date` DATETIME REPLACE DEFAULT "1970-01-01 00:00:00" COMMENT "用户最后一次访问时间",
    `cost` BIGINT SUM DEFAULT "0" COMMENT "用户总消费",
    `max_dwell_time` INT MAX DEFAULT "0" COMMENT "用户最大停留时间",
    `min_dwell_time` INT MIN DEFAULT "99999" COMMENT "用户最小停留时间"
)
AGGREGATE KEY(`user_id`, `date`, `city`, `age`, `sex`)
DISTRIBUTED BY HASH(`user_id`) BUCKETS 1
PROPERTIES (
"replication_allocation" = "tag.location.default: 1"
);
```

可以看到，这是一个典型的用户信息和访问行为的事实表。 在一般星型模型中，用户信息和访问行为一般分别存放在维度表和事实表中。这里我们为了更加方便的解释 Doris 的数据模型，将两部分信息统一存放在一张表中。

表中的列按照是否设置了 `AggregationType`，分为 Key (维度列) 和 Value（指标列）。没有设置 `AggregationType` 的，如 `user_id`、`date`、`age` ... 等称为 **Key**，而设置了 `AggregationType` 的称为 **Value**。

当我们导入数据时，对于 Key 列相同的行会聚合成一行，而 Value 列会按照设置的 `AggregationType` 进行聚合。 `AggregationType` 目前有以下四种聚合方式：

1. SUM：求和，多行的 Value 进行累加。
2. REPLACE：替代，下一批数据中的 Value 会替换之前导入过的行中的 Value。
3. MAX：保留最大值。
4. MIN：保留最小值。



## Unique 模型[](https://doris.apache.org/zh-CN/docs/data-table/data-model#unique-模型)

在某些多维分析场景下，用户更关注的是如何保证 Key 的唯一性，即如何获得 Primary Key 唯一性约束。因此，我们引入了 Unique 的数据模型。该模型本质上是聚合模型的一个特例，也是一种简化的表结构表示方式。我们举例说明。

| ColumnName    | Type         | IsKey | Comment      |
| ------------- | ------------ | ----- | ------------ |
| user_id       | BIGINT       | Yes   | 用户id       |
| username      | VARCHAR(50)  | Yes   | 用户昵称     |
| city          | VARCHAR(20)  | No    | 用户所在城市 |
| age           | SMALLINT     | No    | 用户年龄     |
| sex           | TINYINT      | No    | 用户性别     |
| phone         | LARGEINT     | No    | 用户电话     |
| address       | VARCHAR(500) | No    | 用户住址     |
| register_time | DATETIME     | No    | 用户注册时间 |

这是一个典型的用户基础信息表。这类数据没有聚合需求，只需保证主键唯一性。（这里的主键为 user_id + username）。那么我们的建表语句如下：

```sql
CREATE TABLE IF NOT EXISTS example_db.example_tbl
(
    `user_id` LARGEINT NOT NULL COMMENT "用户id",
    `username` VARCHAR(50) NOT NULL COMMENT "用户昵称",
    `city` VARCHAR(20) COMMENT "用户所在城市",
    `age` SMALLINT COMMENT "用户年龄",
    `sex` TINYINT COMMENT "用户性别",
    `phone` LARGEINT COMMENT "用户电话",
    `address` VARCHAR(500) COMMENT "用户地址",
    `register_time` DATETIME COMMENT "用户注册时间"
)
UNIQUE KEY(`user_id`, `username`)
DISTRIBUTED BY HASH(`user_id`) BUCKETS 1
PROPERTIES (
"replication_allocation" = "tag.location.default: 1"
);
```

即 Unique 模型完全可以用聚合模型中的 REPLACE 方式替代。

```sql
CREATE TABLE IF NOT EXISTS example_db.example_tbl
(
    `user_id` LARGEINT NOT NULL COMMENT "用户id",
    `username` VARCHAR(50) NOT NULL COMMENT "用户昵称",
    `city` VARCHAR(20) REPLACE COMMENT "用户所在城市",
    `age` SMALLINT REPLACE COMMENT "用户年龄",
    `sex` TINYINT REPLACE COMMENT "用户性别",
    `phone` LARGEINT REPLACE COMMENT "用户电话",
    `address` VARCHAR(500) REPLACE COMMENT "用户地址",
    `register_time` DATETIME REPLACE COMMENT "用户注册时间"
)
AGGREGATE KEY(`user_id`, `username`)
DISTRIBUTED BY HASH(`user_id`) BUCKETS 1
PROPERTIES (
"replication_allocation" = "tag.location.default: 1"
);
```

## Duplicate 模型

在某些多维分析场景下，数据既没有主键，也没有聚合需求。因此，我们引入 Duplicate 数据模型来满足这类需求。举例说明。

| ColumnName | Type          | SortKey | Comment      |
| ---------- | ------------- | ------- | ------------ |
| timestamp  | DATETIME      | Yes     | 日志时间     |
| type       | INT           | Yes     | 日志类型     |
| error_code | INT           | Yes     | 错误码       |
| error_msg  | VARCHAR(1024) | No      | 错误详细信息 |
| op_id      | BIGINT        | No      | 负责人id     |
| op_time    | DATETIME      | No      | 处理时间     |

建表语句如下：

```sql
CREATE TABLE IF NOT EXISTS example_db.example_tbl
(
    `timestamp` DATETIME NOT NULL COMMENT "日志时间",
    `type` INT NOT NULL COMMENT "日志类型",
    `error_code` INT COMMENT "错误码",
    `error_msg` VARCHAR(1024) COMMENT "错误详细信息",
    `op_id` BIGINT COMMENT "负责人id",
    `op_time` DATETIME COMMENT "处理时间"
)
DUPLICATE KEY(`timestamp`, `type`, `error_code`)
DISTRIBUTED BY HASH(`type`) BUCKETS 1
PROPERTIES (
"replication_allocation" = "tag.location.default: 1"
);
```



这种数据模型区别于 Aggregate 和 Unique 模型。数据完全按照导入文件中的数据进行存储，不会有任何聚合。即使两行数据完全相同，也都会保留。 而在建表语句中指定的 DUPLICATE KEY，只是用来指明底层数据按照那些列进行排序。（更贴切的名称应该为 “Sorted Column”，这里取名 “DUPLICATE KEY” 只是用以明确表示所用的数据模型。关于 “Sorted Column”的更多解释，可以参阅[前缀索引](https://doris.apache.org/zh-CN/docs/data-table/index/prefix-index)）。在 DUPLICATE KEY 的选择上，我们建议适当的选择前 2-4 列就可以。

这种数据模型适用于既没有聚合需求，又没有主键唯一性约束的原始数据的存储。更多使用场景，可参阅**聚合模型的局限性**小节。



### Tablet & Partition 

在 Doris 的存储引擎中，用户数据被水平划分为若干个数据分片（Tablet，也称作数据分桶）。每个 Tablet 包含若干数据行。各个 Tablet 之间的数据没有交集，并且在物理上是独立存储的。

多个 Tablet 在逻辑上归属于不同的分区（Partition）。一个 Tablet 只属于一个 Partition。而一个 Partition 包含若干个 Tablet。因为 Tablet 在物理上是独立存储的，所以可以视为 Partition 在物理上也是独立。Tablet 是数据移动、复制等操作的最小物理存储单元。

若干个 Partition 组成一个 Table。Partition 可以视为是逻辑上最小的管理单元。数据的导入与删除，都可以或仅能针对一个 Partition 进行。

## 数据划分[](https://doris.apache.org/zh-CN/docs/data-table/data-partition#数据划分-1)

我们以一个建表操作来说明 Doris 的数据划分。

Doris 的建表是一个同步命令，SQL执行完成即返回结果，命令返回成功即表示建表成功。具体建表语法可以参考[CREATE TABLE](https://doris.apache.org/zh-CN/docs/sql-manual/sql-reference/Data-Definition-Statements/Create/CREATE-TABLE)，也可以通过 `HELP CREATE TABLE;` 查看更多帮助。

本小节通过一个例子，来介绍 Doris 的建表方式。

```sql
-- Range Partition

CREATE TABLE IF NOT EXISTS example_db.example_range_tbl
(
    `user_id` LARGEINT NOT NULL COMMENT "用户id",
    `date` DATE NOT NULL COMMENT "数据灌入日期时间",
    `timestamp` DATETIME NOT NULL COMMENT "数据灌入的时间戳",
    `city` VARCHAR(20) COMMENT "用户所在城市",
    `age` SMALLINT COMMENT "用户年龄",
    `sex` TINYINT COMMENT "用户性别",
    `last_visit_date` DATETIME REPLACE DEFAULT "1970-01-01 00:00:00" COMMENT "用户最后一次访问时间",
    `cost` BIGINT SUM DEFAULT "0" COMMENT "用户总消费",
    `max_dwell_time` INT MAX DEFAULT "0" COMMENT "用户最大停留时间",
    `min_dwell_time` INT MIN DEFAULT "99999" COMMENT "用户最小停留时间"
)
ENGINE=OLAP
AGGREGATE KEY(`user_id`, `date`, `timestamp`, `city`, `age`, `sex`)
PARTITION BY RANGE(`date`)
(
    PARTITION `p201701` VALUES LESS THAN ("2017-02-01"),
    PARTITION `p201702` VALUES LESS THAN ("2017-03-01"),
    PARTITION `p201703` VALUES LESS THAN ("2017-04-01")
)
DISTRIBUTED BY HASH(`user_id`) BUCKETS 16
PROPERTIES
(
    "replication_num" = "3",
    "storage_medium" = "SSD",
    "storage_cooldown_time" = "2018-01-01 12:00:00"
);


-- List Partition

CREATE TABLE IF NOT EXISTS example_db.example_list_tbl
(
    `user_id` LARGEINT NOT NULL COMMENT "用户id",
    `date` DATE NOT NULL COMMENT "数据灌入日期时间",
    `timestamp` DATETIME NOT NULL COMMENT "数据灌入的时间戳",
    `city` VARCHAR(20) COMMENT "用户所在城市",
    `age` SMALLINT COMMENT "用户年龄",
    `sex` TINYINT COMMENT "用户性别",
    `last_visit_date` DATETIME REPLACE DEFAULT "1970-01-01 00:00:00" COMMENT "用户最后一次访问时间",
    `cost` BIGINT SUM DEFAULT "0" COMMENT "用户总消费",
    `max_dwell_time` INT MAX DEFAULT "0" COMMENT "用户最大停留时间",
    `min_dwell_time` INT MIN DEFAULT "99999" COMMENT "用户最小停留时间"
)
ENGINE=olap
AGGREGATE KEY(`user_id`, `date`, `timestamp`, `city`, `age`, `sex`)
PARTITION BY LIST(`city`)
(
    PARTITION `p_cn` VALUES IN ("Beijing", "Shanghai", "Hong Kong"),
    PARTITION `p_usa` VALUES IN ("New York", "San Francisco"),
    PARTITION `p_jp` VALUES IN ("Tokyo")
)
DISTRIBUTED BY HASH(`user_id`) BUCKETS 16
PROPERTIES
(
    "replication_num" = "3",
    "storage_medium" = "SSD",
    "storage_cooldown_time" = "2018-01-01 12:00:00"
);
```

### 列定义[](https://doris.apache.org/zh-CN/docs/data-table/data-partition#列定义)

这里我们只以 AGGREGATE KEY 数据模型为例进行说明。更多数据模型参阅 [Doris 数据模型](https://doris.apache.org/zh-CN/docs/data-table/data-model)。

列的基本类型，可以通过在 mysql-client 中执行 `HELP CREATE TABLE;` 查看。

AGGREGATE KEY 数据模型中，所有没有指定聚合方式（SUM、REPLACE、MAX、MIN）的列视为 Key 列。而其余则为 Value 列。

定义列时，可参照如下建议：

1. Key 列必须在所有 Value 列之前。
2. 尽量选择整型类型。因为整型类型的计算和查找效率远高于字符串。
3. 对于不同长度的整型类型的选择原则，遵循 **够用即可**。
4. 对于 VARCHAR 和 STRING 类型的长度，遵循 **够用即可**。
5. 所有列的总字节长度（包括 Key 和 Value）不能超过 100KB。

### 分区和分桶[](https://doris.apache.org/zh-CN/docs/data-table/data-partition#分区和分桶)

Doris 支持两层的数据划分。第一层是 Partition，支持 Range 和 List 的划分方式。第二层是 Bucket（Tablet），仅支持 Hash 的划分方式。

也可以仅使用一层分区。使用一层分区时，只支持 Bucket 划分。下面我们来分别介绍下分区以及分桶：

1. **Partition**
   - Partition 列可以指定一列或多列，分区列必须为 KEY 列。多列分区的使用方式在后面 **多列分区** 小结介绍。
   - 不论分区列是什么类型，在写分区值时，都需要加双引号。
   - 分区数量理论上没有上限。
   - 当不使用 Partition 建表时，系统会自动生成一个和表名同名的，全值范围的 Partition。该 Partition 对用户不可见，并且不可删改。
   - 创建分区时**不可添加范围重叠**的分区。

**Range 分区**

- 分区列通常为时间列，以方便的管理新旧数据。
- Partition 支持通过 `VALUES LESS THAN (...)` 仅指定上界，系统会将前一个分区的上界作为该分区的下界，生成一个左闭右开的区间。同时，也支持通过 `VALUES [...)` 指定上下界，生成一个左闭右开的区间。
- 通过 `VALUES [...)` 同时指定上下界比较容易理解。这里举例说明，当使用 `VALUES LESS THAN (...)` 语句进行分区的增删操作时，分区范围的变化情况：

## Rollup 

Rollup 可以理解为 Table 的一个物化索引结构。**物化** 是因为其数据在物理上独立存储，而 **索引** 的意思是，Rollup可以调整列顺序以增加前缀索引的命中率，也可以减少key列以增加数据的聚合度。

## 物化视图 

物化视图是一种以空间换时间的数据分析加速技术。Doris 支持在基础表之上建立物化视图。比如可以在明细数据模型的表上建立基于部分列的聚合视图，这样可以同时满足对明细数据和聚合数据的快速查询。

同时，Doris 能够自动保证物化视图和基础表的数据一致性，并且在查询时自动匹配合适的物化视图，极大降低用户的数据维护成本，为用户提供一个一致且透明的查询加速体验。

# Rollup与查询

ROLLUP 在多维分析中是“上卷”的意思，即将数据按某种指定的粒度进行进一步聚合。

## 基本概念 

在 Doris 中，我们将用户通过建表语句创建出来的表称为 Base 表（Base Table）。Base 表中保存着按用户建表语句指定的方式存储的基础数据。

在 Base 表之上，我们可以创建任意多个 ROLLUP 表。这些 ROLLUP 的数据是基于 Base 表产生的，并且在物理上是**独立存储**的。

ROLLUP 表的基本作用，在于在 Base 表的基础上，获得更粗粒度的聚合数据。





sh