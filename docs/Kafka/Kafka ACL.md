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

赋予 `usera` 用户向 Topic `quickstart-events` 写入权限。
```shell
bin/kafka-acls.sh --bootstrap-server localhost:9092 --add --allow-principal User:usera --operation Write --topic quickstart-events --command-config admin-jaas
```

测试
```shell
bin/kafka-console-producer.sh --bootstrap-server localhost:9092  --topic quickstart-events --producer.config usera-jaas
```

### 消费

赋予 `usera` 用户使用 `usera` 消费者组消费 Topic `quickstart-events` 的权限。
```shell
bin/kafka-acls.sh --bootstrap-server localhost:9092 --add --allow-principal User:usera --operation Read --topic quickstart-events --group usera --command-config admin-jaas
```

测试
```shell
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic quickstart-events --from-beginning --group usera --consumer.config usera-jaas
```

赋予 `usera` 用户对所有 Topic 的读取权限。
```shell
bin/kafka-acls.sh ----bootstrap-server localhost:9092 --add --allow-principal User:usera --operation Read --topic '*' --group '*' --command-config admin-jaas
```

### 配置

查看所有的 ACL 权限配置。
```shell
bin/kafka-acls.sh --bootstrap-server localhost:9092 --list [--principal User:username] [--topic topicname] [--group topicname] --command-config admin-jaas
```


删除 `usera` 用户对 Topic `quickstart-event` 的读取权限。
```shell
bin/kafka-acls.sh --bootstrap-server localhost:9092 --remove --allow-principal User:usera --operation Read --topic quickstart-events --command-config admin-jaas 
```