## Debian
安装 Redis。
```
apt update
apt install redis-server
```


查看状态。
```
systemctl enable redis-server --now
systemctl status redis-server
```


编辑配置文件 `/etc/redis/reids.conf`，设置密码。然后执行 `systemctl restart redis-server` 命令重启 `redis-server` 使修改生效。
```
requirepass <password>
```


连接测试。
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

编辑配置文件 `/etc/redis/reids.conf`，配置允许远程连接，关闭保护模式（为了方便测试，不设置密码）。
```
bind 0.0.0.0 -::1
protected-mode no
```

## Windows
下载安装文件 Redis-7.4.2-Windows-x64-msys2.zip

解压缩，启动：
```
redis-server.exe redis.conf
```