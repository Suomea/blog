---
comments: true
tags:
  - MySQL
---
## 主从介绍
主从能够将一个 MySQL 的数据复制到一个或者多个 MySQL，默认主从是异步的。能够配置同步所有的数据库、选择的数据库甚至选择的表。

### 主从的优点
**横向扩展解决方案**，将负载分散到多个副本以提高性能。在这种情况下所有的写入和更新都必须在主库服务器上进行，但是读取可以在一个或者多个副本上进行。
读写分离的模型可以提高性能，主库专注于写入和更新，可以扩展更多的副本提高读取速度。

**数据安全**，由于副本可以暂停复制过程，可以在副本上运行备份服务而不会对主库数据造成影响。

**分析**，在副本上分析数据不会影响主库的性能。

**远距离数据分发**，可以使用本地的副本供远程站点使用，而不会影响主库的性能。

### 主从的方式
MySQL 8.1 支持不同的主从方法：

* 传统方式基于从主库的二进制日志复制事件，并且要求日志文件和其中的位置在主库和副本之间同步。
* 新的方式是基于全局事务标识符（Global Transaction Identifiers，GTIDs），这种方式是事务性的，因此不需要处理日志文件或者文件内的位置，简化了许多常见的复制
任务。使用 GTIDs 的主从可以保证主库和副本之间的一致性，只要在主库上提交过的事务也已应用于副本。

### 主从格式类型
主从格式有两种核心的类型：

* 基于语句的复制（Statement Base Replication，SBR），复制整个 SQL 语句执行。
* 基于行的复制（Rwo Base Replication，RBR），仅复制改变的行。

## 基于二进制日志的主从
主库将更新作为“事件”写入二进制日志，从库从源读取日志并在本地数据库上执行二进制日志中的事件。每个从库都会接收二进制文件中的全部内容，但是
从库根据配置可以决定执行二进制日志中的哪些语句，默认情况下执行全部的语句。
!!! note "主库的二进制日志不能配置仅记录某些事件。"

主库和从库都必须配置唯一的 ID（使用 server_id 系统变量）。每个从库都必须配置主库的地址、日志文件名和该文件中的位置信息，可以使用
`CHANGE REPLICATION SOURCE TO`在从库上配置。

这里简要描述一下主从的步骤：

1. 在主库上确保开启了二进制日志，并且配置的唯一的服务器 ID。重新配置需要重启数据库。
2. 在每个从库上配置唯一的服务器 ID。重新配置需要重启数据库。
3. 在主库上创建一个单独的用户，在从库读取主库的二进制日志是进行身份验证。
4. 在进行复制之前，应记录主库二进制日志的当前位置，配置从库时需要这个信息。如果主库已经有数据，则需要创建数据的快照，将数据复制到从库。
5. 配置从库主库的信息（如：地址、登录凭证、二进制日志文件名和位置）。

### 配置主库
创建用于同步的用户
```sql
create user 'repl'@'%' identified by 'password';
grant replication slave on *.* to 'repl'@'%';
flush privileges;
```

二进制日志默认是开启的，服务器 ID 默认是 1，修改配置文件需要重启数据库。配置文件 **/etc/mysql/mysql.conf.d/mysqld.cnf**新增配置：
```cnf
[mysqld]
server_id               = 2
expire_logs_days        = 15

character-set-server    = utf8mb4
max_connections         = 2000
```

查看二进制日志文件和位置，从库配置需要使用 `File`和`Position`。
```text
mysql> show master status \G
*************************** 1. row ***************************
             File: binlog.000003
         Position: 157
     Binlog_Do_DB: 
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 
1 row in set (0.00 sec)
```

### 配置从库
修改配置文件需要重启数据库。编辑配置文件 **/etc/mysql/mysql.conf.d/mysqld.cnf**：
```cnf
[mysqld]
server_id               = 21
expire_logs_days        = 15

read-only               = 1
character-set-server    = utf8mb4
max_connections         = 2000
```

配置主库信息：
```sql
CHANGE REPLICATION SOURCE TO
    SOURCE_HOST='192.168.1.13',
    SOURCE_USER='repl',
    SOURCE_PASSWORD='password',
    SOURCE_LOG_FILE='binlog.000003',
    SOURCE_LOG_POS=157;
```

开启主从同步：
```sql
start replica;
```

查看主从状态：
```sql
mysql> show replica status \G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: 192.168.1.13
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: binlog.000003
          Read_Master_Log_Pos: 157
               Relay_Log_File: suomea04-relay-bin.000002
                Relay_Log_Pos: 323
        Relay_Master_Log_File: binlog.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
···
```

### 主库已经有数据的情况
如果主库已经有数据,需要先使用 mysqldump 导出主库数据，把数据导入从库之后再配置从库进行同步。
```sql
mysqldump -u root -p --source-data=2 --databases test_db > /root/test.sql
```

需要注意的是 `--source-data` 选项会导出主库的二进制文件名和位置信息到 `test.sql` 文件中，默认值是 `1`，如果设置值为 `2` 的话会注释掉该 SQL 语句。
```sql
···
-- CHANGE MASTER TO MASTER_LOG_FILE='binlog.000003', MASTER_LOG_POS=363;
···
```

如果 `--source-data` 设置为 `1`，从库导入数据的时候就会设置主库的二进制文件和位置信息，执行 `CHANGE REPLICATION SOURCE TO` 语句时就不需要再次设置了；
如果设置为 `2`，从库执行 `CHANGE REPLICATION SOURCE TO` 语句需要设置二进制文件和位置信息，值需要去主库导出的 SQL 文件中查找。

## 主从切换
假设主库数据库服务器炸了，那么从库进行操作：
```
-- 首先停止主从同步
stop slave;
reset slave all;

-- 取消数据库的只读限制
SET GLOBAL read_only=0;

-- 服务配置更新数据库地址，重启服务
```