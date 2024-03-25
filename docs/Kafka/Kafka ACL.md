---
comments: true
tags:
  - Kafka
---
### Topic
创建 Topic。
```shell
bin/kafka-topics.sh — bootstrap-server localhost:9092 --create --topic quickstart-events [--partitions num] [--replication-factor num] --command-config admin-jaas
```

### 生产
赋予 `yunlu` 用户向 Topic `quickstart-events` 写入权限。
```shell
bin/kafka-acls.sh --bootstrap-server localhost:9092 --add --allow-principal User:yunlu --operation Write --topic quickstart-events --command-config admin-jaas
```

测试
```shell
bin/kafka-console-producer.sh --bootstrap-server localhost:9092  --topic quickstart-events --producer.config yunlu-jaas
```

### 消费
赋予 `yunlu` 用户使用 `yunlu` 消费者组消费 Topic `quickstart-events` 的权限。
```shell
bin/kafka-acls.sh --bootstrap-server localhost:9092 --add --allow-principal User:yunlu --operation Read --topic quickstart-events --group yunlu --command-config admin-jaas
```

测试
```shell
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic quickstart-events --from-beginning --group yunlu --consumer.config yunlu-jaas
```

赋予 `yunlu` 用户对所有 Topic 的读取权限。
```
bin/kafka-acls.sh ----bootstrap-server localhost:9092 --add --allow-principal User:yunlu --operation Read --topic '*' --group '*' --command-config admin-jaas
```

### 配置

查看所有的 ACL 权限配置。
```shell
bin/kafka-acls.sh --bootstrap-server localhost:9092 --list [--principal User:username] [--topic topicname] [--group topicname] --command-config admin-jaas
```


删除 `yunlu` 用户对 Topic `quickstart-event` 的读取权限。
```shell
bin/kafka-acls.sh --bootstrap-server localhost:9092 --remove --allow-principal User:yunlu --operation Read --topic quickstart-events --command-config admin-jaas 
```