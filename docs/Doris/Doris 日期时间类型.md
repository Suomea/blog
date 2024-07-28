支持的格式
```
yyyy-MM-dd HH:mm:ss
yyyy-MM-dd HH:mm:ss.SSS
yyyyMMddHHmmss
yyyyMMddHHmmssSSS
```

测试
```sql
create table test_dt(
	k1 TINYINT,
    k2 datetime
) 
COMMENT "datetime type test table"
DISTRIBUTED BY HASH(k1) BUCKETS 1
PROPERTIES ('replication_num' = '1');

insert into test_dt(k1, k2)
values
(1, '2024-07-19 12:32:45'),
(1, '2024-07-19 12:32:45.111'),
(1, '20240719123245'),
(1, '20240719123245111'),
(1, '20240719 123245');   // 这一条不会识别正确的日期时间，但是插入不会报错，日期时间字段为 null
```