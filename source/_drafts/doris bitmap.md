doris bitmap

# Bitmap 索引

用户可以通过创建bitmap index 加速查询 本文档主要介绍如何创建 index 作业，以及创建 index 的一些注意事项和常见问题。

## 名词解释 

- bitmap index：位图索引，是一种快速数据结构，能够加快查询速度

## 原理介绍 

创建和删除本质上是一个 schema change 的作业，具体细节可以参照 [Schema Change](https://doris.apache.org/zh-CN/docs/advanced/alter-table/schema-change)。







bitmap索引适用于列基数比较小的列，通俗来说就是值范围比较固定，重复率较高，建议基数在100到100,000之间，比如：country，city等。

bitmap不适用于更新频繁的列，在doris中，bitmap INDEX的创建和删除本质上就是Schema Change的过程，所以数据量较大的情况下耗时较长，相关优化后文详细介绍，如果列值更新，对象的索引值也要跟新，所以在频繁更新的列建索引会加大doris的负荷，对doris的性能大打折扣。

下面是doris索引实践总结：

### 一，适合bitmap索引的场景/条件 ：

通过实践发现，bitmap使用于列值基数低的情况，但不能太低，建议在100到100,000之间，如：jobtitle、city等。基数过高，则空间效率和性能会大大降低。
bitmap索引仅适用于一些特定类型的查询，例如count、or、and等 等值查询或者范围查询操作（这在数据分析的SQL中非常常见），因为只需要进行位运算。如本例中的查询。类似这种场景，如果在每个查询条件列上都建立了bitmap索引，则数据库可以进行高效的bit运算，精确定位到需要的数据，减少磁盘IO。并且筛选出的结果集越小，bitmap索引的优势越明显。基于这样的特点，bitmap索引适合建立在维度表的一些维度值列，且是更新不频繁的列。

