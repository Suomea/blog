## Jaskson
Spring Boot 默认使用 Jackson 进行序列化和反序列化。
```
- jackson-core：提供 JSON 解析和生成的底层支持。
- jackson-databind：提供 Java 对象与 JSON 的转换。
- jackson-annotations：提供用于序列化和反序列化的注解。
- jackson-datatype-jsr310：支持 Java 8 时间 API。
```

代码结构
```
├── lib
│   ├── jackson-annotations-2.19.0.jar
│   ├── jackson-core-2.19.0.jar
│   ├── jackson-databind-2.19.0.jar
│   └── jackson-datatype-jsr310-2.19.0.jar
└── src
    └── com
        └── jacky
            ├── App.java
            └── entity
                └── User.java
```

`App.java` 代码
```java
public class App {  
    public static void main(String[] args) throws JsonProcessingException {  
        User user = new User();  
  
        user.setId(1);  
        user.setName("jacky");  
        user.setBirthday(LocalDate.now());  
        user.setLastLogin(new Date());  
        user.setCreateTime(LocalDateTime.now());  
        user.setDeleted(false);  
  
        ObjectMapper om = new ObjectMapper();  
        om.registerModule(new JavaTimeModule()); // 注册该模块以支持 LocalDate 和 LocalDateTime 类型  
        System.out.printf(om.writeValueAsString(user));  
    }  
}
```

输出
```
{"id":1,"name":"jacky","birthday":[2025,5,15],"lastLogin":1747321286831,"createTime":[2025,5,15,23,1,26,831291000],"deleted":false}
```

实体 `DTEntity` 包含日期时间属性：
```java
public class DTEntity {  
  
    private Date testDate;  
  
    private LocalDate testLocalDate;  
  
    private LocalDateTime testLocalDateTime;
}
```

默认序列化格式
```json
{
  "testDate": "2025-05-13T14:31:20.137+00:00",
  "testLocalDate": "2025-05-13",
  "testLocalDateTime": "2025-05-13T22:31:20.137036"
}
```

Date 类型全局配置
```properties
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss  
spring.jackson.time-zone=GMT+8
```

效果
```json
{
  "testDate": "2025-05-13 22:37:22",
  "testLocalDate": "2025-05-13",
  "testLocalDateTime": "2025-05-13T22:37:22.337772"
}
```

