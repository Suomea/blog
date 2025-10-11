记录 Linux 安装和配置的相关技巧，主要基于 Debian 进行操作。

## 配置静态 IP
备份网卡配置文件
```text
cp /etc/network/interfaces /etc/network/interfaces.bak
```

编辑配置文件
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

重启网络
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

Win 生成密钥要对
```shell
ssh-keygen -t rsa
```

复制 Win 电脑的公钥，内容位于 ~/.ssh/id_rsa.pub 文件。  

新建 Linux ~/.ssh/authorized_keys 文件，将 Win 的公钥内容粘贴进来。  

设置 authorized_keys 文件的权限，只能自己读写：`chmod 0600 ~/.ssh/auhtorized_keys`

## 设置会话超时自动退出

可以在 /etc/profile 文件设置变量 $TMOUT，单位为秒。

## 系统信息查询
### 查询机器上次启动时间

```
who -b
```


### 查看操作系统信息

```
cat /etc/os-release
```

或者
```
ls-release -a
```
### 查看 CPU 信息
```
lscpu
```

或者
```
cat /proc/cpuinfo
```

### 查看 CPU 温度
直接查看 CPU 温度
```
# cat /sys/class/thermal/thermal_zone0/temp 
27800
# cat /sys/class/thermal/thermal_zone1/temp 
20000
# cat /sys/class/thermal/thermal_zone2/temp 
51000
# cat /sys/class/thermal/thermal_zone3/temp 
44000
```
### 查看进程占用的内存

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

### 查看 Intel 核显占用情况
安装必要的工具包
```
apt install intel_gpu_tools
```

查看占用情况
```
intel_gpu_top
```
## Debian 镜像安装包
```
debian-12.10.0-amd64-DVD-1.iso 4G
debian-12.10.0-amd64-netinst.iso 633M
```

DVD 的安装包包含了大量的软件和工具包，安装的时候不需要联网。netinst 只包含基本的安装程序和核心组件，安装额外的组件需要联网。

## ls
常用的选项
```
-t sort by modification time, newest first

-S sort by file size, largest first

-r reverse order while sorting

-h with -l and/or -s, print human readable sizes (e.g., 1K 234M 2G)

--time-style with -l, show times using style STYLE: full-iso, long-iso, iso, locale, or +FORMAT; FORMAT is interpreted like in 'date'
```

可以编辑 `~/.bashrc` 设置别名。
```shell
alias ll='ls -l --time-stype=long-iso'
```

文件的时间：
```
- mtime，modification time 当文件的内容变更时，就会更新这个时间。
- ctime，status time 当文件的状态改变时就会更新这个时间。权限和属性改变时会更新这个时间。
- atime，access time 当文件的内容被读取时，更新这个时间。(根据内核参数配置，这个时间不一定被更新)
```
ls -l 展示的时间就是 mtime
## tar
tar 用于创建、查看和解包归档文件（archive files），归档文件通常以 .tar 为扩展名，可以将多个文件和目录打包成一个文件，便于传输或备份。  
tar 本身并不压缩文件，但可以与其他压缩工具结合使用，生成压缩的归档文件 tar.gz、tar.bz2、tar.xz 等。用法
```
tar [选项] [归档文件名] [文件或目录列表]

-c 创建归档文件。
-x 解包归档文件。

-z 使用 gzip 压缩或解压缩（tar.gz 或 tgz）。
-j 使用 bzip2 压缩或解压缩（tar.bz2）。
-J 使用 xz 压缩或解压缩 （tar.xz）。

-f 指定生成或者处理的归档文件名称。

-v 打印正在处理的文件。
-C 指定解压缩处理后的目录。
```

示例
```
tar -Jxvf mysql-8.4.4-linux-glibc2.28-x86_64.tar.xz -C /usr/local/
```

## 查看硬盘温度
安装工具
```
apt install smartmontools
```

查看温度
```
smartctl -a /dev/sda | grep Temperature
```

## base64
查看用法
```
man base64
```

编码
```
echo -n hello | base64
```

解码
```
echo -n aGVsbG8= | base64 -d
```

## Unix Socket 和 TCP/IP Socket
Unix Socket 叫做本地套接字，使用文件系统路径作为通信端点，只能在同一台机器上使用。地址类型为 AF_UNIX。

不经过 TCP/IP 直接内核进行通信，系统调用少延迟低。安全性依赖文件权限。

进程 A 打开 socket 文件发送数据，内存在内存中把数据传给同一机器上的进程 B。

TCP/IP Socket 叫做网络套接字，可以在同一台机器或者跨机器通信，使用 IP 加上端口号作为通信端点。地址类型为 AF_INET/AF_INET6。

使用 TCP 或者 UDP，经过内核网络栈，需要网络协议开销，性能略低于 Unix Socket。

