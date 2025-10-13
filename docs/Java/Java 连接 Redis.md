使用 Redisson 连接 Redis。

依赖：
```xml
<dependency>  
    <groupId>org.redisson</groupId>  
    <artifactId>redisson</artifactId>  
    <version>3.52.0</version>  
</dependency>
```

代码：
```java
private static final Logger log = LoggerFactory.getLogger(App.class);  
  
public static void main(String[] args) {  
    Config config = new Config();  
    // redis 连接配置信息
    config.useSingleServer()  
            .setAddress("redis://192.168.31.157:6379")  
            .setDatabase(1)  
            .setPassword("xxxx");  
    
    // 设置 key String 序列化，value JSON 序列化
	config.setCodec(new CompositeCodec(StringCodec.INSTANCE,   
        new JsonJacksonCodec(),  
        new JsonJacksonCodec()));
  
    // 创建客户端
    RedissonClient redissonClient = Redisson.create(config);  
    
    // 字符串类型测试
    RBucket<Map> name = redissonClient.getBucket("people");
    Map map = new HashMap();
    map.put("name", "jacky");
    map.put("age", "22");
    name.set(map);
    log.info("hello name: {}", name.get());

	// 列表类型测试
    RList<String> list = redissonClient.getList("pet");
    list.add("dog");
    list.add("bird");
    log.info("hello ped: {}", list.readAll());  
  
    redissonClient.shutdown();  
}

// Redis 存储内容
// 127.0.0.1:6379[1]> get people
// "{\"@class\":\"java.util.HashMap\",\"name\":\"jacky\",\"age\":\"22\"}"
// 127.0.0.1:6379[1]> lrange pet 0 -1
// 1) "\"dog\""
// 2) "\"bird\""
```