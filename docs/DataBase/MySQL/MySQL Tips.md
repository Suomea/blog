---
comments: true
tags:
  - MySQL
---
### `count` 函数的注意事项

`count(*)` 会统计所有行；`count(expr)` 函数只会统计 `expr` 非 NULL 的行。

`count(a != b)` 如果 a 和 b 其中一个为 NULL 或两个都为 NULL，表达式 `a != b` 就为 NULL；否则表达式的值就是 TRUE/FALSE，非 NULL。

`sum(case when (a != b) then 1 else 0 end)` 用来统计 a 和 b 都不为 NULL 且 a 和 b 不相等的数量。

### `find_in_set` 函数的用法

用法 `find_in_set(find_str, field_name)`，`find_str` 为要查找的字符串，`field_name` 为字段名，需要以英文的逗号分隔。

示例查询 SQL：
```sql
select * from user where find_in_set('中国人', tag);
```

`Mybatis Plus` 中的用法：
```java
Wrappers.<User>lambdaQuery()
	.apply("find_in_set({0}, tag)", "中国人");
```

### `root` 用户忘记密码

停止 MySQL 服务，编辑配置文件增加选项 `skip-grant-tables` 启动 MySQL 服务。 

直接进入 MySQL，命令 `mysql`。
```sql
update mysql.user set authentication_string = null where user = 'root';
flush privileges;
exit;
```

再次进入 MySQL，命令  `mysql -u root `。
```
alter user root@localhost identified  by 'new_password';
flush privileges;
exit;
```

去掉增加的配置，重启 MySQL 服务，完成。

参考：https://stackoverflow.com/questions/50691977/how-to-reset-the-root-password-in-mysql-8-0-11

### 错误修复 `Too many connections`

调整 max_connections 系统变量，重启数据库。  

参考：https://dev.mysql.com/doc/refman/8.3/en/server-system-variables.html#sysvar_max_connections

### 用户操作
查询所有的用户
```
select user, host from mysql.user;
```

创建新的用户
```
create user 'test'@'%' identified by 'xxxx';
```

查询用户权限
```
show grants for username;
```

赋予用户查询数据库的权限
```
GRANT SELECT ON mydb.* TO 'username'@'%';
-- 撤销
REVOKE SELECT ON mydb.* TO 'username'@'%';
```

赋予用户数据库的全部权限
```
GRANT ALL PRIVILEGES ON mydb.* TO 'username'@'%';
-- 撤销
REVOKE ALL PRIVILEGES ON mydb.* FROM 'username'@'%';
```

确保权限立即生效
```
FLUSH PRIVILEGES;
```

