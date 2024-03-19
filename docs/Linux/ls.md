#Linux

## 文件的时间：

- mtime，modification time 当文件的内容变更时，就会更新这个时间。
- ctime，status time 当文件的状态改变时就会更新这个时间。权限和属性改变时会更新这个时间。
- atime，access time 当文件的内容被读取时，更新这个时间。(根据内核参数配置，这个时间不一定被更新)

ls -l 展示的时间就是 mtime

## 常用的选项

`-t` sort by modification time, newest first

`-S` sort by file size, largest first

`-r` reverse order while sorting

`-h` with -l and/or -s, print human readable sizes (e.g., 1K 234M 2G)

`--time-style` with -l, show times using style STYLE: full-iso, long-iso, iso, locale, or +FORMAT; FORMAT is interpreted like in 'date'

可以编辑 `~/.bashrc` 设置别名。
```shell
alias ll='ls -l --time-stype=long-iso'
```





