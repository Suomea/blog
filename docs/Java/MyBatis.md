MyBatis 官网：https://mybatis.org/mybatis-3/zh_CN/index.html

直接使用 JDBC 查询数据。

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