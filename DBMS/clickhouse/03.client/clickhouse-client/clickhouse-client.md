## 导入CSV数据

```sh
# 方式一：交互式
insert into tableName (c1, c2, ...) values (v1, v2, ...)
insert into tableName (c1, c2, ...) select ... 

# 方式二：批量
clickhouse-client --quey="insert into tableName format CSV" < file.csv

# 方式三：http客户端
echo -ne '10\n11\n12\n' | POST 'http://localhost:8123/?query=insert into tableName format TabSeparated'
```

### 指定分隔符

```sh
cat xxx.csv | clickhouse-client --format_csv_delimiter="|" --query="INSERT INTO b6logs FORMAT CSV";
```



## 导入数据时忽略错误

```sh
clickhouse-client --input_format_allow_errors_num=100000 --input_format_allow_errors_ratio=0.2
```

`--input_format_allow_errors_num` : 是允许的错误数

`--input_format_allow_errors_ratio` : 是允许的错误率, 范围是 [0-1]



## 导出 CSV 数据

```sh
clickhouse-client --query="select uid, idfa, imei from (select impid, uid from b2logs where impid >= 15289903030261609347 and impid <= 15289904230261609347) any inner join (select impid, idfa, imei from b6logs where impid >= 15289903030261609347 and impid <= 15289904230261609347) using(impid) format CSV" > 9c9dc608-269b-4f02-b122-ef5dffb2669d.log
```

```sh
# 方式一：交互式
select * from tableName into outfile 'path/file'

# 方式二：非交互式
clickhouse-client  --database bdName -u default --password password --query='select * from tableName' > abc

# 方式三：http客户端
echo 'select 1 FORMAT TabSeparated' | curl "http://user:password@localhost:8123/" -d @- > file
```

