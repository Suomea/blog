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
