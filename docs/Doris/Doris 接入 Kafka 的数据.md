
### Kafka 账号授权
```
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
	--add \
	--allow-principal User:xxx \
	--operation Read \
	--topic '*' --group 'xxx' \
	--command-config admin-jaas 
```

### 创建唯一模型
```sql
CREATE TABLE user_info(
    user_id BIGINT NOT NULL COMMENT "用户 ID",
    name VARCHAR(20) COMMENT "用户姓名",
    born_time datetime COMMENT "出生日期"
)UNIQUE KEY(user_id)
DISTRIBUTED BY HASH(user_id) BUCKETS 1;


CREATE TABLE IF NOT EXISTS user_info  
(  
	name varchar(20) comment '姓名',
	age int comment '年龄',
	birth_date date comment '出生日期'
)DISTRIBUTED BY HASH(age) BUCKETS 3;
```

### 创建导入作业
```sql
CREATE ROUTINE LOAD `db_name`.`job_name` ON `user_info`
PROPERTIES (
    "format" = "json",
    "strict_mode" = "false")
FROM KAFKA (
    "kafka_broker_list" = "172.31.8.164:9092,172.31.8.165:9092,172.31.8.166:9092",
    "kafka_topic" = "quickstart-events",
    "property.group.id" = "xxxxx",
    "property.kafka_default_offsets" = "OFFSET_END",
    "property.security.protocol" = "SASL_PLAINTEXT",
    "property.sasl.mechanism" = "PLAIN",
    "property.sasl.username" = "xxxx",
    "property.sasl.password" = "xxxx");
```

### 往 Kafka 中写入数据测试
值为 null 或者日期格式都兼容，unique key 模式也能唯一去重保留新的数据。
```json
{ "user_id" : 1, "name" : "Benjamin", "born_time": "2023-01-01 00:00:00" }
{ "user_id" : 2, "name" : "Emily", "born_time": "20240101122323" }
{ "user_id" : 3, "name" : "Emily", "born_time": "20240101122323" }
{ "user_id" : 4, "name" : "Emily", "born_time": null }
{ "user_id" : 5, "name" : "Alexander"}
{ "user_id" : 4, "name" : "Alexander"}
{ "user_id" : 6, "born_time": "20240101122323", "name" : "Emily"}
```

### 导入任务管理
show routine load 能够查询导入作业的状态。

如果遇到错误，比如必填字段缺少会导致任务 pause，`resume routine load for task_name;` 重启启动任务就好，会跳过错误的消息。

测试完毕之后可以通过 pause 暂停导入任务：`pause routine load for  task_name;`。


## 案例调优

### 过滤数据
前端上传的数据日期格式是对的，但是值不对，比如 0024-01-01 00:00:00，这种数据由于没有对应的分区表，导致 Routine Load 导入任务暂停。

可以通过修改 Routine Load 导入任务，从指定的 Kafka Topic 分区和偏移开始消费数据，但是这样定位脏数据的位置比较麻烦。

或者，可以通过修改 Routine Load 导入任务 load_properties 的 WHERE 子模块，来过滤掉不符合条件的数据。

目前还不支持直接修改 RoutineLoad 增加 Where 子句，所以要 STOP RoutineLoad， 然后再重新创建。重新创建时，可以使用 kafka_partitions 和 kafka_offsets 指定每个分区的起始消费 offset，来避免重复或者遗漏消费数据。
```sql
CREATE ROUTINE LOAD `iot_data`.`kafka-quickstart_events` ON `quickstart_events`
WHERE born_time > '1900-01-01 00:00:00'
PROPERTIES (
    "format" = "json",
    "strict_mode" = "false")
FROM KAFKA (
    "kafka_broker_list" = "172.31.8.164:9092,172.31.8.165:9092,172.31.8.166:9092",
    "kafka_topic" = "quickstart-events",
    "kafka_partitions" = "0,1,2",
    "kafka_offsets" = "1239,1235,1237",
    "property.group.id" = "xxxxx",
    "property.security.protocol" = "SASL_PLAINTEXT",
    "property.sasl.mechanism" = "PLAIN",
    "property.sasl.username" = "xxxx",
    "property.sasl.password" = "xxxx");
```

参考：
https://doris.apache.org/zh-CN/docs/2.0/sql-manual/sql-reference/Data-Manipulation-Statements/Load/ALTER-ROUTINE-LOAD。
https://doris.apache.org/zh-CN/docs/sql-manual/sql-statements/data-modification/load-and-export/ALTER-ROUTINE-LOAD


导入任务报错 TOO_MANY_TASKS，参考官方公众号“2.0.9和2.1.3之前都存在已知的bug导致TOO_MANY_TASKS的问题”。实际使用下来 2.0.12 也存在这个问题，解决方案是升级至 2.0.15 或者 2.1.8，最终 2.0.12 直接升级到 2.1.8 解决报错的问题。

### 修改 RoutineLoad 子任务最大运行时间
默认 60 秒，为了提升数据入库频率，增加到 10 秒。
```
pause routine load for iot_data.hk_anpr;

ALTER ROUTINE LOAD FOR iot_data.hk_anpr PROPERTIES(
    "max_batch_interval" = "10"
)

resume routine load for iot_data.hk_anpr;
```