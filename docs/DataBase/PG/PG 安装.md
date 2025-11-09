配置仓库：
```shell
# 创建目录保存公钥
install -d /usr/share/postgresql-common/pgdg

# 下载公钥
curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc

# 设置环境变量，/etc/os-release 包含 VERSION_CODENAME 环境变量
. /etc/os-release

# 设置 PostgreSQL 官方 APT 仓库源
sh -c "echo 'deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $VERSION_CODENAME-pgdg main' > /etc/apt/sources.list.d/pgdg.list"

# 更新包索引 
apt update
```

查看软件的信息：
```
apt show postgresql-18
```

安装：
```
apt install postgresql-18
```

PG 相关的进程：
```
postgres 1900217       1  0 00:00 ?        00:00:00 /usr/lib/postgresql/18/bin/postgres -D /var/lib/postgresql/18/main -c config_file=/etc/postgresql/18/main/postgresql.conf
postgres 1900218 1900217  0 00:00 ?        00:00:00 postgres: 18/main: io worker 0
postgres 1900219 1900217  0 00:00 ?        00:00:00 postgres: 18/main: io worker 1
postgres 1900220 1900217  0 00:00 ?        00:00:00 postgres: 18/main: io worker 2
postgres 1900221 1900217  0 00:00 ?        00:00:00 postgres: 18/main: checkpointer 
postgres 1900222 1900217  0 00:00 ?        00:00:00 postgres: 18/main: background writer 
postgres 1900224 1900217  0 00:00 ?        00:00:00 postgres: 18/main: walwriter 
postgres 1900225 1900217  0 00:00 ?        00:00:00 postgres: 18/main: autovacuum launcher 
postgres 1900226 1900217  0 00:00 ?        00:00:00 postgres: 18/main: logical replication launcher 

- PostgreSQL 18 主进程（由用户 `postgres` 启动）
- 启动参数：
    - `-D /var/lib/postgresql/18/main`  数据目录
    - `-c config_file=/etc/postgresql/18/main/postgresql.conf`  指定配置文件
- `io worker 0/1/2`  I/O worker 线程，用于异步写入/读取磁盘数据
- `checkpointer`  定期把脏页写入磁盘
- `background writer`  背景缓冲写入
- `walwriter`  WAL 日志写入进程
- `autovacuum launcher`  自动清理表/索引
- `logical replication launcher`  逻辑复制启动进程
```

本地登录进入数据库：
```
# 切换到系统 postgres 用户，使用 Unix socket 进行认证登录
su postgres
psql

# 设置 PG postgres 用户密码
alter user postgres with password 'xxxx'; 

# 然后使用 Linux root 用户进行操作，使用 TCP/IP socket 通信，使用 PG postgres 用户进行登录
psql -h 127.0.0.1 -p 5432 -U postgres
```

查看数据库列表：
```
\l
\list
select * from pg_database;
```

默认的连接配置：
```
# DO NOT DISABLE!
# If you change this first entry you will need to make sure that the
# database superuser can access the database using some other method.
# Noninteractive access to all databases is required during automatic
# maintenance (custom daily cronjobs, replication, and similar tasks).
#
# Database administrative login by Unix domain socket
local   all             postgres                                peer
# local 只允许 Unix socket 连接；all 所有的数据库；postgres PG 的管理用户；peer 使用操作系统的用户认证，同样是 postgres 用户

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# 本地的的系统系统用户也可以通过 Unix socket 登录，前提是 PG 和 Linux 都有对应的用户，使用的是 操作系统的用户认证

# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
# 本地 IPv4 密码认证登录

# IPv6 local connections:
host    all             all             ::1/128                 scram-sha-256
# 本地 IPv6 密码认证登录

# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            scram-sha-256
host    replication     all             ::1/128                 scram-sha-256
```

配置监听所有地址，然后配置所有用户都能通过网络连接：
```
/etc/postgresql/18/main/postgresql.conf
listen_address = '*'

/etc/postgresql/18/main/pg_hba.conf
host    all             all        192.168.31.0/24       scram-sha-256

重启服务
systemctl restart postgresql
```

配置密码复杂度校验配置：
```
编辑配置文件 /etc/postgresql/18/main/postgresql.conf
shared_preload_libraries = 'passwordcheck'              # (change requires restart)

重启服务
systemctl restart postgresql

确认 passwordcheck 扩展已加载
SHOW shared_preload_libraries;

默认长度最小为 8，需要同时包含字母和非字母，密码不能包含用户名

```

配置密码过期时间：
```sql
-- 查看用户列表，其中包含过期时间字段
SELECT * FROM pg_catalog.pg_user;

-- 设置密码过期时间
ALTER ROLE myuser VALID UNTIL '2025-02-08';
```

