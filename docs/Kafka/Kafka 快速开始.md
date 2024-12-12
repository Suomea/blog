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

进入到 Kafka 目录下，新建目录 kafka-files。
```
mkdir kafka-files
```

编辑配置文件 config/kraft/reconfig-server.properties，修改 log.dirs=/usr/local/kafka_2.13-3.9.0/kafka-files。

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

## 安装 Kafka UI 
KafkaUI 仓库地址：https://github.com/provectus/kafka-ui

docker 安装
```bash
docker run -itd --name kafka-ui -p 8080:8080 -e DYNAMIC_CONFIG_ENABLED=true provectuslabs/kafka-ui
```

jar 包直接安装，配置参考：https://github.com/provectus/kafka-ui/tree/master/kafka-ui-api/src/main/resources
```bash
java -jar kafka-ui-api-v0.7.2.jar --dynamic.config.enabled=true
```

jar 包直接运行，指定额外的配置文件：
```bash
java -Dspring.config.additional-location=<path-to-application-local.yml> -jar <path-to-jar>.jar
```
### 生产消费代码示例

引入依赖
```xml
<dependency>  
    <groupId>org.apache.kafka</groupId>  
    <artifactId>kafka-clients</artifactId>  
    <version>3.9.0</version>  
</dependency>
```

生产者
```java
public static void main(String[] args) {  
    // Kafka 配置信息  
    Properties properties = new Properties();  
    properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");  
    properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());  
    properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());  
  
    // Kafka 生产者  
    try (KafkaProducer<String, String> producer = new KafkaProducer<>(properties)) {  
        // Kafka 发送消息  
        for (int i = 0; i < 10; i ++) {  
            String key = String.format("[key: %d]", i);  
            String msg = String.format("[msg: %d]", i);  
            ProducerRecord<String, String> record = new ProducerRecord<>("quickstart-events", key, msg);  
            producer.send(record, (recordMetadata, e) -> {  
                System.out.println(msg + " -- partition: " + recordMetadata.partition() + " -- offset: " + recordMetadata.offset());  
            });  
        }  
    }  
}
```

消费者
```java
private static volatile boolean running = true;  
  
public static void main(String[] args) {  
    Properties properties = new Properties();  
    properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");  
    properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());  
    properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());  
    properties.put(ConsumerConfig.GROUP_ID_CONFIG, "group-a");  
  
    try(KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<>(properties)) {  
        kafkaConsumer.subscribe(Collections.singletonList("quickstart-events"));  
  
        while (running) {  
            ConsumerRecords<String, String> consumerRecords = kafkaConsumer.poll(100);  
            for (ConsumerRecord<String, String> consumerRecord : consumerRecords) {  
                System.out.println(consumerRecord);  
            }  
        }  
    }  
}
```