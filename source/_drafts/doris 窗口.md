doris 窗口



```

PROPERTIES(
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-7",  -- 保留分区个数
    "dynamic_partition.end" = "3",     -- 自动创建分区个数
    "dynamic_partition.prefix" = "p", -- 分区前缀
    "dynamic_partition.buckets" = "10", -- 分桶数量
    "replication_num" = "1",            -- 副本数量
    "storage_medium" = "SSD"
);


select sum(mention_numbers),sum(yesterday)
from (
SELECT dt, mention_numbers ,
 lag(mention_numbers,3, 0) over ( order by dt) yesterday
FROM cem_doris.dimension_metrics_9
WHERE  metrics_library_id='1655888400916538345' and brand ='0' and region ='0' and datasource_id ='0'
and metrics_id ='AA' and dealer ='0' and dt >='2023-5-29' and dt<'2023-06-06'
) c




select sum(mention_numbers),sum(yesterday)
from (
SELECT dt, mention_numbers ,
 lag(mention_numbers,3, 0) over ( order by dt) yesterday
FROM cem_doris.dimension_metrics_9
WHERE  metrics_library_id='1655888400916538345' and brand ='0' and region ='0' and datasource_id ='0'
and metrics_id ='AA' and dealer ='0' and dt >='2023-05-31' and dt<'2023-06-06'
) c
WHERE dt >='2023-06-03' and dt<'2023-06-06' 
```

