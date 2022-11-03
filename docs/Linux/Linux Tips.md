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
