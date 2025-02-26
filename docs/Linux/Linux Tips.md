记录 Linux 安装和配置的相关技巧，主要基于 Debian 进行操作。

## 配置静态 IP
### 备份网卡配置文件
```text
cp /etc/network/interfaces /etc/network/interfaces.bak
```

### 编辑配置文件
```text
# The primary network interface
auto eth0
allow-hotplug eth0
# 1. 修改为 static
iface eth0 inet static
# 2. 配置静态 IP
address 192.168.31.12
# 3. 配置子网掩码
netmask 255.255.255.0
# 4. 配置网关地址
gateway 192.168.31.1
```

### 重启网络
```text
systemctl restart networking
```

## Linux 文件权限
```text
drwx------  8 root root      4096 Nov  9 22:18 .
drwxr-xr-x 19 root root      4096 Nov  9 22:35 ..
-rw-r--r--  1 root root  87473664 Nov  1 21:30 a.tar.gz
```

r: 代表可读  4  
w: 代表可写  2  
x: 代表可执行 1

chgrp: 改变文件所属的群组  
chown: 改变文件拥有者  
chmod: 改变文件的权限

## 配置 SSH 免密登录
假设从 Win 免密登录到 Linux。
### Win 生成密钥要对
```shell
ssh-keygen -t rsa
```

### 配置
1. 复制 Win 电脑的公钥，内容位于 ~/.ssh/id_rsa.pub 文件。  
2. 新建 Linux ~/.ssh/authorized_keys 文件，将 Win 的公钥内容粘贴进来。  
3. 设置 authorized_keys 文件的权限，只能自己读写：`chmod 0600 ~/.ssh/auhtorized_keys`

## 超时自动退出

可以在 /etc/profile 文件设置变量 $TMOUT，单位为秒。

## 查询机器上次启动时间

```
who -b
```


## 查看操作系统信息

```
cat /etc/os-release
```

或者
```
ls-release -a
```
## 查看 CPU 信息
```
lscpu
```

或者
```
cat /proc/cpuinfo
```

## 查看进程占用的内存

使用 ps 命令，查看指定进程的内存占用，rss Resident Set Size 表示占用物理内存大小，不包括 Swap，单位 KB。
```
# ps -o pid,rss,comm -p 170260
    PID   RSS COMMAND
 170260 29016 java
```

使用 top 命令，可以指定进程 ID。使用 Shift + M 可以按照内存使用排序。RES  Resident Set Size 表示占用物理内存大小，不包括 Swap，单位 KB。
```
# top -p 170260
……
    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                                                                    
 170260 root      20   0 2384596  29016  15980 S   0.3   1.7   0:02.49 java
```

使用 `/proc/<PID>/status` 文件查询进程信息，包括内存占用。
```
# cat /proc/170260/status | grep VmRSS
VmRSS:     29016 kB  
```