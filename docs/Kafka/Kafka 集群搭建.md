#Kafka

参考连接：
https://developer.aliyun.com/article/1247585 KRaft 集群安装 Kafka 介绍
https://medium.com/@azsecured/install-kafka-cluster-kraft-with-sasl-plaintext-and-acl-configs-ae01a1e0040d 集群安装，配置 ACL

>基于 KRaft 模式。
## 节点类型

Broker 节点，Kafka 中的工作节点，负责存储和处理消息。

Controller 节点，控制器节点，负责存储和管理整个集群的元数据和状态。

混合节点，同时担任 Broker 和 Contoller 角色。

Client 客户端。

## 安全

Kafka 通信链路

Controller - Controller
Broker - Controller
Broker -Broker
Clinet - Broker

Controller 之间的通信使用使用 `controller.listener.name` 指定 `listener`。Broker 之间的通信使用`inter.broker.listener.name` 指定 `listener`，不能和 `controller.listener.name` 使用同一个 `listener`。



通信链路之间有加密和明文的方式。
- SSL
- PLAINTEXT

通信链路之间可以不进行认证，或者采用 SASL 框架 Simple Authentication Security Layer，支持的认证机制如下。
- GSSAPI
- OATUTHBERAER
- PLAIN
- SCRAM

SASL 配置使用 JAAS Java Authentication and Authorization Service，主要用来指定账号信息。


## 准备工作

准备三台服务器，后续在研究如何新增扩展节点。

192.168.31.14 => node.id = 1
192.168.31.15 => node.id = 2
192.168.31.16 => node.id = 3

三台服务器端口分配情况：
- 9092 PLAINTEXT，不需要认证，明文传输。
- 9093 PLAINTEXT，不需要认证，明文传输，Controller 之间沟通使用。
- 9094 SASL_PLAINTEXT，需要认证，明文传输，认证机制为 SASL/SCRAM-SHA-512。

三服务器上安装 JDK。

## 开始安装

### 解压缩 Kafka，新建数据目录。
```shell
tar -zxvf kafka_2.13-3.6.1.tgz -C /usr/local/

cd /usr/local/kafka_2.13-3.6.1

mkdir kafka-files
```

三台服务器同时新增配置文件 `/etc/kafka/kafka_server_jaas.conf`
```
KafkaServer {
    org.apache.kafka.common.security.scram.ScramLoginModule required
    username="admin"
    password="admin123";
};
```

### 安装

**生成集群ID**，在 192.168.31.14 机器上生成集群ID，其实在任意一台机器上执行该命令都可以。
```shell
bin/kafka-storage.sh random-uuid
	FKjaueTKSqq_tiDIN9lxjw
```


**初始化数据目录**，在三台台机器上执行命令，进行集群元数据配置。-t 选项使用上面命令生成的集群 ID。
```shell
bin/kafka-storage.sh format --cluster-id FKjaueTKSqq_tiDIN9lxjw --config config/kraft/server.properties
```


**启动 Kafka**，在三台机器上执行。
```shell
export KAFKA_OPTS=-Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf
bin/kafka-server-start.sh -daemon config/kraft/server.properties
```


## 命令行客户端测试

首先连接 9092 端口进行测试，测试集群功能是否正常，然后创建 SCRAM 用户并配置用户连接信息。
之后连接 9094 端口进行测试，测试集群功能在使用 SASL/SCRAM-SHA-512 认证机制的情况下是否正常。

查看 Topic 列表，刚启动的集群应该为空。
```shell
bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

控制台生产者生产消息。
```shell
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic quickstart-events
```

控制台消费者消费消息。
```shell
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic quickstart-events --from-beginning
```

查看 SCRAM 用户列表，默认应该没有用户列表。
```shell
bin/kafka-configs.sh --bootstrap-server localhost:9092 --describe --entity-type users 
```

新增两个用户。
```shell
bin/kafka-configs.sh --bootstrap-server localhost:9092 --alter --entity-type users --entity-name admin --add-config 'SCRAM-SHA-512=[password=admin123]'
	
bin/kafka-configs.sh --bootstrap-server localhost:9092 --alter --entity-type users --entity-name jacky --add-config 'SCRAM-SHA-512=[password=jacky123]'
```

新增客户端配置文件 
`admin-jaas` 
```properties
sasl.mechanism=SCRAM-SHA-512
security.protocol=SASL_PLAINTEXT
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
  username="admin" \
  password="admin123";
```
`jacky-jaas`
```properties
sasl.mechanism=SCRAM-SHA-512
security.protocol=SASL_PLAINTEXT
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
  username="jacky" \
  password="jacky123";
```

查看 Topic 列表。
```shell
bin/kafka-topics.sh --bootstrap-server localhost:9094 --list --command-config admin-jaas
```

查看用户列表
```shell
bin/kafka-configs.sh --bootstrap-server localhost:9094 --describe --entity-type users --command-config admin-jaas
```

查看 jacky 用户信息
```
bin/kafka-configs.sh --bootstrap-server 192.168.31.14:9094 --describe --entity-type users --entity-name jacky
```

查看 topic 列表
```shell
bin/kafka-topics.sh --bootstrap-server localhost:9094 --list --command-config jacky-jaas
```

命令行生产者生产消息
```shell
bin/kafka-console-producer.sh --bootstrap-server localhost:9094 --topic quickstart-events --producer.config admin-jaas
```

命令行消费者消费消息
```shell
bin/kafka-console-consumer.sh --bootstrap-server 192.168.31.14:9094 --topic quickstart-events  --from-beginning --consumer.config admin-jaas
```
## Spring Boot Java 客户端测试

### 首先测试 9092 不需要认证的端口。

启动两个项目，kafka-consumer、kafka-producer。
两个项目的依赖如下。
```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-web</artifactId>  
    <version>2.6.10</version>  
</dependency>  
  
<dependency>  
    <groupId>org.springframework.kafka</groupId>  
    <artifactId>spring-kafka</artifactId>  
    <version>2.9.0</version>  
</dependency>
```

kafka-producer 配置文件如下 `application.yml`。
```yaml
server:  
  port: 17000  
  
spring:  
  kafka:  
    producer:  
      key-serializer: org.apache.kafka.common.serialization.StringSerializer  
      value-serializer: org.apache.kafka.common.serialization.StringSerializer  
    bootstrap-servers: 192.168.31.14:9092,192.168.31.15:9092,192.168.31.16:9092
```

kafka-producer 生产消息代码。
```java
@RestController  
@RequestMapping("producer")  
@SpringBootApplication  
public class App {  
    public static void main( String[] args )  
    {  
        SpringApplication.run(App.class, args);  
    }  
  
    private static final DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");  
  
    @Autowired  
    KafkaTemplate<String, Object> kafkaTemplate;  
  
  
    @GetMapping("timerStart")  
    public void timerStart() {  
        Executors.newScheduledThreadPool(1).scheduleAtFixedRate(() -> {  
            kafkaTemplate.send("timerStart", LocalDateTime.now().format(dtf));  
        }, 1, 5, TimeUnit.SECONDS);  
    }  
}
```

kafka-consumer 配置如下 `application.yml`。
```yaml
server:  
  port: 17001
  
spring:  
  kafka:  
    consumer:  
      key-serializer: org.apache.kafka.common.serialization.StringSerializer  
      value-serializer: org.apache.kafka.common.serialization.StringSerializer  
      group-id: kafka-consumer  
      auto-offset-reset: earliest  
    bootstrap-servers: 192.168.31.14:9092,192.168.31.15:9092,192.168.31.16:9092  
```

kakfa-consumer 消费消息代码。
```java
@Component  
public class TimerStartListener {  
  
    @KafkaListener(topics = {"timerStart"})  
    public void timerStartListener(ConsumerRecord<String, String> record) {  
        System.out.println("======> " + record.topic() + "," + record.partition() + ": " + record.value() );  
  
    }  
  
    @KafkaListener(topics = {"quickstart-events"})  
    public void quickStartEvents(ConsumerRecord<String, String> record) {  
        System.out.println("======> " + record.topic() + "," + record.partition() + ": " + record.value() );  
  
    }  
}
```

先启动生产者，然后调用接口触发消息生产，每隔 5 秒钟推送生产者的服务器时间。
```cmd
curl localhost:17000/producer/timerStart
```

然后启动消费者，观察控制台输出。

### 测试 9094 端口需要认证。

相对于 9092 端口的测试来说，只需要更新配置文件配置账号信息。

更新 kafka-producer 配置文件。
```yaml
server:  
  port: 17000  
  
spring:  
  kafka:  
    security:  
      protocol: SASL_PLAINTEXT  
    producer:  
      key-serializer: org.apache.kafka.common.serialization.StringSerializer  
      value-serializer: org.apache.kafka.common.serialization.StringSerializer  
    bootstrap-servers: 192.168.31.14:9094,192.168.31.15:9094,192.168.31.16:9094  
    properties:  
      security.protocol: SASL_PLAINTEXT  
      sasl.mechanism: SCRAM-SHA-512  
      sasl.jaas.config: org.apache.kafka.common.security.scram.ScramLoginModule required username="jacky" password="jacky123";
```

更新 kafka-consumer 配置文件。
```yaml
server:  
  port: 17001

spring:  
  kafka:  
    security:  
      protocol: SASL_PLAINTEXT  
    consumer:  
      key-serializer: org.apache.kafka.common.serialization.StringSerializer  
      value-serializer: org.apache.kafka.common.serialization.StringSerializer  
      group-id: kafka-consumer  
      auto-offset-reset: earliest  
    bootstrap-servers: 192.168.31.14:9094,192.168.31.15:9094,192.168.31.16:9094  
    properties:  
      security.protocol: SASL_PLAINTEXT  
      sasl.mechanism: SCRAM-SHA-512  
      sasl.jaas.config: org.apache.kafka.common.security.scram.ScramLoginModule required username="jacky" password="jacky123";  
```

QA:
1. `export KAFKA_OPTS=-Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf` 这条命令的作用是什么？环境变量指向的配置文件的作用是什么？