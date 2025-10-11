---
comments: true
tags:
  - MySQL
---
## Docker 安装
```shell
安装命令：
docker run --name mysql -v /my/own/datadir:/var/lib/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=pwd -d mysql:tag
```

## ATP 安装

### 添加 MySQL ATP 仓库

下载安装软件包

```shell
wget https://dev.mysql.com/get/mysql-apt-config_0.8.26-1_all.deb 
dpkg -i mysql-apt-config_0.8.26-1_all.deb 
apt update
```

### 安装 MySQL

安装 MySQL，下面命令将会安装 MySQL 服务器，客户端以及数据库通用的文件包。

```shell
apt install mysql-server
```

设置 root 用户允许所有主机远程连接（仅用于测试，生产环境不允许 root 用户的远程连接）。

```sql
update mysql.user set host = '%' where user = 'root'; 
flush privileges;
```

`Systemd` 服务配置，文件地址：`/lib/systemd/system/mysql.service`。
```
[Unit]
Description=MySQL Community Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network-online.target
Wants=network-online.target

[Install]
WantedBy=multi-user.target

[Service]
User=mysql
Group=mysql
Type=notify
ExecStartPre=+/usr/share/mysql-8.0/mysql-systemd-start pre
ExecStart=/usr/sbin/mysqld
TimeoutSec=0
LimitNOFILE = 10000
Restart=on-failure
RestartPreventExitStatus=1

# Always restart when mysqld exits with exit code of 16. This special exit code
# is used by mysqld for RESTART SQL.
RestartForceExitStatus=16

# Set enviroment variable MYSQLD_PARENT_PID. This is required for restart.
Environment=MYSQLD_PARENT_PID=1
```

### 卸载 MySQL

停止 MySQL 服务：

```shell
systemctl stop mysql
```

查看安装了哪些 MySQL 包，然后使用 apt remove 命令逐个卸载：

```shell
apt list --installed
```

删除数据目录：

```shell
rm -rf /var/lib/mysql
```

删除 Systemd 服务配置文件：

```shell
rm -rf /lib/systemd/system/mysql.service 
rm -rf /etc/systemd/system/mysql.service 
systemctl daemon-reload
```

## 通用二进制安装包安装

### 安装必要的库
```shell
apt install libaio1
apt install libnuma1
apt install libncurses6
```

### 下载安装包

下载和解压缩：
```shell
wget https://downloads.mysql.com/archives/get/p/23/file/mysql-8.0.33-linux-glibc2.28-x86_64.tar.gz 
tar -zxvf mysql-8.0.33-linux-glibc2.28-x86_64.tar.gz -C /usr/local/
```

安装包的文件夹：

| 目录            | 目录内容                          |
| ------------- | ----------------------------- |
| bin           | mysqld server，client 和 一些工具程序 |
| docs          | 手册                            |
| man           | Unix manual pages             |
| include       | 头文件                           |
| lib           | 库文件                           |
| share         | 错误信息、字典和数据库安装 SQL             |
| support-files | 支持文件                          |

### 创建用户和组
```shell
groupadd mysql 
useradd -r -g mysql -s /bin/false mysql
```
-r：创建系统用户  
-g：指定用户的初始组  
-s：/bin/false 禁止用户登录任何服务，相对 /sbin/nologin 更为严格。  


### 初始化
初始化数据目录 `Initializing the Data Directory`，使用二进制安装包的安装方式必须手动初始化数据目录，包括初始化系统相关的表，使用 ATP 安装方式则会自动进行。

配置环境变量：
```shell
export PATH=$PATH:/usr/local/mysql/bin
```

新建目录：
```shell
cd /usr/local/mysql 
mkdir mysql-files 
chown mysql:mysql mysql-files 
chmod 750 mysql-files
```

创建配置文件：`/etc/mysql/my.cnf`
```
[mysqld] 
basedir=/usr/local/mysql/ 
datadir=/usr/local/mysql/mysql-files
```

初始化数据目录，其中 root 用户的密码会输出的控制台：
```shell
mysqld --initialize --user=mysql
```

### 启动 MySQL 服务器

可以直接使用 `mysqld` 命令启动 MySQL 服务器。`mysqld_safe` 是一个启动脚本会间接调用 `mysqld`，并且启动一个监控进程，监控 `mysqld` 挂了之后进行重启。启动服务器：
```shell
mysqld_safe --user=mysql &
```

更新密码：
```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root-password'; 
flush privileges;
```

### 关闭 MySQL 服务器：

```shell
./bin/mysqladmin -u root -p shutdown
```

### 导入测试数据

测试数据仓库地址：[https://github.com/datacharmer/test_db](https://github.com/datacharmer/test_db)

mysql 执行 SQL 导入数据的语法：
```sql
mysql -h localhost -P 3306 -u root -p dbname < file.sql
```

手动添加测试数据
```
create database if not exists test;
use test;

create table if not exists user(
	username varchar(100) not null comment '姓名',
	age int not null comment '年龄',
	create_at datetime not null default current_timestamp comment '数据创建时间'
) comment '用户信息表';

insert into user(username, age)
values ('jacky', 22), ('otto', 18), ('zhangsan', 25);
```
