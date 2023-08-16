## 文件属性
`ls` 命令常用的选项如下：
```
ls - list directory contents
       -a, --all
              do not ignore entries starting with .
       -h, --human-readable
              with -l and -s, print sizes like 1K 234M 2G etc.
       -i, --inode
              print the index number of each file
       -l     use a long listing format
       -S     sort by file size, largest first
       -t     sort by time, newest first; see --time
```

使用 `ls -al` 命令，输入如下，以最后一行文件 `logcli-linux-amd64.zip` 为例介绍输出包括的几个部分。
```
root@suomea08:~# ls -la
total 964136
drwx------  3 root root      4096 Jun 28 22:48 .
drwxr-xr-x 20 root root      4096 May 14 21:31 ..
-rw-------  1 root root      9428 Jun 24 23:20 .bash_history
-rw-r--r--  1 root root       571 Apr 11  2021 .bashrc
-rw-r--r--  1 root root 194042837 May 14 21:13 jdk-8u202-linux-x64.tar.gz
-rw-r--r--  1 root root  16554943 Jun 24 15:15 logcli-linux-amd64.zip
……
```
### 文件类型和权限
`-rw-r--r--`
#### 第一个字符表示文件的类型

- `d` 表示目录。
- `-` 表示文件。
- `l` 表示链接文件。
- `b` 表示设备文件里面的可供存储的接口设备，如磁盘（可使用 `ls -l /dev/sd*` 验证）。
- `c` 表示设备文件里面的串行端口设备，如键鼠。

#### 文件的权限
接下来的字符中，以三个为一组，且均为 `[rwx]` 的三个参数的组合。这个三权限的位置不会改变，如果没有对应的权限，就会出现减号 `-`。

对于文文件来说：

- `r` 代表可读。
- `w` 代表可写，但不包含删除该文件。
- `x` 该表该文件具有可以被系统执行的权限。 

对目录来说：

- `r` read contents in directory，表示读取目录结构的权限，可以查询该目录下的文件名数据。
- `w` modify contents of directory，表示具有修改目录结构列表的权限，包括一下部分：
    - 建立新的文件和目录。
    - 删除已经存在的文件和目录，不论该文件和目录的权限如何。
    - 将已经存在的文件或目录进行更名。
    - 搬移该目录内的文件、目录位置。
- `x` access directory，表示能够进入该目录成为工作目录。


每个文件都可以对拥有者、所属群组和其他人三个角色分别设置权限。  

- 第一组为文件拥有者可具备的权限。
- 第二组为加入此群组账号的权限。
- 第三组为非本人且没有加入本群组的其他账号的权限。


### 角色属性介绍

### 链接

### 拥有者和所属群组

### 文件大小

### 日期

### 文件名

### LS 参数

## 文件和目录