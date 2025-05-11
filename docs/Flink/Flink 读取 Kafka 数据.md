添加依赖
```xml
<dependency>  
    <groupId>org.apache.flink</groupId>  
    <artifactId>flink-streaming-java</artifactId>  
    <version>1.20.0</version>  
</dependency>  
  
<dependency>  
    <groupId>org.apache.flink</groupId>  
    <artifactId>flink-clients</artifactId>  
    <version>1.20.0</version>  
</dependency>  
  
<dependency>  
    <groupId>org.apache.flink</groupId>  
    <artifactId>flink-connector-base</artifactId>  
    <version>1.20.0</version>  
</dependency>  
  
<dependency>  
    <groupId>org.apache.flink</groupId>  
    <artifactId>flink-connector-kafka</artifactId>  
    <version>3.1.0-1.18</version>  
</dependency>
```

示例代码：
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();  
env.setParallelism(3);  
  
KafkaSource<String> kafkaSource = KafkaSource.<String>builder()  
        .setBootstrapServers("192.168.31.115:9092")  
        .setGroupId("jacky-flink")  
        .setTopics("quickstart-events")  
        .setValueOnlyDeserializer(new SimpleStringSchema())  
        .setStartingOffsets(OffsetsInitializer.latest())  
        .build();  
  
env.fromSource(kafkaSource, WatermarkStrategy.noWatermarks(), "kafkaQuickstartEvents")  
        .print();  
  
env.execute();
```

关于 offset
Kafka 的参数 auto.reset.offsets 的配置
- earliest：如果存在 offset，则从 offset 开始消费；否则从最早的数据开始消费。
- latest：如果存在 offset，则从 offset 开始消费；否则从最新的数据开始消费。

Flink 的 setStartingOffsets 方法参数：
- OffsetsInitializer.earliest()：从最早的数据开始消费。
- OffsetsInitializer.latest()：从最新的数据开始消费。