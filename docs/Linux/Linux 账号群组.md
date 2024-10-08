## 账号
Linux 并不会直接认识账号名称，只认识账号ID（UID）和群组ID（GID）。Linux 登录进入 Shell 的流程：

1. 先去 /etc/passwd 里面查找账号，如果有的话将 UID 和 GID 以及账号的家目录和 shell 配置一起读出来。
2. 核对密码，去 /etc/shadow 文件找出密码进行比对，如果正确就能够进入 shell。

/etc/passwd 的文件结构：
```
# head -n 4 /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
```
每一行共有七个部分，每个部分使用冒号隔开。

- 账号。
- 密码，早期的 Unix 将密码放在这个字段上，但是由于这个文件的读权限所有人都有，所以后来将密码改到 /etc/shadow 文件中，因此这个字段目前都是 “x”。
-  账号的 UID。

	0：系统管理员，如果想要让其它账号具备 root 权限，只需要将账号的 UID 修改为 0 即可。
	
	1~499：系统账号，UID 除了 0 之外其它的并没有什么权限特性。默认 500 以下的数字让给系统作为保留账号只是一个习惯。
	
	由于系统上一些启动的服务我们可能希望以比较小的权限去运行，不希望以 root 的身份去运行，所以可以使用这个区间的 UID 创建账号，这些账号通常是不可登录的。
	
	通常有细分：1~99 有系统自行创建的账号。100~499 用户有系统账号需求时可以使用的 UID。
	
	500~65535: 给一般的使用者（新版本的内核支持的 UID 为 4 字节大小）。
	
- GID 用户的组ID，与 /etc/group 文件相关。
- 用户信息说明。
- 用户的家目录，登录之后 shell 默认的工作目录。
- shell，登录之后取得的 shell 用来与内核进行沟通。

/etc/shadow 文件结构：
```
# head -n 4 /etc/shadow
root:$y$j9T$lRTGiRgqLqrcjL2OoYzIf1$EiahK3dX/eO9tz6Y6ZTOvFZOHcWWeZA9rlXlKXVDyb.:19592:0:99999:7:::
daemon:*:19592:0:99999:7:::
bin:*:19592:0:99999:7:::
sys:*:19592:0:99999:7:::
```

- 账号名称

- 密码，通过更新密码可以让登录暂时失效。

- 最近更新密码的日期，从 1970.1.1 到更新密码日期的天数。

- 密码不可变动的天数（以第三个字段为起始），距离最近更新密码的日期。0 表示随时可以更新密码，如果设置为 20 的话，表示更新密码之后的 20 天只能不能再次更新密码。

- 密码需要变更的天数（以第三个字段为起始）。设置为 20 的话，表示更新密码之后的 20 天内必须要更新密码，否则密码将会过期，再次登录的话需要更新密码才能登录。

- 密码过期之前的警告天数，如果设置为 7，表示如果还有 7 天就要过期的话，这天内每次登录都会提示更新密码。

- 密码过期之后的宽限日期（第五个字段设置的过期时间），设置为 7 表示，表示密码过期之后 7 天之内没有更新密码的话，密码失效，没有更改的机会也不能在登录了。

- 账号失效日期，1970 年以来的天数，失效之后不能在使用该账号（无论密码是否过期）。

- 保留字段。

## 群组

/etc/group 文件结构：
```
# head -n 4 /etc/group
root:x:0:
daemon:x:1:
bin:x:2:
sys:x:3:
```

- 组名。
- 群组密码，很少使用，通常不需要配置，并且已经移动到 /etc/gshadow。
- GID 群组ID。
- 群组支持的账号，即群组里有哪些用户，用户名逗号隔开。

### 有效群组和初始群组

一个账号能够加入多个群组，在 /etc/passwd 中的 GID 就是用户的**初始群组**，默认登录进来就拥有这个群组的权限。比如 root 用户和 root 群组，root用户在登录进来之后查看 /etc/passwd 的 GID 就能获得 root 群组的权限，也就是初始群组，所以在 /etr/group 中就不需要配置 root 账号的 root 群组权限了。

使用 suomea 账号，查看 suomea 账号的有效群组

suomea@suomea01:~$ groups

suomea cdrom floppy audio dip video plugdev users netdev

使用 groups 命令查看群组，默认第一个就是有效群组，如果此时创建文件，那么文件的所属的群组就是当前账号的有效群组。

使用 newgrp 命令切换有效群组，前提是切换的群组是当前账号已经加入群组。

suomea@suomea01:~$ newgrp users

suomea@suomea01:~$ groups

users cdrom floppy audio dip video plugdev netdev suomea

需要注意的是 newgrp 命令会新建一个 shell，新 shell 的有效群组改为了 users，但是如果使用 exit 退出新 shell 的话，原来的 shell 的有效群组仍然是 suomea。

## 用户管理

### 新增用户

useradd 命令。
```
useradd [-u UID] [-g 初始群组名称] [-G 次要群组] [-mM] [-c 账号说明] [-d 家目录的绝对路径] [-s shell] 账号名称

-u: 接 UID，直接手动指定账号的 UID。

-g：指定用户的初始群组名称。

-G：指定用户所属的群组。

-M：强制不建立用户的家目录。

-m：强制建立用户的家目录。

-c：账号说明。

-d：指定用户的家目录，绝对路径。

-r: 建立一个系统账号，UID 会有限制，参考 /etc/login.defs

-s：指定用户登录后的 shell。如果指定：-s /sbin/nologin 则用户不能登录主机

-e：指定失效日期，格式 yyyy-MM-dd。。
```

新建用户，可以看到默认新建了同名的组，指定了家目录和 shell，但是默认没有设置密码。

```
# useradd suomea
# grep suomea /etc/passwd /etc/shadow /etc/group
/etc/passwd:suomea:x:1002:1003::/home/suomea:/bin/sh
/etc/shadow:suomea:!:19615:0:99999:7:::
/etc/group:suomea:x:1003:
```

查看创建用户参数的默认值
```
root@VM-4-5-debian:~# useradd -D
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/sh
SKEL=/etc/skel
CREATE_MAIL_SPOOL=no
LOG_INIT=yes
```
- GROUP=100
	GID 等于 100 的默认是 users 群组，系统自带的。但是 useradd suomea 创建的 suomea 账号并不是属于 users 群组，而是新增了一个 suomea 群组。机制问题：
	- 私有群组机制：系统会建立一个与账号一样的群组作为账号的初始群组。这样比较有保密性，每个账号都有自己的群组。
	- 公共群组机制：以 GID 100 作为新建账号的初始群组。
	- 如何设定群组机制：
	
- HOME=/home：账号家目录的基准目录。

- INACTIVE=-1：指定 /etc/shadow 第七个字段，-1 标识密码永远不会失效。

- EXPIRE=：账号的失效日期，指定 /etc/shadow 第八个字段。
 
- SHELL=/bin/sh：默认使用 shell 的文件名，如果不允许账号登录 shell，可以指定为 /sbin/nologin。
 
- SKEL=/etc/skel：用户家目录的参考目录，新建用户家目录的文件都是从 /etc/skel 复制过去。

还有一些其他的参数会影响用户新建的行为参考 /etc/login.defs，里面配置了家目录的权限等等。

用户创建默认会使用以下文件或者目录
- /etc/default/useradd  命令
- /etc/login.defs 一些配置信息
- /etc/skel/* 家目录的模板目录

### 修改密码

useradd 新建账号之后是没有设置密码的，也就无法登录，可以使用 **passwd** 命令给账号设置密码。
```
passwd [-l] [-u] [-S] 账号

-l: 是密码失效，会在 /etc/shadow 第二栏最前面加上 !，使密码失效。

-u: 与 -l 相对，是 unlock 的意思。

-S：列出密码相关的参数，即 shadow 文件内的大部分信息。
	root@suomea06:~# passwd -S root
	root P 2023-08-25 0 99999 7 -1

-n：后面接天数，shadow 的第四个字段。

-x: 后面接天数，shadow 的第五个字段。

-w：后面接天数，shadow 的第六个字段。

-i: 后面接日期，shadow 的第七个字段。
```

修改 suomea 账号的密码  `passwd suomea`，修改当前登录账号的密码 `passwd`。

也可使使用 **chage** 命令查看密码的详细参数信息
```
chage [-ldEImMw] 账号名称

-l：列出账号的详细密码参数
	root@suomea06:~# chage -l suomea
	Last password change                                        : Aug 25, 2023
	Password expires                                        : never
	Password inactive                                        : never
	Account expires                                                : never
	Minimum number of days between password change                : 0
	Maximum number of days between password change                : 99999
	Number of days of warning before password expires        : 7

-d：后面接日期，修改 shadow 第三个字段，yyyy-MM-dd

-E：后面接日期，修改 shadow 第八个字段，yyyy-MM-dd

-I：后面接天数，修改 shadow 第七个字段。

-m：后面接天数，修改 shadow 第四个字段。

-M：后面接天数，修改 shadow 第五个字段。

-W：后面接天数，修改 shadow 第六字段。
```

### 更新用户

如果使用 useradd 命令添加用户时指定了错误的选项，当然可以直接修改 /etc/passwd 或者 /etc/shadow 文件，也可以使用 **usermod** 进行账号的细部设定更新。
```
usermod [-cdegGlsuLU] 账号

-c：后面接账号说明，即 /etc/passwd 第五栏的说明

-d: 后面接账号的家目录，即修改 /etc/passwd 的第六栏。

-e：后面接日期，shadow 第八个字段，yyyy-MM-dd 格式。

-f：后面接天数，shadow 第七个字段。

-g：后面接初始群组，修改 /etc/passwd 第四个字段。

-G：后面接次要群组，是这个账号加入这个群组，修改的时 /etc/group

-l: 后面接账号名称，修改账号名称，即 /etc/passwd 的第一栏。

-s: 后面接 shell 的可执行文件。

-u: 后面接 UID 数字，修改 /etc/passwd 第三栏。

-L: 暂时冻结用户密码，使用户无法登录。仅修改 /etc/shadow 的密码栏。

-U：将 shadow 密码栏的 ! 拿掉。解冻。
```

### 删除用户

**userdel** 删除用户相关的数据
- 账号密码相关的参数 /etc/passwd，/etc/shadow
- 群组相关的参数 /etc/group，/etc/shadow
- 个人文件数据 /home/username，/var/spool/mail/username 

确定要完全删除用户之前，最好先查找一下用户在系统中有哪些文件，find / -user username

```
userdel [-r] username

-r: 连同用户的家目录一起删除。

确定要完全删除用户之前，最好先查找一下用户在系统中有哪些文件，find / -user username
```

### 查询自己的 UID

```
id [username]
```

### 新增群组

```
groupadd [-g gid] [-r] 组名

-g: 指定 GID

-r：建立系统群组
```

### 修改群组

```
groupmod [-g gid] [-n group_name] 群组名称

-g 修改现有的群组 GID

-n 修改现有的组名
```

### 删除群组

删除群组名称是有限制的，确保没有账号的初始群组之后才能允许删除该群组。
```
groupdel 群组名称
```
