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

**日志监控** fail2ban 根据 `paths-*.conf`确定日志路径（如 `/var/log/auth.log`）。通过 `inotify`或 `polling`实时监听日志变化，会记录文件的 inode 和偏移量。

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

## SSH 配置
查看监禁的所有配置：
```
fail2ban-client get sshd all
```

查看监禁状态：
```
fail2ban-client status sshd
```

## Nginx 配置
