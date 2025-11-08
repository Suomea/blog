## 备份的要求
对于大数据来说物理备份是必须的：逻辑备份太慢并且受到资源限制，从逻辑备份中恢复数据需要很长时间。对于较小的数据库，逻辑备份可以很好的胜任。

遵循 3-2-1 原则。至少 3 分数据副本，通常是一份原始数据和两份备份数据；至少 1 个异地备份，严格的要求是至少 30km 以外的另一个城市；2 可以简单理解为存储在两块不同的设备上。

定期从备份中抽取数据进行恢复测试，确保备份流程没有问题，以及能够在发生故障时能被成功恢复。

## mysqldump 逻辑备份

### mysqldump 命令
命令用法：
```shell
mysqldump --source-data=2 --single-transaction -h localhost -u root -p --all-databases | gzip > test.sql.gz

--source-data=2：导出二进制日志文件名称和位置信息，并且在导出文件中以注释的形式展示。
--single-transaction：导出过程在一个事务中。
--all-databases：导出全部数据库，会包含系统库。也可以使用 --databases tb1 tb2 指定导出数据库的名称。
--ignore-table：忽略指定的表，需要指定数据库名：--ignore-table=database_name.table_name。
```

导出指定表的数据：
```
mysqldump -u <user> -p <dbname> <tablename1> <tablename2> > tablename.sql
```

在数据库执行指定的 SQL 文件：
```
mysql -u <user> -p <db> < a.sql
```
### 连接凭证文件
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
备份软件安装：
```shell
wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb 
dpkg -i percona-release_latest.bookworm_all.deb 
apt update 
apt install percona-xtrabackup-80
```

创建备份账号并授权：
```mysql
CREATE USER 'bkpuser'@'localhost' IDENTIFIED BY '123456'; 
GRANT BACKUP_ADMIN, PROCESS, RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO 'bkpuser'@'localhost'; 
GRANT SELECT ON performance_schema.log_status TO 'bkpuser'@'localhost';
GRANT SELECT ON performance_schema.keyring_component_status TO bkpuser@'localhost'; 
GRANT SELECT ON performance_schema.replication_group_members TO bkpuser@'localhost'; 
FLUSH PRIVILEGES;
```
### 本地直接全量物理备份
**备份**命令：
```shell
xtrabackup --user=bkpuser --password=123456 \
	--backup \
	--target-dir=/data/bkps/

--backup 选项创建备份。
--target-dir 选项指定备份存放的目录。
--compress 选项开启压缩备份，能大量减少备份文件占用磁盘空间大小，默认使用 zstd 算法。
--compress-threads 指定并行压缩的线程数量。
```

备份的文件不是同一时刻状态一致的，因为备份程序运行需要时间并且运行过程中源文件可能会发生变化。

**解压缩**，如果创建备份的时候使用了 `--compress` 选项，那么在进行 prepare 之前需要先进行 decomprss。
```
xtrabackup --decompress --target-dir=/data/bkps/

--remove-original 解压缩默认不会删除压缩文件，添加该选项删除原始压缩文件。
```

**准备**，对于备份还原来说，需要先 `prepare` 备份文件，使数据达到 `a single instant in time`。（还原可以在任何服务器上执行，前提是安装 XtraBackup）。
```shell
xtrabackup --prepare --target-dir=/data/bkps
```

**恢复**，将准备好的数据还原到 mysql 服务器。可以新安装一个 MySQL，然后将数据目录清空，再将准备好的恢复文件复制到数据目录即可。

- 恢复之后用户账号和权限体系将与备份时刻完全一致，即新安装的 MySQL 服务器的用户和权限体系将移除（因为数据目录清空了）。
- 恢复之后需要注意 Linux 系统的文件权限，修改为 mysql 用户。
- 恢复不会还原 MySQL 配置文件，如 /etc/my.cnf，需要确保新安装的 MySQL 服务器配置与备份时的配置兼容。
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
**XtraBackup 备份还原数据中文乱码问题。**
需要配置所有数据库字符集相关的变量值一致。

