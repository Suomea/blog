安装
```
apt install nfs-kernel-server
systemctl enable nfs-kernel-server --now
```


配置
```
# vim /etc/exports
# cat /etc/exports 
/data/disk1 *(rw,sync,no_subtree_check,insecure,fsid=0)
```
1. **`fsid=0`​**​：绕过 macOS 对文件系统边界的严格校验，避免因路径层级导致的挂载失败。
2. ​**​`insecure`​**​：允许 macOS 使用非特权端口连接，解决 `Operation not permitted` 错误。

重新加载配置目录
```
exportfs -ra
```
- ​**​`exportfs`​**​: NFS 服务的管理工具，用于控制导出的共享目录。
- ​**​`-r`​**​: 重新导出（re-export）所有在 `/etc/exports` 中定义的共享目录。
- ​**​`-a`​**​: 对所有共享目录进行操作（包括导出和取消导出）。

组合 `-ra` 表示：​**​重新加载 `/etc/exports` 中的所有配置，立即生效​**​。