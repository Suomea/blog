### Kafka 安装
准备两个文件
kafka_2.13-3.9.0.tgz
OpenJDK21U-jdk_aarch64_linux_hotspot_21.0.5_11.tar.gz

解压缩
 tar -zxvf OpenJDK21U-jdk_aarch64_linux_hotspot_21.0.5_11.tar.gz -C /usr/local/
 tar -zxvf kafka_2.13-3.9.0.tgz -C /usr/local/

配置 Java 环境  /etc/profile.d/java.sh， 执行 source /etc/profile，java -version 验证配置。
```
export JAVA_HOME=/usr/local/jdk-21.0.5+11
export PATH=$PATH:$JAVA_HOME/bin
```

进入到 Kafka 目录下，新建目录 kafka-files，编辑配置文件 config/kraft/server.properties，修改 log.dirs=/usr/local/kafka_2.13-3.9.0/kafka-files。

初始化启动 Kafka
```
KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"
bin/kafka-storage.sh format --standalone -t $KAFKA_CLUSTER_ID -c config/kraft/reconfig-server.properties
bin/kafka-server-start.sh -daemon config/kraft/reconfig-server.properties
```


创建 Topic
```
# bin/kafka-topics.sh --bootstrap-server localhost:9092 --topic quickstart-events --create
Created topic quickstart-events.
```

查看 Topic 列表
```
# bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
quickstart-events
```

查看 Topic 详情
```
# bin/kafka-topics.sh --bootstrap-server localhost:9092 --topic quickstart-events --describe
Topic: quickstart-events TopicId: l9MVAfeWQ1am7pCdFHbdiA PartitionCount: 1 ReplicationFactor: 1 Configs: segment.bytes=1073741824
Topic: quickstart-events Partition: 0 Leader: 1 Replicas: 1 Isr: 1 Elr: LastKnownElr:
```

修改 Topic 的分区
```
# bin/kafka-topics.sh --bootstrap-server localhost:9092 --topic quickstart-events --alter --partitions 2
# bin/kafka-topics.sh --bootstrap-server localhost:9092 --topic quickstart-events --describe
Topic: quickstart-events TopicId: l9MVAfeWQ1am7pCdFHbdiA PartitionCount: 2 ReplicationFactor: 1 Configs: segment.bytes=1073741824
Topic: quickstart-events Partition: 0 Leader: 1 Replicas: 1 Isr: 1 Elr: LastKnownElr: 
Topic: quickstart-events Partition: 1 Leader: 1 Replicas: 1 Isr: 1 Elr: LastKnownElr:
```

命令行消费消息
```
# bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic quickstart-events
```

命令行生产消息
```
# bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic quickstart-events
```

删除 Topic
```
# bin/kafka-topics.sh --bootstrap-server localhost:9092 --topic quickstart-events --delete
```

### 生产消费代码示例
