## fail2ban
ssh 登录失败锁定
https://zhuanlan.zhihu.com/p/597751230

nginx 401|403|404 锁定
https://www.8kiz.cn/archives/24353.html

安装：
```
apt install fail2ban
```

配置文件目录：

| **文件/目录​**​           | ​**​作用​**​                                            | ​**​关键内容/示例​**​                                                 |
| --------------------- | ----------------------------------------------------- | --------------------------------------------------------------- |
| `action.d/`           | 存放封禁动作（Actions）的配置文件，定义触发规则后的操作（如调用防火墙、发邮件）。          | `iptables.conf`（封禁 IP）、`sendmail.conf`（邮件通知）。                   |
| `fail2ban.conf`       | fail2ban 主配置文件，控制服务全局行为（如日志级别、套接字路径）。                 | `loglevel = INFO`、`logtarget = /var/log/fail2ban.log`。          |
| `fail2ban.d/`         | 存放覆盖 `fail2ban.conf`的模块化配置片段（类似 `conf.d`设计）。          | 可添加 `custom.conf`覆盖主配置参数（优先级高于 `fail2ban.conf`）。                |
| `filter.d/`           | 存放日志过滤器规则，用于解析服务日志并匹配攻击行为（如 SSH 暴力破解）。                | `sshd.conf`（匹配 SSH 失败日志）、`nginx-http-auth.conf`（匹配 Nginx 认证失败）。 |
| `jail.conf`           | 默认监禁规则（Jails）配置文件，定义各服务的保护规则（如 `maxretry`、`bantime`）。 | `[sshd]`、`enabled = true`、`maxretry = 5`。                       |
| `jail.d/`             | 存放独立的 Jail 配置片段，覆盖或扩展 `jail.conf`（优先级最高）。             | `sshd.conf`（自定义 SSH 保护规则）、`nginx.conf`（Nginx 专用配置）。             |
| `paths-common.conf`   | 定义通用日志文件路径（跨系统兼容）。                                    | `auth.log = /var/log/auth.log`。                                 |
| `paths-debian.conf`   | Debian 系统的特定日志路径（覆盖 `paths-common.conf`）。             | `auth.log = /var/log/auth.log`（Debian 默认路径）。                    |
| `paths-arch.conf`     | Arch Linux 的日志路径配置。                                   | `auth.log = /var/log/auth.log`（Arch 路径）。                        |
| `paths-opensuse.conf` | openSUSE 的日志路径配置。                                     | `auth.log = /var/log/messages`（openSUSE 路径）。                    |

按照下面的顺序加载配置，后者覆盖前者：
1. `jail.conf` 基础默认配置。
2. `jail.loca`l 用户自定义配置。
3. `jail.d/*.conf` 更细粒度的配置。

**日志监控** fail2ban 根据 `paths-*.conf`确定日志路径（如 `/var/log/auth.log`）。通过 `inotify`或 `polling`实时监听日志变化，会记录文件的 inode 和偏移量。也可以通过 logpath 指定日志路径。

如果配置了日志轮转，那么需要在轮转里面配置 fail2ban 重新加载日志。

如果服务日志通过 journald 管理，fail2ban 也可以配置监控：
```
[sshd]
enabled   = true
backend   = systemd                  # 使用 systemd 后端
journalmatch = _SYSTEMD_UNIT=sshd.service  # 只监控 sshd 服务的日志
```

**规则匹配** 读取新增日志行后，调用 `filter.d/`中的规则（如 `sshd.conf`）进行正则匹配。若匹配成功，触发对应 Jail 的计数器（记录在 `/var/lib/fail2ban/fail2ban.sqlite3`）。

**监禁判定** 根据 `jail.conf`、`jail.local`或 `jail.d/*.conf`中的参数（如 `maxretry=3`）判断是否封禁 IP。若达到阈值，执行 `action.d/`中定义的动作。

**执行封禁** 动作脚本（如 `iptables.conf`）添加防火墙规则，禁止 IP 访问指定端口。可选通知操作（如 `sendmail.conf`发送告警邮件）。

## 配置解禁

白名单配置，将可信 IP 加入 `ignoreip`，避免被封禁。
```
# 更新配置
[DEFAULT]
ignoreip = 127.0.0.1/8 192.168.1.0/24

# 需要重启服务
systemctl restart fail2ban
```

解封操作：
```
# 清空并重置 jail
fail2ban-client restart <JAIL名称> 

# 解封某个 ip
fail2ban-client set <JAIL名称> unbanip 192.168.1.100
```

Tips：如果多个 jail 都封禁了一个 IP，那么解禁只会影响当前 jail。如果多个 jail 都是 port=all，那么也需要都解禁才能访问进来。
## SSH 配置
新增配置文件 /etc/fail2ban/jail.d/sshd.local：
```
[sshd]
# 使用 journal 日志
backend=systemd
enabled=true
# 10m 内，5 次，封禁 12h
maxretry=5
findtime=10m
bantime=12h
```

查看属性信息：
```
fail2ban-client get sshd bantime
fail2ban-client get sshd findtime
fail2ban-client get sshd maxretry
fail2ban-client get sshd failregex
```

查看监禁状态：
```
fail2ban-client status sshd
```

## Nginx 配置
假设 Nginx 日志配置：
```
log_format j_format escape=json  
  '{"time":"$time_iso8601",'  
   '"remote_addr":"$remote_addr",'  
   '"scheme":"$scheme",'  
   '"status":$status,'  
   '"request_method":"$request_method",'  
   '"uri":"$uri",'  
   '"args":"$args",'  
   '"server_protocol":"$server_protocol",'  
   '"body_bytes_sent":$body_bytes_sent,'  
   '"request_time":$request_time,'  
   '"http_referer":"$http_referer",'  
   '"http_user_agent":"$http_user_agent",'  
   '"http_token":"$http_token",'  
   '"upstream_response_time":"$upstream_response_time",'  
   '"service_name":"$service_name",'  
   '"upstream_addr":"$upstream_addr"}';

access_log  logs/access.json j_format;
```

实现下面两个需求：
1. 配置如果同一个 IP 10 分钟内有 20 个 404 请求，就封禁 12 小时。
2. 配置如果同一个 IP 10 分钟内有 20 个 /login 接口调用，就封禁 12 个小时。

规则匹配配置，需要注意时间时间和文件修改时间，如果没有指定时间的话，默认使用文件修改时间来做处理：
```
# /etc/fail2ban/filter.d/nginx-404.conf
[Definition]
failregex = ^{"time":".*?","remote_addr":"<HOST>",.*?"status":404,.*?}$

# 时间戳配置
datestart = ^{"time":"
datepattern = ^"%%Y-%%m-%%dT%%H:%%M:%%S%%z"

# /etc/fail2ban/filter.d/nginx-login.conf
[Definition]
failregex = ^{"time":".*?","remote_addr":"<HOST>",.*?"uri":"/login",.*?}$

# 时间戳配置
datestart = ^{"time":"
datepattern = ^"%%Y-%%m-%%dT%%H:%%M:%%S%%z"
```

监禁配置：
```
# /etc/fail2ban/jail.d/nginx.conf
[nginx-404]
enabled = true
# 这里端口号默认 80,443 如果想要全部封禁直接 port=all
port = http,https
filter = nginx-404
logpath = /usr/local/nginx/logs/access.json
maxretry = 20
findtime = 600
bantime = 3600

[nginx-login]
enabled = true
port = http,https
filter = nginx-login
logpath = /usr/local/nginx/logs/access.json
maxretry = 20
findtime = 600
bantime = 3600
```

测试配置：
```
# 配置文件是否正确
fail2ban-client --test

# 配置是否能正确匹配
fail2ban-regex /usr/local/nginx/logs/access.json /etc/fail2ban/filter.d/nginx-404.conf 
fail2ban-regex /usr/local/nginx/logs/access.json /etc/fail2ban/filter.d/nginx-login.conf 

# fail2ban 测试输出匹配的行
tail -n 1000 /usr/local/nginx/logs/access.json > /tmp/test.log
fail2ban-regex /tmp/test.log /etc/fail2ban/filter.d/nginx-404.conf --print-all-matched

# grep 测试输出匹配的行
tail -n 2000 /usr/local/nginx/logs/access.json | grep --color=always -E '^{"time":".*?","remote_addr":".*",.*?"status":404,.*?}$'
```


加载配置：
```
fail2ban-client reload
fail2ban-client status
```
