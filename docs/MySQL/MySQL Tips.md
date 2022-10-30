

记录 MySQL 的安装配置和查询相关的技巧，相关的概念不会深入进行解释。具体的详细介绍可以单独写一篇文章。

## How to install MySQL on Debian 11
### 添加 MySQL 软件仓库
Debian 默认的仓库并不包含 MySQL，所以要先为 APT 添加 MySQL 的软件仓库。
```shell
wget https://dev.mysql.com/get/mysql-apt-config_0.8.23-1_all.deb
dpkg -i mysql-apt-config_0.8.23-1_all.deb
```

添加完成之后更新软件包列表。
```shell
apt update
```

### 安装 mysql-server
执行命令安装 `mysql-server`，安装过程中需要输入和确认密码，其它选项默认。
```
apt install mysql-server
```

### 设置 root 用户允许远程连接
安装完成之后，默认只允许使用 `root` 账户在本地进行连接。在安装 MySQL 的机器上连接命令如下，之后输入密码即可。
```shell
mysql -u root -p
```

在登录进入 MySQL 服务之后，设置 `root` 账户只允许处于 `192.168.31.0/24` 网络的客户端进行连接服务，执行如下 SQL。如果将 `host` 的值设置为 `%` 则不限
制使用 `root` 账号连接服务的  客户端所处的网络。
```sql
-- 更新 root 账号的连接限制
update mysql.user set host = '192.168.31.%' where user = 'root';
-- 刷新配置，使之生效
flush privileges;
```

## 