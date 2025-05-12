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

## 查询更新许可信息
查看许可信息，EXPIRED_DATE 为空说明永久有效。
```sql
select * from v$license
```

查看版本信息。
```sql
select * from v$instance;
```

更新许可文件 dm.key
```
进入到 DM 安装目录的 bin 目录
dmdbms/bin

备份原有的 key
mv dm.key dm.key.bak

导入新的 key 到 bin 目录下
mv new.key dm.key

更新文件属性
chown dmdba:dmdba dm.key

使用 SYSDBA 登录达梦，执行更新许可操作
SP_LOAD_LIC_INFO();

查询许可信息
select * from v$license
```
