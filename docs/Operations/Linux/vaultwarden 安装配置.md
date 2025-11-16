docker 部署
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
使用 PG 数据库，参考 github

取消注册

nginx https 代理
```
openssl genrsa -out root.key 4096

openssl req -new -x509 -key root.key -sha256 -days 3650 -out root.crt \
  -subj "/C=CN/ST=Shanghai/L=Shanghai"
  
  
openssl req -new -newkey rsa:2048 -nodes -keyout vaultwarden.key -out vaultwarden.csr \
  -subj "/C=CN/ST=Shanghai/L=Shanghai"

openssl x509 -req -in vaultwarden.csr -CA root.crt -CAkey root.key -CAcreateserial \
  -out vaultwarden.crt -days 3650 -sha256 \
  -extfile <(printf "basicConstraints=CA:FALSE\nkeyUsage=digitalSignature,keyEncipherment\nextendedKeyUsage=serverAuth\nsubjectAltName=DNS:localhost,IP:127.0.0.1,IP:xxxx")
```

浏览器安装插件