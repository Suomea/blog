## docker 部署
```shell
docker run --detach --name vaultwarden \
  --env DOMAIN="https://xxxx" \
  --env DATABASE_URL=postgresql://vaultwarden:password@xxxx:5432/vaultwarden \
  -e SMTP_HOST=smtp.qq.com \
  -e SMTP_FROM=xxxx@qq.com \
  -e SMTP_PORT=587 \
  -e SMTP_SECURITY=starttls \
  -e SMTP_USERNAME=xxxx@qq.com \
  -e SMTP_PASSWORD=xxxx \
  -e SIGNUPS_ALLOWED=false \
  --volume ./data/:/data/ \
  --restart unless-stopped \
  --publish 127.0.0.1:80:80 \
  vaultwarden/server:latest
```

## 二进制安装部署
准备
mysql

创建账号、创建数据库、账号权限。

nginx
```
	server {
        listen 7443 ssl;
    
        ssl_certificate /etc/nginx/certs/vaultwarden/vaultwarden-chain.crt;
        ssl_certificate_key /etc/nginx/certs/vaultwarden/vaultwarden.key;

        location / {
            proxy_pass http://localhost:8000;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /notifications/hub {
            proxy_pass http://127.0.0.1:8000;

            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
    }
```


从 Docker 镜像里面获取二进制文件
```
$ wget https://raw.githubusercontent.com/jjlin/docker-image-extract/main/docker-image-extract
$ chmod +x docker-image-extract
$ ./docker-image-extract vaultwarden/server:latest-alpine
```

二进制服务运行

创建用户
```
groupadd vaultwarden 
useradd -r -g vaultwarden -s /bin/false vaultwarden
```

环境变量
```
DATABASE_URL=mysql://vaultwarden:xxxxxx@localhost:3306/vaultwarden
SIGNUPS_ALLOWED=true

# 如果密码中有 @ 字符，需要使用 %40 替换
```

服务文件
```
[Unit]
Description=Vaultwarden Server (Rust Edition)
Documentation=https://github.com/dani-garcia/vaultwarden

# Mysql
After=network.target mysql.service
Requires=mysql.service

[Service]
# 设置 Vaultwarden 用户/群组。此用户/群组对工作目录（见下文）允许有读写权限
User=vaultwarden
Group=vaultwarden
# 使用环境文件进行配置
EnvironmentFile=/srv/vaultwarden/vaultwarden.env
# 已编译的二进制的位置
ExecStart=/srv/vaultwarden/vaultwarden
# 设置合理的连接和进程限制
LimitNOFILE=65535
LimitNPROC=64
# 将 bitwarden_rs 与系统的其他部分隔离开
PrivateTmp=true
PrivateDevices=true
ProtectHome=true
ProtectSystem=strict
# 仅允许对以下目录进行写入，并将其设置为工作目录
WorkingDirectory=/srv/vaultwarden
ReadWritePaths=/srv/vaultwarden

[Install]
WantedBy=multi-user.target
```

备份

安全