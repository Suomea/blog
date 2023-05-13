## How to install MySQL on Debian 11

### 添加 MySQL 软件仓库

Debian 默认的仓库并不包含 MySQL，所以要先为 APT 添加 MySQL 的软件仓库。

```shell
$ wget https://dev.mysql.com/get/mysql-apt-config_0.8.23-1_all.deb
$ dpkg -i mysql-apt-config_0.8.23-1_all.deb
```

添加完成之后更新软件包列表。

```shell
$ apt update
```

### 安装 mysql-server

执行命令安装 `mysql-server`，安装过程中需要输入和确认密码，其它选项默认。

```
$ apt install mysql-server
```

### 设置 root 用户允许远程连接

安装完成之后，默认只允许使用 `root` 账户在本地进行连接。在安装 MySQL 的机器上连接命令如下，之后输入密码即可。

```shell
$ mysql -u root -p
```

在登录进入 MySQL 服务之后，设置 `root` 账户只允许处于 `192.168.31.0/24` 网络的客户端进行连接服务，执行如下 SQL。如果将 `host` 的值设置为 `%` 则不限
制使用 `root` 账号连接服务的 客户端所处的网络。

```sql
-- 更新 root 账号的连接限制
update mysql.user
set host = '192.168.31.%'
where user = 'root';
-- 刷新配置，使之生效
flush privileges;
```

## MySQL count() 函数的注意事项

简化过后的场景是一条业务数据上来之后，机器会自动对数据进行分级，存储在 `auto_level` 字段，之后需要人工进行复检执行手动分级操作，最终的分级以手动分级为准。
现在的业务需求是求机器自动分级的准确率。  
表结构定义如下：

```sql
create table test_tb
(
    id           int primary key auto_increment,
    auto_level   int null comment '自动分级',
    manual_level int null comment '手动分级'
);
```

只要查询出一共有多少条数据，再查询出手动分级和自动分级不一致的数据即可。  
错误的 SQL：

```sql
select count(*)                          as totalCount,
       count(manual_level != auto_level) as diffCount
from test_tb;
```

错误的原因在于 `manual_level != auto_level` 逻辑表达式，返回的结果是 `True` 或者 `False`（如果有一列为 `NULL` 的话返回 `NULL`，这里假设两列都为不 `NULL`），
那么无论表达式的返回结果是 `True` 还是 `False`，都会被 `count()` 函数统计进去。

正确的 SQL：

```sql
select count(*)                                                      as totalCount,
       sum(case when (manual_level != auto_level) then 1 else 0 end) as diffCount
from test_db;
```

## 在 MySQL 中创建用户并赋予权限