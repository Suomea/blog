## 日期时间支持的值格式
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


## 将轨迹数据转为 GeoJson
```sql
-- 将查询的数据 geojson 展示路径
SELECT CONCAT(
    '{"type": "LineString", "coordinates": [',
    GROUP_CONCAT(
        CONCAT('[', lon, ',', lat, ']') order by time 
    ),
    ']}'
) AS geojson
FROM vehicle_gps where license = '沪A-99999' and time >= '2001-01-01 00:00:00'  ;
```

## 赋予用户查询权限
赋予用户对数据库所有表的查询权限
```sql
create user 'user_a' identified by 'password';

-- 赋予部分表的查询权限
grant select on db.table1 to 'user_a';
grant select on db.table2 to 'user_a';
grant select on db.table3 to 'user_a';

-- 赋予数据库全部表的查询权限
grant select on db.* to 'user_a';

flush privileges;
```


