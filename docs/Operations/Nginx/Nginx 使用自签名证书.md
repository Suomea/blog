注意 Nginx 编译安装时需要带上 --with-http_ssl_module 选项，已经安装的 Nginx 可以使用 nginx -V 命令查看是否包含该选项。

自签名证书类型

- 自签名证书
- 自建 CA 签名的证书

自签名证书无法被吊销，CA 签名的证书可以被吊销。

## 自签名证书

生成私钥
```shell
openssl genrsa -out client.key 4096
```

生成证书签名请求 CSR
```shell
openssl req -new -key client.key -out client.csr
```

生成证书，证书中包含公钥。
```shell
openssl x509 -req -days 365 -in client.csr -signkey client.key -out client.crt
```


将私钥和证书移动到 Nginx 配置目录下
```
mv client.key client.crt /usr/local/nginx/conf/mainssl/
```

## 创建自签 CA 证书
创建 CA 的私钥和证书：
```shell
openssl genrsa -out ca.key 4096
  
openssl req -new -x509 -key ca.key -sha256 -days 3650 -out ca.crt \
  -subj "/C=CN/ST=Shanghai/L=Shanghai/O=OTTO Inc./OU=IT Department/CN=ca.otto.com"
```

使用 CA 的证书签发网站的证书：
```shell
# 需要输入密码
openssl req -new -newkey rsa:2048 -keyout nginx.key -out nginx.csr \
  -subj "/C=CN/ST=Shanghai/L=Shanghai/OU=OTTO"
  
openssl x509 -req -in nginx.csr -CA ca.crt -CAkey ca.key -out nginx.crt -days 3650 -sha256 \
  -extfile <(printf "basicConstraints=CA:FALSE\nkeyUsage=digitalSignature,keyEncipherment\nextendedKeyUsage=serverAuth\nsubjectAltName=DNS:localhost,DNS:localhost,IP:127.0.0.1,IP:172.29.222.43")
```

这样签发的网站私钥需要输入密码，即网站的私钥被密码保护。

subj 用来描述证书的主体身份；ext 信息证书的扩展字段，用于定义证书的用途、约束、信任链信息以及可绑定的域名/IP。

在 Chrome 浏览器中导入 CA 的证书，然后访问网站即可。需要注意的是，网站的证书 `subjectAltName` 字段要包含访问网站的域名或者 IP。

Chrome "设置"-"隐私与安全"-"安全"-"管理证书"。

## Nginx 配置
配置 Nginx：
```
server {
        listen       80 ssl;
        server_name  localhost;

        ssl_certificate /etc/nginx/ssl/nginx.crt;
        ssl_certificate_key /etc/nginx/ssl/nginx.key;
        ssl_password_file /etc/nginx/ssl/.passwords;
		
        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }
}
```

文件 `/etc/nginx/ssl/.passwords` 的内容为签发网站私钥时输入的密码，每个密码一行，Nginx 会逐个尝试。

参考连接：[如何用 OpenSSL 创建自签名证书](https://support.azure.cn/docs/azure-operations-guide/application-gateway/aog-application-gateway-howto-create-self-signed-cert-via-openssl.html)


## 自建 CA 签名证书
### 根 CA
创建目录
```
mkdir -p ~/certs/{root,intermediate,server}
```

生成根证书私钥：
```
openssl genrsa -aes256 -out rootCA.key 4096
```

生成自签名的根证书：
```
openssl req -x509 -new -nodes -key rootCA.key \
  -sha256 -days 3650 -out rootCA.crt \
  -subj "/C=CN/ST=Shanghai/L=Shanghai/O=Jacky Company/OU=IT Department/CN=Jacky Root Global CA"
  
# 查看证书信息
openssl x509 -in rootCA.crt -text -noout
```
根证书不需要指定 ext 信息，默认 `CA:TRUE` 表示是信任链顶端的证书。

### 中间 CA
生成中间证书私钥：
```
openssl genrsa -aes256 -out intermediateCA.key 4096
```

生成中间 CA 的证书签名请求（CSR）：
```
openssl req -new -key intermediateCA.key -out intermediateCA.csr \
  -subj "/C=CN/ST=Shanghai/L=Shanghai/O=Jacky Company/OU=Security Department/CN=Jacky Intermediate CA"
```

用根证书为中间 CA 的 CSR 签名，生成中间证书：
```
openssl x509 -req -in intermediateCA.csr -CA ../root/rootCA.crt -CAkey ../root/rootCA.key \
  -CAcreateserial -out intermediateCA.crt -days 1825 -sha256 \
  -extfile <(printf "basicConstraints=CA:TRUE\nkeyUsage=keyCertSign,cRLSign,digitalSignature")
  
# 查看证书信息
openssl x509 -in intermediateCA.crt -text -noout
```

### 网站服务端证书
生成网站证书的私钥
```
openssl genrsa -aes256 -out server.key 2048
```

生成网站证书的 CSR
```
openssl req -new -key server.key -out server.csr \
  -subj "/C=CN/ST=Shanghai/L=Shanghai/O=Jacky Company/OU=Web Site/CN=www.jacky.com"
```

准备 SAN 配置文件
```
cat > server.ext << EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = www.example.com
DNS.2 = example.com
DNS.3 = app.example.com
IP.1 = 192.168.1.100
EOF
```

用中间 CA 证书为网站 CSR 签名，生成网站证书
```
openssl x509 -req -in server.csr -CA ../intermediate/intermediateCA.crt -CAkey ../intermediate/intermediateCA.key \
  -CAcreateserial -out server.crt -days 1095 -sha256 \
  -extfile server.ext
  
# 查看证书
openssl x509 -in server.crt -text -noout
```

验证证书链
```
openssl verify -CAfile ../root/rootCA.crt -untrusted ../intermediate/intermediateCA.crt server.crt
# 应输出: server.crt: OK
```

合成证书链文件
```
cat server.crt ../intermediate/intermediateCA.crt > server-chain.crt
```

Nginx 配置
```
server {
    listen 443 ssl;

    # 指定证书文件
    ssl_certificate     /etc/nginx/certs/server/server-chain.crt; # 合成的证书链文件
    ssl_certificate_key /etc/nginx/certs/server/server.key;      # 服务器私钥
    ssl_password_file /etc/nginx/certs/server/.passwords;

    location / {
        root   html;
        index  index.html index.htm;
    }
}

# HTTP 强制跳转 HTTPS
server {
    listen 80;

    return 301 https://$host$request_uri;
}
```