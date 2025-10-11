---
comments: true
tags:
  - Kafka
---
## 主题操作

topic 操作选项
```
--topic 指定要操作的 topic 名称
--create 创建 topic
--delete 删除 topic
--describe 查看 topic 的详细描述
--alter 修改主题
--partitions <num> 修改主题的分区数，只能增加，不能减少
--replication-factor <num> 设置分区副本数（如果设置为 3，那么就是 1 个 Leader 副本，3 个 Follower 副本）
--config <key=val> 更新系统默认设置
```

创建 Topic：
```shell
bin/kafka-topics.sh --bootstrap-server localhost:9092 \
	--create \
	--topic quickstart-events [--partitions num] [--replication-factor num] --command-config admin-jaas
```

查看 Topic 的分区 offset 信息：
```
bin/kafka-run-class kafka.tools.GetOffsetShell ----bootstrap-server localhost:9092 \
 --topic <TOPIC> \
 --time -1
 
# time -1 表示获取最新的 offset 信息
```
## 消费消息
查看消费者组的消费情况
```
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group GOUP_NAME --describe 
```

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
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 \
	--topic <topic_name> --partition <partitionidu> --offset <offset> --max-messages 2
```

## 生产者权限

赋予 usera 用户向 Topic `quickstart-events` 写入权限。
```shell
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
	--add \
	--allow-principal User:usera \
	--operation Write \
	--topic quickstart-events \
	--command-config admin-jaas
```

赋予 usera 用户向所有 topic 写入的权限：
```
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
	--add \
	--allow-principal User:usera \
	--operation Write \
	--topic '*' \
	--command-config admin-jaas
```

测试
```shell
bin/kafka-console-producer.sh --bootstrap-server localhost:9092  \
	--topic quickstart-events \
	--producer.config usera-jaas
```

## 消费者权限

赋予 `usera` 用户使用 `usera` 消费者组消费 Topic `quickstart-events` 的权限。
```shell
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
	--add \
	--allow-principal User:usera \
	--operation Read \
	--topic quickstart-events \
	--group usera \
	--command-config admin-jaas
```

测试
```shell
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 \
	--topic quickstart-events \
	--from-beginning \
	--group usera \
	--consumer.config usera-jaas
```

赋予 `usera` 用户对所有 Topic 的读取权限。
```shell
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
	--add \
	--allow-principal User:usera \
	--operation Read \
	--topic '*' \
	--group '*' \
	--command-config admin-jaas
```

## 权限查看

查看所有的 ACL 权限配置。
```shell
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
	--list [--principal User:username] [--topic topicname] [--group topicname] \
	--command-config admin-jaas
```

## 权限删除
删除 `usera` 用户对 Topic `quickstart-event` 的读取权限。
```shell
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
	--remove \
	--allow-principal User:usera \
	--operation Read \
	--topic quickstart-events \
	--command-config admin-jaas 
```