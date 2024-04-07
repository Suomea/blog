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

查看每个库数据占用的磁盘空间
```
select datname, pg_size_pretty (pg_database_size(datname)) AS size from pg_database;
```

查看库中每个表占用的磁盘空间
```
select relname, pg_size_pretty(pg_relation_size(relid)) from pg_stat_user_tables where schemaname='public' order by pg_relation_size(relid) desc;
```