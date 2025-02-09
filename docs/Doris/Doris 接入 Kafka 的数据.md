
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
    "property.kafka_default_offsets" = "OFFSET_BEGINNING",
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

前端上传的数据日期格式是对的，但是值不对，比如 0024-01-01 00:00:00，这种数据由于没有对应的分区表，导致 Routine Load 导入任务暂停。

可以通过修改 Routine Load 导入任务，从指定的 Kafka Topic 分区和偏移开始消费数据，但是这样定位脏数据的位置比较麻烦。

或者，可以通过修改 Routine Load 导入任务 load_properties 的 WHERE 子模块，来过滤掉不符合条件的数据。
```sql
CREATE ROUTINE LOAD `yunlu_iot_data`.`kafka-quickstart_events` ON `quickstart_events`
WHERE born_time > '1900-01-01 00:00:00'
PROPERTIES (
    "format" = "json",
    "strict_mode" = "false")
FROM KAFKA (
    "kafka_broker_list" = "172.31.8.164:9092,172.31.8.165:9092,172.31.8.166:9092",
    "kafka_topic" = "quickstart-events",
    "property.group.id" = "xxxxx",
    "property.kafka_default_offsets" = "OFFSET_BEGINNING",
    "property.security.protocol" = "SASL_PLAINTEXT",
    "property.sasl.mechanism" = "PLAIN",
    "property.sasl.username" = "xxxx",
    "property.sasl.password" = "xxxx");
```

参考：https://doris.apache.org/zh-CN/docs/2.0/sql-manual/sql-reference/Data-Manipulation-Statements/Load/ALTER-ROUTINE-LOAD