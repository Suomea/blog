---
comments: true
tags:
  - MySQL
---
## 使用 Docker 安装
```shell
安装命令：
docker run --name mysql -v /my/own/datadir:/var/lib/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=pwd -d mysql:tag
```

## APT 安装
下载安装软件包：
```shell
wget https://dev.mysql.com/get/mysql-apt-config_0.8.26-1_all.deb 
dpkg -i mysql-apt-config_0.8.26-1_all.deb 
apt update
```

安装 MySQL 服务器、客户端以及数据库通用的文件包：
```shell
apt install mysql-server
```

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

## 二进制包安装
### 环境准备
安装必要的库：
```shell
apt install libaio1 libnuma1 libncurses6
```

创建用户和组：
```shell
groupadd mysql 
useradd -r -g mysql -s /bin/false mysql

-r：创建系统用户  
-g：指定用户的初始组  
-s：/bin/false 禁止用户登录任何服务，相对 /sbin/nologin 更为严格。  
```

下载和解压缩安装包：
```shell
wget https://downloads.mysql.com/archives/get/p/23/file/mysql-8.0.33-linux-glibc2.28-x86_64.tar.gz 
tar -zxvf mysql-8.0.33-linux-glibc2.28-x86_64.tar.gz -C /usr/local/
```

配置环境变量：
```shell
export PATH=$PATH:/usr/local/mysql/bin
```
### 初始化目录
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

初始化数据目录 `Initializing the Data Directory`，使用二进制安装包的安装方式必须手动初始化数据目录，包括初始化系统相关的表，使用 ATP 安装方式则会自动进行。执行初始化数据目录命令，初始化完成后 root 用户的密码会输出的控制台：
```shell
mysqld --initialize --user=mysql
```

### 启动和关闭
可以直接使用 `mysqld` 命令启动 MySQL 服务器。`mysqld_safe` 是一个启动脚本会间接调用 `mysqld`，并且启动一个监控进程，监控 `mysqld` 挂了之后进行重启。启动服务器：
```shell
mysqld_safe --user=mysql &
```

更新密码：
```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root-password'; 
flush privileges;
```

关闭 MySQL 服务器：
```shell
./bin/mysqladmin -u root -p shutdown
```

### 导入测试数据
测试数据仓库地址：[https://github.com/datacharmer/test_db](https://github.com/datacharmer/test_db)

mysql 执行 SQL 导入数据的语法：
```sql
mysql -h localhost -P 3306 -u root -p dbname < file.sql
```


## 密码复杂度和过期策略
### 复杂度设置
密码复杂度插件位于安装目录下的 plugins 目录，文件名：`validate_password.so`。

启用插件不会影响先前的用户，但是如果先前的用户更新密码则会收到影响。

安装：
```
-- 查看安装的插件
show PLUGINS;

-- 启用插件，重启不会失效
INSTALL PLUGIN validate_password SONAME 'validate_password.so';
```

查看复杂度配置：
```
-- 查看密码复杂度配置
SHOW VARIABLES LIKE 'validate_password%';

-- 密码不能与用户名相同
validate_password_check_user_name	ON
-- 指定字典文件，如果密码与字典中的单词匹配，密码将被拒绝
validate_password_dictionary_file	
-- 密码最小长度
validate_password_length	8
-- 密码至少包含一个大写和一个小写字母
validate_password_mixed_case_count	1
-- 密码至少包含一个数字
validate_password_number_count	1
-- MEDIUM：检查长度，大小写，数字，特殊字符；LOW：检查长度；STRONG：在 MEDIUM 基础上，增加字典文件检查
validate_password_policy	MEDIUM
-- 至少包含一个特殊字符
validate_password_special_char_count	1
```

调整密码复杂度配置：
```
-- 根据需要调整策略（可选），重启不会失效
SET PERSIST validate_password.policy = 'STRONG';
SET PERSIST validate_password.length = 12;
```
### 过期时间设置
通过系统变量 `default_password_lifetime` 来控制，通过系统用户表的 `password_last_changed` 和当前时间来判断密码是否过期。

查看值：
```sql
show variables like 'default_password_lifetime';
```

设置密码过期时间：
```sql
-- 禁用密码过期机制，重启不会失效
SET PERSIST default_password_lifetime = 0;

-- 设置密码 90 天过期，重启不会失效
SET PERSIST default_password_lifetime = 90;
```

系统变量 `default_password_lifetime` 是全局的密码过期策略，还可以针对单个用户进行设置。针对单个用户的设置优先级高于系统变量。
```sql
-- 设置 test 用户密码的过期策略为 60 天
ALTER USER 'test'@'localhost' PASSWORD EXPIRE INTERVAL 60 DAY;

-- 设置 root 用户的密码过期策略为永不过期
ALTER USER 'root'@'%' PASSWORD EXPIRE NEVER;
```

操作手册：
```sql
-- 查看用户密码过期相关的字段
SELECT user, host, password_last_changed, password_lifetime, password_expired FROM mysql.user;

-- 更新所有用户的最近密码修改时间
UPDATE mysql.user SET password_last_changed = NOW();

-- 设置  root 用户密码永不过期
ALTER USER 'root'@'%' PASSWORD EXPIRE NEVER;

-- 全局设置用户密码 90 天过期
SET PERSIST default_password_lifetime = 90;

-- 查看全局过期时间
show variables like 'default_password_lifetime';
```

总结：

1. 系统变量 `default_password_lifetime` 全局设置用户的密码过期策略，通过 `password_last_changed` 和当前时间来判断用户密码是否过期。
2. 语句 `ALTER USER PASSWORD EXPIRE` 针对单个用户设置密码过期策略，通过 `password_lifetime` 和当前时间来判断用户密码是否过期。优先级高于系统变量。

