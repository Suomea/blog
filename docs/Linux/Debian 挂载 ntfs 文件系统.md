需要安装 ntfs-3g 驱动，支持 ntfs 文件系统。

查看是否已经安装了该驱动：
```
dpkg -l | grep ntfs
```

安装驱动：
```
apt install ntfs-3g
```

临时挂载：
```
mount -t ntfs-3g /dev/sda1 /data/disk1
```

