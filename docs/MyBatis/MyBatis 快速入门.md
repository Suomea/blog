MyBatis 官网：https://mybatis.org/mybatis-3/zh_CN/index.html

## 准备数据
DDL
```sql
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `username` varchar(100) NOT NULL COMMENT '姓名',
  `age` int NOT NULL COMMENT '年龄',
  `create_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '数据创建时间'
) ENGINE=InnoDB COMMENT='用户信息表';

INSERT INTO `user` 
VALUES 
	('jacky',22,'2025-02-14 10:52:01'),
	('otto',18,'2025-02-14 10:52:01'),
	('zhangsan',25,'2025-02-14 10:52:01');
```

实体类
```java
package com.otto.mybatis.entity;

import java.time.LocalDateTime;

public class User {

    private String username;

    private Integer age;

    private LocalDateTime createAt;

	// ...
}

```
## 直接使用 JDBC 查询数据
添加依赖
```
<dependency>  
    <groupId>com.mysql</groupId>  
    <artifactId>mysql-connector-j</artifactId>  
    <version>8.0.33</version>  
</dependency>
```

代码
```java
public class JDBCTest {
    public static void main(String[] args) {
        String url = "jdbc:mysql://192.168.199.129:3306/test";
        String user = "test";
        String password = "xxxx";

        try (Connection conn = DriverManager.getConnection(url, user, password);
             PreparedStatement pstmt = conn.prepareStatement("SELECT * FROM user WHERE username = ?")) {
            pstmt.setString(1, "jacky"); // 设置参数值
            ResultSet rs = pstmt.executeQuery();
            while (rs.next()) {
                String username = rs.getString("username");
                Integer age = rs.getInt("age");
                System.out.println("username: " + username + ", age: " + age);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

## 使用 MyBatis
添加依赖
```xml
<dependency>  
    <groupId>com.mysql</groupId>  
    <artifactId>mysql-connector-j</artifactId>  
    <version>8.0.33</version>  
</dependency>  
  
<dependency>  
    <groupId>org.mybatis</groupId>  
    <artifactId>mybatis</artifactId>  
    <version>3.5.19</version>  
</dependency>
```

添加 MyBatis 配置文件 /resourse/mybatis-config.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--配置打印 SQL 到标准输出-->
    <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://192.168.199.129:3306/test"/>
                <property name="username" value="xxx"/>
                <property name="password" value="xxxxxx"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="mapper/UserMapper.xml"/>
    </mappers>
    
</configuration>
```

创建 SQL xml 文件，/resource/mapper/UserMapper.xml。一般情况下不建议在 SQL 末尾增加分号
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="UserMapper">
    <select id="selectAll" resultType="com.otto.mybatis.entity.User">
        select username, age, create_at as createAt from user where username = #{username} and age = #{age}
    </select>
</mapper>
```

查询数据代码
```java

public class MyBatisTest {
    public static void main(String[] args) throws IOException {
        // XML 中构建 SqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        // 从 SqlSessionFactory 中获取 SqlSession
        try(SqlSession sqlSession = sqlSessionFactory.openSession()) {

            Map<String, Object> params = new HashMap<>();
            params.put("username", "jacky");
            params.put("age", 22);
            List<User> userList = sqlSession.selectList("UserMapper.selectAll", params);
            System.out.println("query result: ==> " + userList);
        }
    }
}
```

> Tips:   
> SqlSessionFactory 实例的作用域应该是全局的，且为单例。  
> SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。SqlSession 的关闭很重要，为了确保关闭可以使用 AutoCloseable 进行关闭。
>
> 示例代码中参数如果有多个占位符参数的话，可以使用 Map 传递参数，使用占位符名称作为键，实际的参数作为值。
>
> 示例代码只是查询操作，所以不需要进行 commit 或者 rollback 操作。如果进行数据修改，需要手动 commit 并且捕获异常，在异常处理代码块中进行 rollback。 
> 
> 也可以在获取 SqlSession 的时候设置自动提交 sqlSessionFactory.openSession(true)，那么每个 sql 都会当成一个事务并自动提交，因此也不需要进行手动回滚。如果你有一连串的操作需要事务执行，那么还是手动提交和处理回滚逻辑比较好。

### 使用 Mapper 代理开发
配置 Maven 打包代码包里面的 xml 文件。
```xml
    <build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
            </resource>

            <resource>
                <directory>src/main/java</directory>
                <filtering>true</filtering>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
        </resources>
    </build>
```

修改 MyBatis 配置文件，增加扫描目录。
```xml
<mappers>  
    <mapper resource="mapper/UserMapper.xml"/>  
    <package name="com.otto.mybatis.mapper"/>  
</mappers>
```

新增 com.otto.mybatis.mapper.UserMapper.java 文件
```java
public interface UserMapper {  
    List<User> selectAll(@Param("username") String username, @Param("age") Integer age);  
}
```

新增 com.otto.mybatis.mapper.UserMapper.xml 文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE mapper  
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">  
<mapper namespace="com.otto.mybatis.mapper.UserMapper">  
    <select id="selectAll" resultType="com.otto.mybatis.entity.User">  
        select username, age, create_at as createAt from user where username = #{username} and age = #{age}  
    </select>  
</mapper>
```

测试使用 UserMapper 代理执行 SQL。
```java
UserMapper userMapper = sqlSession.getMapper(UserMapper.class);  
List<User> userList = userMapper.selectAll("jacky", 22);  
System.out.println("query result: ==> " + userList);
```

### 不使用 XML 构建 SqlSessionFactory
使用单例模式构建 SqlSessionFactory
```java
public class MyBatisConfig {  
  
    private static SqlSessionFactory FACTORY_INSTANCE = null;  
  
    public static SqlSessionFactory sqlSessionFactory() {  
        if (FACTORY_INSTANCE == null) {  
            synchronized (MyBatisConfig.class) {  
                if (FACTORY_INSTANCE == null) {  
                    // 配置数据源  
                    PooledDataSource dataSource = new PooledDataSource();  
                    dataSource.setDriver("com.mysql.cj.jdbc.Driver");  
                    dataSource.setUrl("jdbc:mysql://192.168.199.129:3306/test_db");  
                    dataSource.setUsername("xxx");  
                    dataSource.setPassword("xxxxx");  
                    // 配置事务管理器  
                    JdbcTransactionFactory transactionFactory = new JdbcTransactionFactory();  
                    // 配置环境  
                    Environment environment = new Environment("development", transactionFactory, dataSource);  
                    // 配置 MyBatis                    
                    Configuration configuration = new Configuration(environment);  
                    // 设置日志实现为标准输出  
                    configuration.setLogImpl(org.apache.ibatis.logging.stdout.StdOutImpl.class);  
                    // 如果 Mapper 接口都在同一个包下，可以使用包扫描  
                    configuration.addMappers("com.otto.mybatis.mapper");  
                    // 构建 SqlSessionFactory                    
                    FACTORY_INSTANCE =  new SqlSessionFactoryBuilder().build(configuration);  
                }  
            }  
        }  
        return FACTORY_INSTANCE;  
    }  
}
```

进一步简化调用代码
```java
public class MyBatisTest {
    public static void main(String[] args) throws IOException {
        // 从 SqlSessionFactory 中获取 SqlSession
        try(SqlSession sqlSession = MyBatisConfig.sqlSessionFactory().openSession()) {

            UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

            List<User> userList = userMapper.selectAll("jacky", 22);
            System.out.println("query result: ==> " + userList);
        }
    }
}
```