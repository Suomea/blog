## Debian
查看要安装的版本：
```
apt show redis-server
```

安装 Redis：
```
apt update
apt install redis-server
```

查看状态：
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

编辑配置文件 `/etc/redis/reids.conf`，配置允许远程连接。
```
bind 0.0.0.0 -::1
# 表示监听所有的 IPv4 地址，不监听 IPv6 的本地回环地址
```

Redis 相关的文件路径：

| 类型         | 路径                                                    | 说明                      |
| ---------- | ----------------------------------------------------- | ----------------------- |
| 主程序        | `/usr/bin/redis-server`                               | Redis 服务端               |
| 客户端        | `/usr/bin/redis-cli`                                  | Redis 命令行客户端            |
| 工具         | `/usr/bin/redis-benchmark`、`/usr/bin/redis-check-rdb` | 性能测试与检查工具               |
| 配置文件       | `/etc/redis/redis.conf`                               | 主配置文件                   |
| 数据目录       | `/var/lib/redis`                                      | 存储 `.rdb` 数据快照文件        |
| 日志目录       | `/var/log/redis`                                      | Redis 运行日志              |
| systemd 服务 | `/lib/systemd/system/redis-server.service`            | 用于 `systemctl` 控制 Redis |
| 文档         | `/usr/share/doc/redis-server/`                        | README、版权等              |

## Windows
下载安装文件 Redis-7.4.2-Windows-x64-msys2.zip

解压缩，启动：
```
redis-server.exe redis.conf
```