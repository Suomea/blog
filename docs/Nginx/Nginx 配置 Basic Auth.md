首先使用 openssl 获取加密密码 openssl passwd -5 your-password，比如密码 123456。
```
# openssl passwd -5 123456
$5$jZFBH2tXARpLGmik$YDeK49P73WVVYzfjhWVZG7ZjHXyc7g0MVaWCZXHc0U9
```

然后新增配置文件 conf/.htpasswd。
```
usera:$5$jZFBH2tXARpLGmik$YDeK49P73WVVYzfjhWVZG7ZjHXyc7g0MVaWCZXHc0U9
userb:$5$jZFBH2tXARpLGmik$YDeK49P73WVVYzfjhWVZG7ZjHXyc7g0MVaWCZXHc0U9
```

编辑 Nginx 配置文件。
```
    server {
        listen       80; 
        server_name  localhost;

        auth_basic "Restricted Access";           # 设置提示信息
        auth_basic_user_file ./.htpasswd; # 指定认证文件路径

        location / { 
            root   html;
            index  index.html index.htm;
        }
    }
```

配置文件生效之后，访问主机的 80 端口，使用 usera/123456 和 userb/123456 都能进行访问。