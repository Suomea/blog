## 介绍
在早期的 Unix/Linux 系统中安装软件相当费时费力，系统管理员需要从源码编译软件并为自己的系统做各种调整，甚至修改源码。这样虽然增强了用户定制的自由度，但在各种小的细节上耗费巨大的精力显然是缺乏效率的。

软件包管理使得管理员从各种兼容性的问题中解脱出来，软件包安装使软件安装成为了一系列不可分割的原子操作。一旦发生错误，可以卸载软件包，也可以重新安装。使用软件包系统安装软件同样需要考虑依赖性问题，只有应用软件所依赖的所有库和支持都已经正确安装好了，软件才能被正确安装。

一些高级的软件包工具可以自动解析依赖关系并安装软件。

## dpkg

Debian Pakcage，dpkg --help 查看帮助文档。

### 安装

Debian 使用 dpkg 管理软件包，这些软件包通常以 .deb 结尾。

dpkg 使用 --install 选项安装，这个选项也可以简写为 -i，-i 选项会在安装软件包之前把系统上原有的旧的版本删除。

### 查询

查看当前系统安装的软件版本信息 openssh 为例：
```shell
root@suomea01:~# dpkg -l | grep openssh
ii  openssh-client                          1:9.2p1-2+deb12u1                   amd64        secure shell (SSH) client, for secure access to remote machines
ii  openssh-server                          1:9.2p1-2+deb12u1                   amd64        secure shell (SSH) server, for secure access from remote machines
ii  openssh-sftp-server                     1:9.2p1-2+deb12u1                   amd64        secure shell (SSH) sftp server module, for SFTP access from remote machines
```

查看安装软件向系统中复制了那些文件：
```shell
root@suomea01:~# dpkg -S openssh
openssh-client: /usr/share/doc/openssh-client/changelog.Debian.gz
openssh-server: /usr/share/lintian/overrides/openssh-server
openssh-client: /usr/lib/openssh/ssh-pkcs11-helper
openssh-server: /etc/ufw/applications.d/openssh-server
···
```

### 卸载

使用 --remove 或者 -r 选项卸载已经安装的软件包。

## APT

全称为 Advanced Package Tool，即高级软件包工具。它可以自动检测软件依赖问题，下载和安装所有的文件。

apt -h 查看帮助文档。

### 软件源

所有用于 apt 下载软件的地址通常称为安装源。

Debian 12 配置清华软件源，编辑 `/etc/apt/source.list` 文件。
```
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware

deb https://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
# deb-src https://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
```

## 查找软件

apt search regex

该命令会查找软件名和软件描述，如果软件名或者软件描述匹配查询字符串，则会在输出中展示软件名和简短的描述。

--full：选项则展示全部描述。因为默认搜索全部描述而只展示简短描述，输出可能会造成困惑。

--names-only：选项指定搜锁只匹配软件名称，不搜索软件描述。

### 安装软件

如果软件源只有一个版本的软件，想要安装其它的版本怎么处理？

### 卸载软件
