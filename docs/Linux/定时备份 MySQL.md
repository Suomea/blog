备份采用 crontab 定时执行脚本的方式进行。

## 备份

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

mysqldump --defaults-file=$cre_file $database_name | gzip > $backup_dir/$database_name-$dd.gz

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
