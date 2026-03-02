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

下面是 AI 给出的根证书，中间证书，服务器证书生成命令，待验证。
```
1. 生成 root 私钥
   openssl genrsa -out root-ca.key 4096
   
```
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