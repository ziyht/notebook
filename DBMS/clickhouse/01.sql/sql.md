

## `表操作`

### 创建表

### 重命名表

```sql
rename table tbl1 to btl2;
```

### 删除表

```sql
drop table tbl;
```

### 添加列

```sql
alter table dsp_statis add column cost UInt32 default 0;
```

### 查看表结构

```sql
desc tbl;
```



```mysql
#添加列
ALTER TABLE [db].name [ON CLUSTER cluster] ADD COLUMN [IF NOT EXISTS] name [type] [default_expr] [AFTER name_after]
#删除列
ALTER TABLE [db].name [ON CLUSTER cluster] DROP COLUMN [IF EXISTS] name
#重置指定分区中列的所有数据
ALTER TABLE [db].name [ON CLUSTER cluster] CLEAR COLUMN [IF EXISTS] name IN PARTITION partition_name
#添加列注解
ALTER TABLE [db].name [ON CLUSTER cluster] COMMENT COLUMN [IF EXISTS] name 'comment'
#修改列类型或者列的默认值
ALTER TABLE [db].name [ON CLUSTER cluster] MODIFY COLUMN [IF EXISTS] name [type] [default_expr]
#添加索引
ALTER TABLE [db].name ADD INDEX name expression TYPE type GRANULARITY value AFTER name [AFTER name2]
#删除索引
ALTER TABLE [db].name DROP INDEX name
#分离分区
ALTER TABLE table_name DETACH PARTITION partition_expr
#删除分区
ALTER TABLE table_name DROP PARTITION partition_expr
#添加被分离的分区
ALTER TABLE table_name ATTACH PARTITION|PART partition_expr
#复制table1中的分区数据到table2
ALTER TABLE table2 REPLACE PARTITION partition_expr FROM table1
#重置列值为默认值，默认值为创建表时指定
ALTER TABLE table_name CLEAR COLUMN column_name IN PARTITION partition_expr
#创建指定分区或者所有分区的备份
ALTER TABLE table_name FREEZE [PARTITION partition_expr]
#从其他分片中复制分区数据
ALTER TABLE table_name FETCH PARTITION partition_expr FROM 'path-in-zookeeper'
```



### 查询

```mysql
SELECT [DISTINCT] expr_list
    [FROM [db.]table | (subquery) | table_function] [FINAL]
    [SAMPLE sample_coeff]
    [ARRAY JOIN ...]
    [GLOBAL] ANY|ALL INNER|LEFT JOIN (subquery)|table USING columns_list
    [PREWHERE expr]
    [WHERE expr]
    [GROUP BY expr_list] [WITH TOTALS]
    [HAVING expr]
    [ORDER BY expr_list]
    [LIMIT [n, ]m]
    [UNION ALL ...]
    [INTO OUTFILE filename]
    [FORMAT format]
    [LIMIT n BY columns]
```



### 分区

按时间分区：

toYYYYMM(EventDate)：按月分区

toMonday(EventDate)：按周分区

toDate(EventDate)：按天分区

按指定列分区：

PARTITION BY cloumn_name

对分区的操作：

alter table test1 DROP PARTITION [partition]  #删除分区

alter table test1 DETACH PARTITION [partition]#下线分区

alter table test1 ATTACH PARTITION [partition]#恢复分区

alter table .test1 FREEZE PARTITION [partition]#备份分区



### 数据同步

1)  采用remote函数

insert into db.table select * from remote('目标IP',db.table,'user','passwd')

2)  csv文件导入clickhouse

cat test.csv | clickhouse-client -u user --password password --query="INSERT INTO db.table FORMAT CSV"

3)  同步mysql库中表

CREATE TABLE tmp ENGINE = MergeTree ORDER BY id AS SELECT * FROM **mysql**('hostip:3306', 'db', 'table', 'user', 'passwd') ;

4） clickhouse-copier 工具



### 时间戳转换

select toUnixTimestamp('2018-11-25 00:00:02');

select toDateTime(1543075202);



查看表的空间占用情况：

```sql
SELECT \
database, \
table, \
formatReadableSize(size) AS size, \
formatReadableSize(bytes_on_disk) AS bytes_on_disk, \
formatReadableSize(data_uncompressed_bytes) AS data_uncompressed_bytes, \
formatReadableSize(data_compressed_bytes) AS data_compressed_bytes, \
compress_rate, \
rows, \
days, \
formatReadableSize(avgDaySize) AS avgDaySize \
FROM \
( \
SELECT \
database, \
table, \
sum(bytes) AS size, \
sum(rows) AS rows, \
min(min_date) AS min_date, \
max(max_date) AS max_date, \
sum(bytes_on_disk) AS bytes_on_disk, \
sum(data_uncompressed_bytes) AS data_uncompressed_bytes, \
sum(data_compressed_bytes) AS data_compressed_bytes, \
(data_compressed_bytes / data_uncompressed_bytes) * 100 AS compress_rate, \
max_date - min_date AS days, \
size / (max_date - min_date) AS avgDaySize \
FROM system.parts \
WHERE active \
GROUP BY \
database, \
table \
ORDER BY \
database ASC, \
size DESC \
)
```

示例：

```
┌─database─┬─table──────────────┬─size───────┬─bytes_on_disk─┬─data_uncompressed_bytes─┬─data_compressed_bytes─┬──────compress_rate─┬──────rows─┬─days─┬─avgDaySize─┐
│ logs     │ bsccd_logs         │ 5.64 GiB   │ 5.64 GiB      │ 36.72 GiB               │ 5.64 GiB              │ 15.345941116575846 │ 344986044 │   34 │ 169.88 MiB │
│ logs     │ cstc9_logs         │ 2.44 GiB   │ 2.44 GiB      │ 14.75 GiB               │ 2.43 GiB              │  16.49556309245452 │ 134889655 │    6 │ 415.64 MiB │
│ logs     │ bscca_logs         │ 854.81 MiB │ 854.81 MiB    │ 4.97 GiB                │ 854.10 MiB            │ 16.767609741408222 │  46648672 │    5 │ 170.96 MiB │
│ logs     │ devtest_logs       │ 204.21 MiB │ 204.21 MiB    │ 2.31 GiB                │ 204.05 MiB            │  8.623345473646571 │   9808875 │    4 │ 51.05 MiB  │
│ logs     │ cstc9_logs_times   │ 1.14 KiB   │ 1.14 KiB      │ 96.00 B                 │ 412.00 B              │  429.1666666666667 │         4 │    0 │ inf YiB    │
│ logs     │ bsccd_logs_times   │ 876.00 B   │ 876.00 B      │ 72.00 B                 │ 309.00 B              │  429.1666666666667 │         3 │    0 │ inf YiB    │
│ logs     │ bscca_logs_times   │ 584.00 B   │ 584.00 B      │ 48.00 B                 │ 206.00 B              │  429.1666666666667 │         2 │    0 │ inf YiB    │
│ logs     │ devtest_logs_times │ 292.00 B   │ 292.00 B      │ 24.00 B                 │ 103.00 B              │  429.1666666666667 │         1 │    0 │ inf YiB    │
│ system   │ trace_log          │ 13.69 MiB  │ 13.69 MiB     │ 78.64 MiB               │ 13.60 MiB             │ 17.296384195353557 │    423491 │   35 │ 400.64 KiB │
│ system   │ query_thread_log   │ 13.72 KiB  │ 13.72 KiB     │ 150.48 KiB              │ 11.92 KiB             │  7.921036237150866 │       246 │    0 │ inf YiB    │
│ system   │ query_log          │ 9.54 KiB   │ 9.54 KiB      │ 37.40 KiB               │ 7.41 KiB              │  19.81565617003499 │        44 │    0 │ inf YiB    │
└──────────┴────────────────────┴────────────┴───────────────┴─────────────────────────┴───────────────────────┴────────────────────┴───────────┴──────┴────────────┘
```

查看分区信息：

```
select partition, name, active from system.parts WHERE table = 'bsccd_logs'
```

示例：

```
┌─partition─┬─name─────────────────────┬─active─┐
│ 202004    │ 202004_1_252726_226      │      1 │
│ 202004    │ 202004_252727_577988_231 │      1 │
│ 202004    │ 202004_577989_579287_187 │      1 │
│ 202004    │ 202004_579288_580404_208 │      1 │
│ 202004    │ 202004_580405_580528_34  │      1 │
│ 202004    │ 202004_580529_580532_1   │      1 │
│ 202005    │ 202005_580533_627423_194 │      1 │
│ 202005    │ 202005_627424_682123_190 │      1 │
│ 202005    │ 202005_682124_684646_175 │      1 │
└───────────┴──────────────────────────┴────────┘
```





```
CREATE TABLE IF NOT EXISTS prometheus.samples\
    (\
          date Date DEFAULT toDate(0),\
          name String,\
          tags Array(String),\
          val Float64,\
          ts DateTime,\
          updated DateTime DEFAULT now()\
    )\
    ENGINE = GraphiteMergeTree(\
          date, (name, tags, ts), 8192, 'graphite_rollup'\
    );
```

