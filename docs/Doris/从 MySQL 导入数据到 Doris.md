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

## JDBC Catalog 配合 INSERT INTO SELECT
参考文档
1. https://doris.apache.org/zh-CN/docs/2.0/lakehouse/database/jdbc
2. https://doris.apache.org/zh-CN/docs/2.0/data-operate/import/insert-into-manual
INSERT INTO SELECT 导入
下载最新的 MySQL JDBC 驱动，放在 FE 和 BE 的 `fe/jdbc_drivers/` `be/jdbc_drivers` 目录下。

创建数据源
```sql
-- 创建 mysql 数据源
CREATE CATALOG mysql PROPERTIES (
    "type"="jdbc",
    "user"="root",
    "password"="Chxy@122619",
    "jdbc_url" = "jdbc:mysql://192.168.31.11:3306",
    "driver_url" = "mysql-connector-j-8.4.0.jar",
    "driver_class" = "com.mysql.cj.jdbc.Driver"
);

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

insert into
  thermo_hygro_meter(
    device_id,
    detect_time,
    temperature,
    humidity,
    create_time
  )
select
  device_id,
  detect_time,
  temperature,
  humidity,
  create_time
from
  mysql.iot_data.thermo_hygro_meter;
```

有一个问题，创建完 mysql 数据源之后，mysql 的元数据信息如表的字段列表貌似会被 Doris 缓存下来，如果 MySQL 更新了字段会导致在 Doris 中查询（`select *` 或者 `select new_field_name`）出错。 [提问链接](https://ask.selectdb.com/questions/D1ff1/jdbc-catalog-geng-xin-yuan-shu-ju-biao-zi-duan-wen-ti)

测试大量数据导入的性能

这种导入方式好像不支持 CDC。
## SeaTunnel 
