
## LVM 介绍

磁盘
查看磁盘的分区类型
```
parted /dev/sda print
```

分区
```
fdisk /dev/sda
```

PV
实际的分区，但是要调整类型为 8E00（LVM 的识别码），然后使用 pvcreate 命令将分区转换为 LVM 最底层的实体卷。
!!!note
	GPT 分区类型代码，8300 用作 Linux 文件系统，8e00 用作 Linux LVM。对应 MBR 分区类型代码为 83 和 8e。

pvcreate 将分区创建为 PV
pvscan 展示 PV 列表
pcdisplay 显示系统所有的 PV 状态
pvremove 将分区的 PV 属性移除

VG
卷群组，将多个 PV 整合成 VG。

vgcreate 创建 VG
vgscan
vgdisplay
vgextend 在 VG 内增加额外的 PV
vgreduce 在 VG 内移除 PV
vgremove 删除一个 VG

PE
PE 是 LVM 的最小存储单位。

LV
逻辑卷，VG 被划分为一个或者多个 LV。

lvcreate
lvscan
lvdisplay
lvextend
lvreduce
lvremove
lvresize

## LVM 扩容
fdisk 进行分区

如果磁盘实际剩余有 50GB，但是 fdisk 输出剩余空间明显少于 50GB 可以使用 parted /dev/vda 对分区信息进行修复。

创建 PV
```
pvcreate /dev/vda4
```

加入 VG
```
vgextend <vgname> <pvname>
```

扩展 LV
```
lvextend -l +100%FREE lvpath
```

扩展文件系统
blkid 查看 LV 的文件系统。
如果 ext 系列使用 
```
resize2fs <lvpath>
```

如果 nfs 文件系统使用
```
xfs_growfs <lvpath>
```