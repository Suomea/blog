**​PAM​**​（​**​Pluggable Authentication Modules​**​，可插拔认证模块）是 ​**​Linux/Unix 系统中的一个核心安全框架​**​，用于统一管理各类应用程序的认证机制。它通过模块化设计，允许系统管理员灵活配置身份验证、授权、密码策略等功能，而无需修改应用程序代码。

应用程序，libpam.so 框架 API，具体的 so 库为具体的实现。

查看安装的库
```
find /lib /usr/lib -name 'pam_*.so'
```

PAM 的配置位于 `/etc/pam.d/`目录，每个服务（如 `sshd`、`sudo`）有独立的配置文件。

工作流程，当执行 passwd 之后：
1. 执行 passwd 这个程序，并输入密码。
2. passwd 会调用 PAM 模块进行验证。
3. PAM 会找到 /etc/pam.d/passwd 配置文件，可能是 passwd 调用接口的时候传的配置文件路径。
4. PAM 模块就会解析配置文件，根据配置文件的设定进行验证分析。
5. 将验证结果（成功，失败）回传给 passwd。
6. passwd 获取结果之后，决定下一个动作。

​**​文件示例​**​：/etc/pam.d/passwd
```
# 可以看出可以引用其他的配置文件
@include common-password

# /etc/pam.d/common-password
password        requisite                       pam_pwquality.so retry=3 minlen=8 ucredit=-1 lcredit=-1 dcredit=-1
password        [success=1 default=ignore]      pam_unix.so obscure use_authtok try_first_pass sha512
password        requisite                       pam_deny.so
password        required                        pam_permit.so

```
​
具体的每行配置
第一个字段：验证类别
- auth 认证，主要用来检验使用者的身份。
- account 账号，主要进行授权时检验是或否有正确的权限。
- session 会话。
- passwd 密码，密码的修改变更使用。

第二个字段：控制标识
- required 验证成功 success 或者验证失败 failure，都会继续后续的验证流程。
- requisite 失败 failure 立刻返回原程序，并终止后续验证流程。成功 success 继续后续的流程。
- sufficient 成功 success 立刻返回原程序，并终止后续验证流程。失败 failure 继续后续的流程。
- optional 显示讯息，不使用在验证方面的。

第三个字段：调用的 PAM 模块。

第四个字段：调用 PAM 模块的参数。
