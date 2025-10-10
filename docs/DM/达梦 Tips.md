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

查询表空间：
```sql
SELECT * FROM V$TABLESPACE; 
SELECT * FROM DBA_DATA_FILES;
```

创建表空间，设置初始大小 128M，设置自动扩展，每次 100M，上限 10240M：
```sql
create tablespace "TEST" datafile '/data/dmdata/DAMENG/TEST.DBF' size 128 autoextend on next 100 maxsize 10240;
```

更新表空间，设置自动扩展，每次 100M，上限 10240M：
```sql
alter tablespace "TEST" datafile '/data/dmdata/DAMENG/TEST.DBF' autoextend on next 100 maxsize 10240;
```

创建用户，并赋予角色：
```sql
create user "TEST" identified by "Dameng@123" hash with SHA512 salt 
encrypt by "123456" 
default tablespace "TEST" 
default index tablespace "TEST"; 

grant "PUBLIC","SOI","DBA","RESOURCE" to "TEST";
```

## 达梦排序 NULL 值处理
达梦使用 order by 字段排序的时候，将字段值为 null 的数据永远排在最后。使用 nulls last|first。
```sql
select * from user order by age desc nulls last;
```

使用 case when 进行处理，首先将 null 设置为 1，非 null 设置为 0 正序排列，这样就将 null 值排在最后了，然后再按字段值排序。
```sql
select * from user order by CASE WHEN age IS NULL THEN 1 ELSE 0 END, age desc
```

可以在 dm.ini 中进行配置 ORDER_BY_NULLS_FLAG 参数，默认值为 0。来控制 NULL 值排序位置，但是注意该参数无法将 NULL 值始终在最后面返回。

- 0 表示 NULL 值始终在最前面返回；
- 1 表示 ASC 升序排序时 NULL 值在最后返回，DESC 降序排序时 NULL 值在最前面返回，在参数等于 1 的情况下，NULL 值的返回与 ORACLE 保持一致；
- 2 表示 ASC 升序排序时 NULL 值在最前面返回，DESC 降序排序时 NULL 值在最后返回，在参数等于 2 的情况下，NULL 值的返回与 MYSQL 保持一致；
- 3 表示在取值为 1 的基础上，将空串置于 NULL 值和非空值之间
```sql
SELECT * FROM V$PARAMETER WHERE name = 'ORDER_BY_NULLS_FLAG';

ALTER SYSTEM SET 'ORDER_BY_NULLS_FLAG' = 2;
```





