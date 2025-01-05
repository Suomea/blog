---
comments: true
tags:
  - MySQL
---
## 系统变量
官方文档：https://dev.mysql.com/doc/refman/8.4/en/server-system-variables.html

很多系统变量可以通过启动选项在 MySQL 启动时设置，可以在命令行设置启动选项或者在配置文件中设置启动选项，命令行的优先级大于配置文件的优先级。

通过命令行设置启动选项：
```
mysqld --max-connections=2000
```

 通过配置文件设置启动选项：
```
[server]
max-connections=2000
```

需要注意的是启动选项 max-connections 对应的系统变量为 max_connections。

系统变量有两种作用范围（scope），全局变量 GLOBAL 和会话变量 SESSION。

参考：https://dev.mysql.com/doc/refman/8.4/en/server-system-variable-reference.html 可以查阅系统变量的作用范围。

大部分系统变量只有 GLOBAL 作用范围，一部分变量只有 SESSION 作用范围，一部分变量的作用范围为 GLOBAL，SESSION。

服务器在启动时，会将每个全局变量初始化为其默认值，可以通过命令行或者配置文件指定启动选项修改这些默认值（只能设置 GLOBAL 作用范围的）。

服务器还为每一个连接的客户端维护了一组会话变量，客户端的会话变量在连接时使用相应全局变量的当前值进行初始化。

比如 max_connections 系统变量就是 GLOBAL 作用范围的，全局限定了同时连接到系统的客户端数量。

比如 autocommit 系统变量的作用范围就是 GLOBAL，SESSION。数据库启动时会维护一个全局变量 autocommit，之后每一个客户端连接的时候数据库会为每个客户端维护一个会话变量 autocommit，会话变量 autocommit 的默认值为客户端连接时全局变量 autocommit 的值。客户端可以修改自己的会话变量而不会影响全局变量以及其它客户端的会话变量。

运行期间通过 SQL 设置系统变量的值，如果不指定 GLOBAL 的话则默认为 SESSION 作用范围。
```
set [GLOBAL|SESSION] var_name = value;
```

以 autocommit 为例，客户端连接成功之后系统维护了当前客户端的 SESSION autocommit 和 GLOBAL autocommit 两个变量。假设此时修改 GLOBAL autocommit 并不会对当前客户端的 SESSION autocommit 变量造成影响，只会对后续建立连接的客户端的 SESSION autocommit 产生影响。

有些系统变量设置后立即生效，有些则需要在启动选项中配置重启数据库生效。

查看系统变量语法，如果不指定 GLOBAL 的话则默认为 SESSION 作用范围。
```sql
show [GLOBAL|SESSION] variables [like pattern];
```
需要注意的是：  

- 如果使用 GLOBAL 修饰符，则显示全局变量的值。如果系统变量没有 GLOBAL 作用范围，则不显示。

- 如果使用 SESSION 修饰符，则展示会话变量的值。如果系统变量没有 SESSION 作用范围，则显示 GLOBAL 作用范围的值。
## 状态变量
官方文档：https://dev.mysql.com/doc/refman/8.4/en/server-status-variable-reference.html

状态变量是用来显示数据库的运行状态的，它们的值只能由数据库自己设置，不能手动设置。状态变量也有作用范围（scope）GLOBAL 和 SESSION。

查看状态变量的语法，如果不指定 GLOBAL 的话则默认为 SESSION 作用范围。
```
show [GLOBAL|SESSION] status [like pattern];
```
需要注意的是：

- 如果使用 GLOBAL 修饰符，则显示全局变量的值。如果状态变量没有 GLOBAL 作用范围，则不显示。

- 如果使用 SESSION 修饰符，则展示会话变量的值。如果状态变量没有 SESSION 作用范围，则显示 GLOBAL 作用范围的值。