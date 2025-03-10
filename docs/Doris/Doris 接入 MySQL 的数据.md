## 数据准备
### MySQL 准备
创建表
```sql
create database iot_data;
use iod_data;

CREATE TABLE `thermo_hygro_meter` (
  `device_id` varchar(20) DEFAULT NULL COMMENT '检测设备ID',
  `detect_time` datetime NOT NULL COMMENT '检测时间',
  `temperature` decimal(5,3) DEFAULT NULL COMMENT '温度，单位：摄氏度',
  `humidity` decimal(5,3) DEFAULT NULL COMMENT '相对湿度',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  UNIQUE KEY `device_id` (`device_id`,`detect_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='温湿度检测指标';
```

插入示例数据
```sql
insert into thermo_hygro_meter 
values 
("TH-001", '2024-01-01 00:00:00', 23.50, 50.01, '2024-01-01 00:00:02'),
("TH-001", '2024-01-02 00:00:00', 25.00, 48.00, '2024-01-02 00:00:01'),
("TH-001", '2024-01-03 00:00:00', 28.20, 49.21, '2024-01-03 00:00:02'),
("TH-002", '2024-01-01 00:00:00', 23.30, 47.11, '2024-01-01 00:00:01'),
("TH-002", '2024-01-02 00:00:00', 24.40, 48.21, '2024-01-02 00:00:02'),
("TH-002", '2024-01-03 00:00:00', 27.50, 49.31, '2024-01-03 00:00:02'),
("TH-003", '2024-01-01 00:00:00', 23.60, 45.41, '2024-01-01 00:00:01'),
("TH-003", '2024-01-02 00:00:00', 22.70, 52.56, '2024-01-02 00:00:02'),
("TH-003", '2024-01-03 00:00:00', 21.80, 53.74, '2024-01-03 00:00:02');
```

### Doris 准备
创建表
```sql
create database iot_data;
use iod_data;

CREATE TABLE `thermo_hygro_meter` (  
  `device_id` varchar(20) DEFAULT NULL COMMENT '检测设备ID',  
  `detect_time` datetime NOT NULL COMMENT '检测时间',  
  `temperature` decimal(5, 3) DEFAULT NULL COMMENT '温度，单位：摄氏度',  
  `humidity` decimal(5, 3) DEFAULT NULL COMMENT '相对湿度',  
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间'  
)   
UNIQUE KEY (`device_id`, `detect_time`)   
DISTRIBUTED BY HASH(`detect_time`) BUCKETS 1;  
```

另一张不相干的表，用来演示动态分区表的创建：
```sql
CREATE TABLE vehicle_gps_data (
  license varchar(255) comment '车牌',
  `time` datetime comment '时间',
  color varchar(100) comment '车牌颜色',
  lon decimal(15, 12) comment '经度',
  lat decimal(15, 12) comment '纬度',
  speed float comment '速度',
  altitude int comment '海拔',
  direction int comment '方向角',
  acc int comment '加速度',
  create_time datetime not null default current_timestamp comment '创建时间'
)
UNIQUE KEY (`license`, `time`)  -- 这里最好调整一下 time 和 license 的顺序，一般查询都会带上时间但是不一定会带上车牌号
PARTITION BY RANGE(`time`)() DISTRIBUTED BY HASH(`time`) BUCKETS 8 PROPERTIES (
 "replication_num" = "1",
  "dynamic_partition.enable" = "true",
  "dynamic_partition.time_unit" = "MONTH",
  "dynamic_partition.end" = "10",
  "dynamic_partition.prefix" = "p",
  "dynamic_partition.buckets" = "8",
  "dynamic_partition.create_history_partition" = "true",
  "dynamic_partition.history_partition_num" = "8",
  "dynamic_partition.time_zone"="Asia/Shanghai"
);

-- 将查询的数据 geojson 展示路径
SELECT CONCAT(
    '{"type": "LineString", "coordinates": [',
    GROUP_CONCAT(
        CONCAT('[', lon, ',', lat, ']') order by time 
    ),
    ']}'
) AS geojson
FROM za_gps_data where  license = 'xxxxx' and time >= '2024-07-09 00:00:00'  ;
```

```sql
-- 查询分区信息
show partitions from vehicle_gps_data;
```
## INSERT INTO SELECT
参考文档  
1. https://doris.apache.org/zh-CN/docs/2.0/lakehouse/database/jdbc  
2. https://doris.apache.org/zh-CN/docs/2.0/data-operate/import/insert-into-manual  

下载最新的 MySQL JDBC 驱动，放在 FE 和 BE 的 `fe/jdbc_drivers/` `be/jdbc_drivers` 目录下。

创建数据源
```sql
-- 创建 mysql 数据源
CREATE CATALOG mysql PROPERTIES (
    "type"="jdbc",
    "user"="root",
    "password"="xxxx",
    "jdbc_url" = "jdbc:mysql://192.168.31.11:3306",
    "driver_url" = "mysql-connector-j-8.4.0.jar",
    "driver_class" = "com.mysql.cj.jdbc.Driver"
);

-- 查询添加的 catalog 数据源
show catalogs;

-- 查询 mysql 中有哪些数据库
show databases from mysql;

-- 查看 mysql iot_data 数据库中有哪些表
show tables from mysql.iot_data;

-- 查看 mysql iot_data 数据库中 thermo_hygro_meter 表的数据
select * from mysql.iot_data.thermo_hygro_meter;
```

导入 MySQL 中的数据到 Doris。
```sql
insert into thermo_hygro_meter select * from mysql.iot_data.thermo_hygro_meter;

-- 或者

insert into thermo_hygro_meter(device_id, detect_time, temperature, humidity, create_time)
select device_id, detect_time, temperature, humidity, create_time from mysql.iot_data.thermo_hygro_meter;
```

有一个问题，创建完 mysql 数据源之后，mysql 的元数据信息如表的字段列表貌似会被 Doris 缓存下来，如果 MySQL 更新了字段会导致在 Doris 中查询（`select *` 或者 `select new_field_name`）出错。[提问链接](https://ask.selectdb.com/questions/D1ff1/jdbc-catalog-geng-xin-yuan-shu-ju-biao-zi-duan-wen-ti) 解决方案是刷新 Catalog 数据源的元数据信息，[参考链接](https://doris.apache.org/zh-CN/docs/dev/sql-manual/sql-statements/Utility-Statements/REFRESH/?_highlight=refresh)：
```
REFRESH CATALOG catalog_name;  
REFRESH DATABASE [catalog_name.]database_name;  
REFRESH TABLE [catalog_name.][database_name.]table_name;
```

测试大量数据导入的性能。

在 2.1 版本 Doris 引入了 Job Scheduler 实现作业调度，结合 Catalog 能够实现定时同步数据的方案，最高频率为 1 分钟执行一次。

创建同步作业
```
CREATE JOB my_job 
ON SCHEDULE EVERY 1 DAY STARTS '2020-01-01 00:00:00' 
DO INSERT INTO db1.tbl1 SELECT * FROM db2.tbl2 WHERE  create_time >=  days_add(now(),-1);
```

查询同步作业和任务
```
select * from jobs(type='insert');
select * from tasks(type='insert');
```

删除定时同步作业
```
DROP JOB where jobName = <job_name> ;
```
## JDBC

对于一些数据库（比如 MySQL），默认情况下 JDBC 并不总是将多个 INSERT 语句合并成一个批量插入语句发送给数据库，而是逐条发送。这会增加网络延迟和数据库处理开销。启用 `rewriteBatchedStatements` 后，JDBC 会尝试将多个 INSERT 语句合并为一个大的 INSERT 语句，以便一次性发送给数据库，从而显著提升插入效率。

Doris 在启用了 `rewriteBatchedStatements=true` 之后大批量的实时写入，还是有点慢。单次写入 2w 条数据大概耗时 20s。

## StreamLoad
参考数据导入的 [StreamLoad](https://doris.apache.org/zh-CN/docs/2.0/data-operate/import/stream-load-manual) 的方式。

如果 JSON 字段与表字段完全一致，可以省略 jsonpaths 和 columns 请求头。

```shell
curl -L -X PUT 'http://172.31.8.116:8030/api/dbname/tablename/_stream_load' \
	-H 'Expect: 100-continue' \
	-H 'format: json' \
	-H 'strip_outer_array: true' \
	-H 'jsonpaths: ["$.user_id", "$.name", "$.born_time"]' \
	-H 'columns: user_id,name,born_time' \
	-H 'Content-Type: application/json' \
	-H 'Authorization: Basic base64(username:password)' \
	-d '[
    {
        "user_id": 1,
        "name": "Emily",
        "born_time": "20231212234523111"
    },
    {
        "user_id": 2,
        "name": "Benjamin",
        "born_time": "20231212234523222"
    }]'
```
