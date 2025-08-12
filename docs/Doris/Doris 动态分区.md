## Doris 创建分区表
创建分区表的示例 SQL：
```sql
CREATE TABLE vehicle_gps_data (
  license varchar(255) comment '车牌',
  `time` datetime comment '时间',
  color varchar(100) comment '车牌颜色',
  lon decimal(15, 12) comment '经度',
  lat decimal(15, 12) comment '纬度',
  create_time datetime not null default current_timestamp comment '创建时间'
)
UNIQUE KEY (`time`, `license`)
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
```

查询分区信息：
```sql
-- 查询分区信息
show partitions from vehicle_gps_data;
```

修改动态分区属性：
```
alter table vehicle_gps_data set (
	"dynamic_partition.start" = "-2"
)
```
!!!note
	在使用 ALTER TABLE 语句修改动态分区时，不会立即生效。Doris 会以 `dynamic_partition_check_interval_seconds` 参数指定的时间间隔轮训检查 dynamic partition 分区，完成需要的分区创建与删除操作。

## 动态分区属性参数

动态分区的参数以 dynamic_partition 为前缀，可以设置以下规则参数：

| 参数                                           | 必选  | 说明                                                                                                                                                                                                                                                          |
| -------------------------------------------- | --- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `enable`                                     | 否   | 是否开启动态分区特性。可以指定为 TRUE 或 FALSE。如果指定了动态分区其他必填参数，默认为 TRUE。                                                                                                                                                                                                     |
| `time_unit`                | 是   | 动态分区调度的单位。可指定为 `HOUR`、`DAY`、`WEEK`、`MONTH`、`YEAR`。分别表示按小时、按天、按星期、按月、按年进行分区创建或删除：                                                                                                                                                                            |
| `start`                    | 否   | 动态分区的起始偏移，为负数。默认值为 -2147483648，即不删除历史分区。根据 `time_unit` 属性的不同，以当天（星期/月）为基准，分区范围在此偏移之前的分区将会被删除。此偏移之后至当前时间的历史分区如不存在，是否创建取决于 `dynamic_partition.create_history_partition`。                                                                                      |
| `end`                      | 是   | 动态分区的结束偏移，为正数。根据 `time_unit` 属性的不同，以当天（星期/月）为基准，提前创建对应范围的分区。                                                                                                                                                                                                |
| `prefix`                   | 是   | 动态创建的分区名前缀。                                                                                                                                                                                                                                                 |
| `buckets`                  | 否   | 动态创建的分区所对应的分桶数。设置该参数后会覆盖 `DISTRIBUTED` 中指定的分桶数。量。                                                                                                                                                                                                           |
| `replication_num`          | 否   | 动态创建的分区所对应的副本数量，如果不填写，则默认为该表创建时指定的副本数量。                                                                                                                                                                                                                     |
| `create_history_partition` | 否   | 默认为 false。当置为 true 时，Doris 会自动创建所有分区，具体创建规则见下文。同时，FE 的参数 `max_dynamic_partition_num` 会限制总分区数量，以避免一次性创建过多分区。当期望创建的分区个数大于 `max_dynamic_partition_num` 值时，操作将被禁止。当不指定 `start` 属性时，该参数不生效。                                                                      |
| `history_partition_num`    | 否   | 当`create_history_partition` 为 `true` 时，该参数用于指定创建历史分区数量。默认值为 -1，即未设置。该变量与 `dynamic_partition.start` 作用相同，建议同时只设置一个。                                                                                                                                          |
| `start_day_of_week`        | 否   | 当 `time_unit` 为 `WEEK` 时，该参数用于指定每周的起始点。取值为 1 到 7。其中 1 表示周一，7 表示周日。默认为 1，即表示每周以周一为起始点。                                                                                                                                                                       |
| `start_day_of_month`       | 否   | 当 `time_unit` 为 `MONTH` 时，该参数用于指定每月的起始日期。取值为 1 到 28。其中 1 表示每月 1 号，28 表示每月 28 号。默认为 1，即表示每月以 1 号为起始点。暂不支持以 29、30、31 号为起始日，以避免因闰年或闰月带来的歧义。                                                                                                                    |
| `reserved_history_periods` | 否   | 需要保留的历史分区的时间范围。当`dynamic_partition.time_unit` 设置为 "DAY/WEEK/MONTH/YEAR" 时，需要以 `[yyyy-MM-dd,yyyy-MM-dd],[...,...]` 格式进行设置。当`dynamic_partition.time_unit` 设置为 "HOUR" 时，需要以 `[yyyy-MM-dd HH:mm:ss,yyyy-MM-dd HH:mm:ss],[...,...]` 的格式来进行设置。如果不设置，默认为 `"NULL"`。 |
| `time_zone`                | 否   | 动态分区时区，默认为当前服务器的系统时区，如 `Asia/Shanghai`。更多时区设置可以参考[时区管理](https://doris.apache.org/zh-CN/docs/admin-manual/cluster-management/time-zone/)。                                                                                                                    |

## FE 配置参数

可以在 FE 配置文件或通过 `ADMIN SET FRONTEND CONFIG` 命令修改 FE 中的动态分区参数配置：

|参数|默认值|说明|
|---|---|---|
|`dynamic_partition_enable`|false|是否开启 Doris 的动态分区功能。该参数只影响动态分区表的分区操作，不影响普通表。|
|`dynamic_partition_check_interval_seconds`|600|动态分区线程的执行频率，单位为秒。|
|`max_dynamic_partition_num`|500|用于限制创建动态分区表时可以创建的最大分区数，避免一次创建过多分区。|