## 常用命令
查看版本号
```sql
select version();
-- PostgreSQL 18.0 (Debian 18.0-1.pgdg12+3) on x86_64-pc-linux-gnu, compiled by gcc (Debian 12.2.0-14+deb12u1) 12.2.0, 64-bit

show server_version;
-- 18.0 (Debian 18.0-1.pgdg12+3)
```

查看数据库列表
```sql
SELECT * FROM pg_database;
```

查看角色列表
```sql
SELECT * FROM pg_roles;
```
## PG 介绍
数据库是最顶层的容器，一个 PG 实例里可以有多个数据库。

模式是数据库里的逻辑分组空间，一个数据库可以有多个模式。模式里面存放表、视图、函数，数据库默认包含 public 模式。

PG 数据只有角色，角色是权限的集合，可以继承其他角色。角色也可以设置属性：登陆、管理员、创建数据库等等。

创建用户和库
```sql
CREATE USER vaultwarden WITH ENCRYPTED PASSWORD 'yourpassword';
CREATE DATABASE vaultwarden OWNER vaultwarden;
```

删除数据库
```sql
-- 断开连接
SELECT pg_terminate_backend(pid) FROM pg_stat_activity
WHERE datname = 'vaultwarden'
AND pid <> pg_backend_pid();

-- 删除数据库
drop database vaultwarden;

-- 删除用户
drop role vaultwarden;
```

创建一张表
```sql
create table person (
	id bigserial primary key,
	name varchar(20),
	age int,
	create_time timestamp default current_timestamp
);

comment on table person is '人员信息表';
comment on column person.name is '姓名';
comment on column person.age is '年龄';
comment on column person.create_time is '创建时间';
```