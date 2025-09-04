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

## 用户、模式、表空间
表空间是一个逻辑概念，由一个或者多个物理数据库文件组成。数据库中所有的对象都存储在表空间对应的数据文件中。

查询表空间：
```sql
SELECT * FROM V$TABLESPACE; 
SELECT * FROM DBA_DATA_FILES;
```

默认的表空间：
- SYSTEM，存储数据字典、系统元数据。
- MAIN，用户默认的表空间。
- TEMP，临时表空间，用户排序操作、临时表等临时数据存储。
- ROLL，回滚表空间，存储事务回滚数据。
- HMAIN，HUGE 表的默认表空间，用户存储列式存储的大数据表。

自定义表空间，用户自己创建的表空间，用来存储用户自己的表数据、索引数据等。

表空间的意义在于物理的磁盘可以进行逻辑上的划分为不通的表空间，对表空间进行管理（容量限制/扩容，备份，权限控制）。

达梦创建用户的时候会默认创建一个同名的模式，用户对该模式拥有全部的权限，不需要额外授权。

如果创建用户的时候没有设置默认表空间，那么用户默认使用 MAIN 表空间。如果设置了默认的自定义表空间，那么该用户的默认数据都存储在自定义表空间中。

用户创建表和索引的时候都可以单独指定表空间，前提是用户对指定的表空间有权限。

模式只是一个存粹的逻辑概念，没有存储属性，不关联表空间。但是模式是权限管理的核心单元。

如果用户对模式中的表有读写权限，但是对模式中的表所属的表空间没有配额，那么不影响用户对模式中已经存在表进行读写。
