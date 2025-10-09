## 安装
最小化安装，仅需要 smb 文件共享功能。
```shell
apt install --no-install-recommends samba samba-common-bin
```

安装的依赖
- samba 核心的守护进程，提供 smbd 文件共享服务和 nmbd NetBIOS。
- samba-common 公共配置文件和脚本。
- samba-common-bin 管理命令。
- samba-libs 核心库。

相关的服务
- smbd 文件共享核心服务。
- nmbd 用于 NetBIOS 网络邻居广播，可以停用。
- winbind 域控/LDAP 环境使用。
- samba-ad-dc AD 域控制器功能。
- samba 包装服务，会同时启动其它服务。

停止不必要的服务
```
systemctl stop nmbd winbind samba-ad-dc samba
systemctl disable nmbd winbind samba-ad-dc samba
```

smbd 默认会监听两个端口：
- 445/tcp 现代 SMB2/SMB3 直接使用。
- 139/tcp 旧 SMB1/NetBIOS 模式。

## 配置连接
配置文件 /etc/samba/smb.conf
```
[disk1]
path = /data/disk1
browseable = yes
writable = yes
guest ok = no

[disk2]
path = /data/disk2
browseable = yes
writable = yes
guest ok = no
```

## 状态监控
smbstatus 命令查看连接状态。

## 用户管理
添加一个 smb 用户
1. 首先系统中要存在这个 Linux 用户，没有的话可以新增。
```
useradd -M -s /sbin/nologin jacky

-M 不创建 home 目录。
-s /sbin/nologin 禁止直接登录系统。
```

2. 添加 smb 用户。
```
smbpasswd -a jacky
```

3. 验证是否添加成功。
```
pdbedit -L
```
