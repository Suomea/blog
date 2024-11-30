## 安裝达梦 Jar 包
Maven 依赖
```xml
<dependency>  
    <groupId>com.dm</groupId>  
    <artifactId>DmJdbcDriver18</artifactId>  
    <version>18</version>  
</dependency>
```

安装 Jar 包
```
mvn install:install-file -Dfile=.\DmJdbcDriver18.jar -DgroupId=com.dm -DartifactId=DmJdbcDriver18 -Dversion=18 -Dpackaging=jar 
```

## 查询信息
查看许可信息，EXPIRED_DATE 为空说明永久有效。
```sql
select * from v$license
```

查看版本信息。
```sql
select * from v$instance;
```
## 达梦默认的角色
