---
comments: true
tags:
  - MySQL
---
### `count` 函数的注意事项

表结构定义如下：

```sql
create table test_tb
(
    id           int primary key auto_increment,
    auto_level   int null comment '自动分级',
    manual_level int null comment '手动分级'
);
```

只要查询出一共有多少条数据，再查询出手动分级和自动分级不一致的数据即可。  

错误的 SQL：
```sql
select count(*)                          as totalCount,
       count(manual_level != auto_level) as diffCount
from test_tb;
```

错误的原因在于 `manual_level != auto_level` 逻辑表达式，无论表达式返回的结果是 `True` 或者 `False` 都会被 `count()` 函数统计进去。
!!! note "如果表达式两边至少一方为 NULL，那么表达式的结果就是 NLL，count 函数不会统计 NULL。"

正确的 SQL：
```sql
select count(*)                                                      as totalCount,
       sum(case when (manual_level != auto_level) then 1 else 0 end) as diffCount
from test_db;
```

### `find_in_set` 函数的用法

用法 `find_in_set(find_str, field_name)`

`find_str` 为要查找的字符串。

`field_name` 为字段名，需要以英文的逗号分隔。

示例，例如 `user` 表的 `tag` 字段值为 `学生,成年人,男,中国人`。

查询 SQL：
```sql
select * from user where find_in_set('中国人', tag);
```

Mybatis Plus 中常见的用法：
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