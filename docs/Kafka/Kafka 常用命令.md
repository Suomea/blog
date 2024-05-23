---
comments: true
tags:
  - Kafka
---
## Topic

查看所有的 topic
```
bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

查看 topic 的信息
```
bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic <topic_name>
```

查看 topic 每个分区的 offset
```
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic <topic_name>
```
## Consumer

控制台消费者
```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic <topic_name>

默认从 latest 消费消息，创建一个新的消费者组。下面是一些可以指定的选项：
--from-beginning 指定了从开头读取消息

--max-message 指定了读取消息的数量，不指定的话会持续消费下去

--offset 执行消费的起始 offset 位置

--partition 指定分区
```

指定分区消费最新的两条消息，partition 和 offset 需要同时指定。
```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic <topic_name> --partition <partitionidu> --offset <offset> --max-messages 2
```

## ACL
### Topic

创建 Topic。
```shell
bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic quickstart-events [--partitions num] [--replication-factor num] --command-config admin-jaas
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
bin/kafka-acls.sh --bootstrap-server localhost:9092 --add --allow-principal User:usera --operation Read --topic '*' --group '*' --command-config admin-jaas
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