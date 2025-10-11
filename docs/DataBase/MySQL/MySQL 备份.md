---
comments: true
tags:
  - MySQL
  - 备份
---
## 备份的要求

1. 在生产实践中，对于大数据来说，物理备份是必须的：逻辑备份太慢并且受到资源限制，从逻辑备份中恢复数据需要很长时间。对于较小的数据库，逻辑备份可以很好的胜任。
2. 保留多个备份集。
3. 定期从逻辑备份（或者物理备份）中抽取数据进行恢复测试。
4. 备份文件不要和 `MySQL` 服务器数据文件放在同一个磁盘、服务器、机房、城市。（一般软测的要求是 30km 外的另一个机房）

## mysqldump 逻辑备份

### mysqldump 命令

```shell
mysqldump --source-data=2 --single-transaction -h localhost -u root -p --all-databases | gzip > test.sql.gz
```

`--source-data=2`：导出二进制日志文件名称和位置信息，并且在导出文件中以注释的形式展示。

`--single-transaction`：导出过程在一个事务中。

`--all-databases`：导出全部数据库，会包含系统库。也可以使用 --databases tb1 tb2 指定导出数据库的名称。


导出一个数据库的数据。
```
mysqldump -u <user> -p <dbname> > dbname.sql
```

导出多个数据库的数据。
```
mysqldump -u <user> -p --databases db1 db2 db3 > db.sql
```

导出所有数据库的数据。
```
mysqldump -u <user> -p --all-databases > alldb.sql
```

导出指定表的数据。
```
mysqldump -u <user> -p <dbname> <tablename1> <tablename2> > tablename.sql
```

在数据中执行指定的 SQL 文件。
```
mysql -u <user> -p <db> < a.sql
```
### 创建连接凭证配置文件
MySQL 连接凭证单独放在文件一个文件中，置于 /data/database/db_name 目录下，文件名为 `.bak.cnf`，内容如下：
```text
[client]
user=root
password=pwd
```

设置文件的权限，只有自己可读（r:4）可写(w:2)：
```shell
chmod 600 .bak.cnf
```

### 备份脚本文件
脚本文件放置在 /data/database/db_name 目录下，文件名为 bak.sh，内容如下：
```shell
#!/bin/bash

source /etc/profile

number=31
backup_dir=/data/database/db_name
cre_file=/data/database/db_name/.bak.cnf
dd=`date +%Y-%m-%d-%H-%M-%S`
database_name=db_name


if [ ! -d $backup_dir ];
then
    mkdir -p $backup_dir;
fi

mysqldump --single_transaction --master-data=2 --defaults-file=$cre_file $database_name | gzip > $backup_dir/$database_name-$dd.gz

echo "dump file: $backup_dir/$database_name-$dd.gz" >> $backup_dir/dump.log

delfile=`ls -l -crt  $backup_dir/*.gz | awk '{print $9 }' | head -1`

count=`ls -l -crt  $backup_dir/*.gz | awk '{print $9 }' | wc -l`

if [ $count -gt $number ]
then
  rm $delfile
  echo "delete $delfile" >> $backup_dir/dump.log
fi
```

### crontab 配置
设置为每天凌晨 2 点执行，使用 `crontab -e` 命令，新增一行配置：
```text
0 2 * * * bash /data/database/db_name/bak.sh
```

## XtraBackup 物理备份
### 备份准备工作
APT 安装。
```shell
wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb 
dpkg -i percona-release_latest.bookworm_all.deb 
apt update 
apt install percona-xtrabackup-80
```

创建备份账号并授权。
```mysql
CREATE USER 'bkpuser'@'localhost' IDENTIFIED BY '123456'; 
GRANT BACKUP_ADMIN, PROCESS, RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO 'bkpuser'@'localhost'; 
GRANT SELECT ON performance_schema.log_status TO 'bkpuser'@'localhost';
GRANT SELECT ON performance_schema.keyring_component_status TO bkpuser@'localhost'; 
GRANT SELECT ON performance_schema.replication_group_members TO bkpuser@'localhost'; 
FLUSH PRIVILEGES;
```

### 本地直接全量物理备份

备份命令。备份的文件不是同一时刻状态一致的，因为备份程序运行需要时间并且运行过程中源文件可能会发生变化。192.168.31.13 服务器上执行。
```shell
xtrabackup --user=bkpuser --password=123456 \
	--backup \
	--target-dir=/data/bkps/
```
`--backup` 选项创建备份。
`--target-dir` 选项指定备份存放的目录。
`--compress` 选项开启压缩备份，能大量减少备份文件占用磁盘空间大小，默认使用 zstd 算法。`--compress-threads` 指定并行压缩的线程数量。关于压缩参考：[创建压缩备份](https://docs.percona.com/percona-xtrabackup/innovation-release/create-compressed-backup.html)

如果创建备份的时候使用了 `--compress` 选项，那么在进行 prepare 之前需要先进行 decomprss。
```
xtrabackup --decompress --target-dir=/data/bkps/
```
`--remove-original` 解压缩默认不会删除压缩文件，添加该选项删除原始压缩文件。

所以对于备份还原来说，需要先 prepare 备份文件，使数据达到 a single instant in time。192.168.31.13 服务器上执行（还原可以在任何服务器上执行，前提是安装 XtraBackup）。
```shell
xtrabackup --prepare --target-dir=/data/bkps
```

恢复之前先停止 Docker mysql，并删除 mysql 的数据文件。192.168.31.14 服务器上执行。
```shell
docker stop mysql
rm -rf mysql-data/*
```

同步文件到 Docker mysql 数据目录，一般情况下文件的恢复需要注意 Linux 文件属性问题。192.168.31.13 服务器上执行。
```shell
rsync -avrP /data/bkps/ root@192.168.31.14:/root/mysql-data/
```

启动 Docker mysql，验证数据是否恢复。

### 远程全量物理备份

在 192.168.31.14 服务器上执行备份任务，并且将备份文件存储到 192.168.31.14 服务器。所有命令均在 192.168.31.14 服务器执行。需要在 192.168.31.14 服务器安装 XtraBackup。

备份。
```shell
ssh root@192.168.31.13 "xtrabackup --user=bkpuser --password=123456  --compress --compress-threads=4 --backup --stream=xbstream" \
    > backup.xbstream \
    2> backup.log
```
`--stream` 压缩文件会以流的形式追加到标准输出，并且指定了流的格式。

停止 Docker mysql，并删除 Docker mysql 的数据目录。

解压缩
```shell
xbstream --extract < backup.xbstream -C /root/restore/ 
xtrabackup --decompress --remove-original --target-dir=/root/restore/
```

prepare
```shell
xtrabackup --prepare --target-dir=/root/restore/
```

还原
```shell
xtrabackup --copy-back --target-dir=/root/restore/ --datadir=/root/mysql-data
```

## QA

**MySQL 查询数据库中每个库占用的空间大小。**
```MySQL
select sum(DATA_LENGTH / 1024 / 1024 / 1024) as data, 'GB', TABLE_SCHEMA from information_schema.TABLES where TABLE_SCHEMA not in ('mysql', 'sys', 'performance_schema', 'information_schema') group by TABLE_SCHEMA order by data;
```


**MySQL 查看数据库中每个表占用的空间大小。**
```MySQL
select sum(DATA_LENGTH / 1024 / 1024 ) as data, 'MB', TABLE_SCHEMA, TABLE_NAME from information_schema.TABLES where TABLE_SCHEMA not in ('mysql', 'sys', 'performance_schema', 'information_schema') and TABLE_SCHEMA = 'yl_dyzx' group by TABLE_SCHEMA, TABLE_NAME order by TABLE_SCHEMA, data desc ;
```


**XtraBackup 备份还原数据中文乱码问题。**
需要配置所有数据库字符集相关的变量值一致。

**关于压缩备份的 `--compress` 和 `--remove-original` 选项效果。**
```
--  直接备份不压缩的备份目录
root@suomea03:~# ls -l /data/bkps/
total 70712
-rw-r----- 1 root root      447 Dec 17 00:40 backup-my.cnf
-rw-r----- 1 root root      157 Dec 17 00:40 binlog.000014
-rw-r----- 1 root root       16 Dec 17 00:40 binlog.index
-rw-r----- 1 root root     4287 Dec 17 00:40 ib_buffer_pool
-rw-r----- 1 root root 12582912 Dec 17 00:40 ibdata1
drwxr-x--- 2 root root     4096 Dec 17 00:40 mysql
-rw-r----- 1 root root 26214400 Dec 17 00:40 mysql.ibd
drwxr-x--- 2 root root     4096 Dec 17 00:40 performance_schema
drwxr-x--- 2 root root     4096 Dec 17 00:40 suomea_book
drwxr-x--- 2 root root     4096 Dec 17 00:40 sys
-rw-r----- 1 root root 16777216 Dec 17 00:40 undo_001
-rw-r----- 1 root root 16777216 Dec 17 00:40 undo_002
-rw-r----- 1 root root       18 Dec 17 00:40 xtrabackup_binlog_info
-rw-r----- 1 root root      134 Dec 17 00:40 xtrabackup_checkpoints
-rw-r----- 1 root root      480 Dec 17 00:40 xtrabackup_info
-rw-r----- 1 root root     2560 Dec 17 00:40 xtrabackup_logfile
-rw-r----- 1 root root       39 Dec 17 00:40 xtrabackup_tablespaces

-- 直接备份并且压缩的备份目录
root@suomea03:~# ls -l /data/bkps/
total 1584
-rw-r----- 1 root root     302 Dec 17 00:35 backup-my.cnf.zst
-rw-r----- 1 root root     112 Dec 17 00:35 binlog.000012.zst
-rw-r----- 1 root root      29 Dec 17 00:35 binlog.index.zst
-rw-r----- 1 root root     615 Dec 17 00:35 ib_buffer_pool.zst
-rw-r----- 1 root root    3960 Dec 17 00:35 ibdata1.zst
drwxr-x--- 2 root root    4096 Dec 17 00:35 mysql
-rw-r----- 1 root root 1364740 Dec 17 00:35 mysql.ibd.zst
drwxr-x--- 2 root root    4096 Dec 17 00:35 performance_schema
drwxr-x--- 2 root root    4096 Dec 17 00:35 suomea_book
drwxr-x--- 2 root root    4096 Dec 17 00:35 sys
-rw-r----- 1 root root  101826 Dec 17 00:35 undo_001.zst
-rw-r----- 1 root root   94040 Dec 17 00:35 undo_002.zst
-rw-r----- 1 root root      31 Dec 17 00:35 xtrabackup_binlog_info.zst
-rw-r----- 1 root root     134 Dec 17 00:35 xtrabackup_checkpoints
-rw-r----- 1 root root     326 Dec 17 00:35 xtrabackup_info.zst
-rw-r----- 1 root root     193 Dec 17 00:35 xtrabackup_logfile.zst
-rw-r----- 1 root root      52 Dec 17 00:35 xtrabackup_tablespaces.zst

-- 压缩备份解压缩后的备份目录，不加 --remove-original 选项
root@suomea03:~# ls -l /data/bkps/
total 72284
-rw-r--r-- 1 root root      447 Dec 17 00:36 backup-my.cnf
-rw-r----- 1 root root      302 Dec 17 00:35 backup-my.cnf.zst
-rw-r--r-- 1 root root      157 Dec 17 00:36 binlog.000012
-rw-r----- 1 root root      112 Dec 17 00:35 binlog.000012.zst
-rw-r--r-- 1 root root       16 Dec 17 00:36 binlog.index
-rw-r----- 1 root root       29 Dec 17 00:35 binlog.index.zst
-rw-r--r-- 1 root root     4287 Dec 17 00:36 ib_buffer_pool
-rw-r----- 1 root root      615 Dec 17 00:35 ib_buffer_pool.zst
-rw-r--r-- 1 root root 12582912 Dec 17 00:36 ibdata1
-rw-r----- 1 root root     3960 Dec 17 00:35 ibdata1.zst
drwxr-x--- 2 root root     4096 Dec 17 00:36 mysql
-rw-r--r-- 1 root root 26214400 Dec 17 00:36 mysql.ibd
-rw-r----- 1 root root  1364740 Dec 17 00:35 mysql.ibd.zst
drwxr-x--- 2 root root    12288 Dec 17 00:36 performance_schema
drwxr-x--- 2 root root     4096 Dec 17 00:36 suomea_book
drwxr-x--- 2 root root     4096 Dec 17 00:36 sys
-rw-r--r-- 1 root root 16777216 Dec 17 00:36 undo_001
-rw-r----- 1 root root   101826 Dec 17 00:35 undo_001.zst
-rw-r--r-- 1 root root 16777216 Dec 17 00:36 undo_002
-rw-r----- 1 root root    94040 Dec 17 00:35 undo_002.zst
-rw-r--r-- 1 root root       18 Dec 17 00:36 xtrabackup_binlog_info
-rw-r----- 1 root root       31 Dec 17 00:35 xtrabackup_binlog_info.zst
-rw-r----- 1 root root      134 Dec 17 00:35 xtrabackup_checkpoints
-rw-r--r-- 1 root root      521 Dec 17 00:36 xtrabackup_info
-rw-r----- 1 root root      326 Dec 17 00:35 xtrabackup_info.zst
-rw-r--r-- 1 root root     2560 Dec 17 00:36 xtrabackup_logfile
-rw-r----- 1 root root      193 Dec 17 00:35 xtrabackup_logfile.zst
-rw-r--r-- 1 root root       39 Dec 17 00:36 xtrabackup_tablespaces
-rw-r----- 1 root root       52 Dec 17 00:35 xtrabackup_tablespaces.zst

-- 压缩备份解压缩后的备份目录，加 --remove-original 选项
root@suomea03:~# ls -l /data/bkps/
total 70712
-rw-r--r-- 1 root root      447 Dec 17 00:38 backup-my.cnf
-rw-r--r-- 1 root root      157 Dec 17 00:38 binlog.000013
-rw-r--r-- 1 root root       16 Dec 17 00:38 binlog.index
-rw-r--r-- 1 root root     4287 Dec 17 00:38 ib_buffer_pool
-rw-r--r-- 1 root root 12582912 Dec 17 00:38 ibdata1
drwxr-x--- 2 root root     4096 Dec 17 00:38 mysql
-rw-r--r-- 1 root root 26214400 Dec 17 00:38 mysql.ibd
drwxr-x--- 2 root root     4096 Dec 17 00:38 performance_schema
drwxr-x--- 2 root root     4096 Dec 17 00:38 suomea_book
drwxr-x--- 2 root root     4096 Dec 17 00:38 sys
-rw-r--r-- 1 root root 16777216 Dec 17 00:38 undo_001
-rw-r--r-- 1 root root 16777216 Dec 17 00:38 undo_002
-rw-r--r-- 1 root root       18 Dec 17 00:38 xtrabackup_binlog_info
-rw-r----- 1 root root      134 Dec 17 00:37 xtrabackup_checkpoints
-rw-r--r-- 1 root root      521 Dec 17 00:38 xtrabackup_info
-rw-r--r-- 1 root root     2560 Dec 17 00:38 xtrabackup_logfile
-rw-r--r-- 1 root root       39 Dec 17 00:38 xtrabackup_tablespaces
```