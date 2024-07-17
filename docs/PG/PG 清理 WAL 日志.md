---
comments: true
---
## PG 清理 WAL 日志

如果 PG WAL 日志占用过多磁盘空间可以手动清理。

先找到 PG 进程
```
ps -ef | grep postgres
ls -l /proc/pg_pid
```

进入 pg 命令所在的目录
```
./pg_controldata /var/lib/postgresql/13/main/
```

输出
```
Latest checkpoint location: 7EE/3A9CFE0

Latest checkpoint's REDO location: 7EE/2B0E5F0

Latest checkpoint's REDO WAL file: 00000001000007EE00000002

```

清除 00000001000007EE00000002 之前的文件，注意有个空格在 wal 存储路径后面。
```
./pg_archivecleanup -d /var/lib/postgresql/13/main/pg_wal/ 00000001000007EE00000002
```

## PG 查看数据库和表磁盘空间占用

查看每个库数据占用的磁盘空间
```
select datname, pg_size_pretty (pg_database_size(datname)) AS size from pg_database;
```

查看库中每个表占用的磁盘空间
```
select relname, pg_size_pretty(pg_relation_size(relid)) from pg_stat_user_tables where schemaname='public' order by pg_relation_size(relid) desc;
```

## pg_dump 导出数据

pg_dump 命令的用法
```
pg_dump 把一个数据库转储为纯文本文件或者是其它格式.

用法:
  pg_dump [选项]... [数据库名字]

一般选项:
  -f, --file=FILENAME          输出文件或目录名
  -F, --format=c|d|t|p         输出文件格式 (定制, 目录, tar
                               明文 (默认值))
  -j, --jobs=NUM               执行多个并行任务进行备份转储工作
  -v, --verbose                详细模式
  -V, --version                输出版本信息，然后退出
  -Z, --compress=0-9           被压缩格式的压缩级别
  --lock-wait-timeout=TIMEOUT  在等待表锁超时后操作失败
  --no-sync                    不用等待变化安全写入磁盘
  -?, --help                   显示此帮助, 然后退出

控制输出内容选项:
  -a, --data-only              只转储数据,不包括模式
  -b, --blobs                  在转储中包括大对象
  -B, --no-blobs               排除转储中的大型对象
  -c, --clean                  在重新创建之前，先清除（删除）数据库对象
  -C, --create                 在转储中包括命令,以便创建数据库
  -E, --encoding=ENCODING      转储以ENCODING形式编码的数据
  -n, --schema=PATTERN         dump the specified schema(s) only
  -N, --exclude-schema=PATTERN do NOT dump the specified schema(s)
  -O, --no-owner               在明文格式中, 忽略恢复对象所属者
  -s, --schema-only            只转储模式, 不包括数据
  -S, --superuser=NAME         在明文格式中使用指定的超级用户名
  -t, --table=PATTERN          dump the specified table(s) only
  -T, --exclude-table=PATTERN  do NOT dump the specified table(s)
  -x, --no-privileges          不要转储权限 (grant/revoke)
  --binary-upgrade             只能由升级工具使用
  --column-inserts             以带有列名的INSERT命令形式转储数据
  --disable-dollar-quoting     取消美元 (符号) 引号, 使用 SQL 标准引号
  --disable-triggers           在只恢复数据的过程中禁用触发器
  --enable-row-security        启用行安全性（只转储用户能够访问的内容）
  --exclude-table-data=PATTERN do NOT dump data for the specified table(s)
  --extra-float-digits=NUM     覆盖extra_float_digits的默认设置
  --if-exists                  当删除对象时使用IF EXISTS
  --include-foreign-data=PATTERN
                               include data of foreign tables on foreign
                               servers matching PATTERN
  --inserts                    以INSERT命令，而不是COPY命令的形式转储数据
  --load-via-partition-root    通过根表加载分区
  --no-comments                不转储注释
  --no-publications            不转储发布
  --no-security-labels         不转储安全标签的分配
  --no-subscriptions           不转储订阅
  --no-synchronized-snapshots  在并行工作集中不使用同步快照
  --no-tablespaces             不转储表空间分配信息
  --no-unlogged-table-data     不转储没有日志的表数据
  --on-conflict-do-nothing     将ON CONFLICT DO NOTHING添加到INSERT命令
  --quote-all-identifiers      所有标识符加引号，即使不是关键字
  --rows-per-insert=NROWS      每个插入的行数；意味着--inserts
  --section=SECTION            备份命名的节 (数据前, 数据, 及 数据后)
  --serializable-deferrable    等到备份可以无异常运行
  --snapshot=SNAPSHOT          为转储使用给定的快照
  --strict-names               要求每个表和(或)schema包括模式以匹配至少一个实体
  --use-set-session-authorization
                               使用 SESSION AUTHORIZATION 命令代替
                               ALTER OWNER 命令来设置所有权

联接选项:
  -d, --dbname=DBNAME      对数据库 DBNAME备份
  -h, --host=主机名        数据库服务器的主机名或套接字目录
  -p, --port=端口号        数据库服务器的端口号
  -U, --username=名字      以指定的数据库用户联接
  -w, --no-password        永远不提示输入口令
  -W, --password           强制口令提示 (自动)
  --role=ROLENAME          在转储前运行SET ROLE

如果没有提供数据库名字, 那么使用 PGDATABASE 环境变量的数值.
```

## PG 日志占用磁盘过高处理，案例

磁盘占用过高，排查到 PG 数据存储目录占用很高。
```
# pwd
/var/lib/pgsql/13/data
# du -sh ./*
62G	    ./base
4.0K	./current_logfiles
620K	./global
54G	    ./log
```

base 目录存放的是数据。
log 目录存放的是 PG 错误日志，PG 配置开启了错误日志记录。

查看错误日志目录，发现错误日志每天都很大。
```
# ls -lh
total 54G
-rw------- 1 postgres postgres 8.1G Apr  6 00:00 postgresql-Fri.log
-rw------- 1 postgres postgres 8.0G Apr  8 23:59 postgresql-Mon.log
-rw------- 1 postgres postgres 8.1G Apr  6 23:59 postgresql-Sat.log
-rw------- 1 postgres postgres 8.1G Apr  7 23:59 postgresql-Sun.log
-rw------- 1 postgres postgres 8.0G Apr  4 23:59 postgresql-Thu.log
-rw------- 1 postgres postgres 5.2G Apr  9 15:31 postgresql-Tue.log
-rw------- 1 postgres postgres 8.1G Apr  4 00:00 postgresql-Wed.log
```

查看日志内容，基本上都是重复的该条错误日志。后续和相关的业务同事了解到，业务上每 5 秒查询 GPS 数据直接插入到数据库，重复的数据没有任何处理，直接插入，导致错误日志过多。
```
2024-04-09 15:40:08.396 CST [2255299] 错误:  重复键违反唯一约束"xxx_xxxxxx"
2024-04-09 15:40:08.396 CST [2255299] 详细信息:  键值"xxxxxxxxxxxxxxxx" 已经存在
```

解决办法。
示例表
```sql
CREATE TABLE test ( 
	id SERIAL PRIMARY KEY, 
	name VARCHAR(100), 
	age INT, 
	CONSTRAINT unique_name_age UNIQUE (name, age) 
);
```

插入数据的时候使用 on conflict do nothing 语句来指定出现冲突时什么也不做。
```sql
INSERT INTO test(name, age) VALUES ('John', 25) ON CONFLICT (name, age) DO NOTHING;
```