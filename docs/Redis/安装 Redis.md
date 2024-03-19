更新包信息
```
apt update
```

安装 Redis
```
apt install redis-server
```

查看状态
```
systemctl status redis-server
```

编辑配置文件 /etc/redis/reids.conf，设置密码
```
requirepass <password>
```

重启
```
systemctl restart redis-server
```

连接测试
```
# redis-cli
127.0.0.1:6379> set name jacky
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth <password>
OK
127.0.0.1:6379> set name jacky
OK
127.0.0.1:6379> get name
"jacky"
```