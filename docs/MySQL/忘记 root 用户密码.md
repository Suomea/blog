#MySQL

参考：https://stackoverflow.com/questions/50691977/how-to-reset-the-root-password-in-mysql-8-0-11

停止 MySQL 服务 `systemctl stop mysql`

编辑配置文件，增加选项 `skip-grant-tables`

启动 MySQL 服务 `systemctl start mysql`

进入 MySQL `mysql`
	`update mysql.user set authentication_string = null where user = 'root';`
	`flush privileges`
	`exit`

再次进入 MySQL `mysql -u root `
	`alter user root@localhost identified  by 'new_password';`
	`flush privileges;`
	`exit`

重启 MySQL 服务 `systemctl restart mysql`

使用密码登录 MySQL `mysql -u root -p`