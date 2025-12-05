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
  -subj "/C=CN/ST=Shanghai/L=Shanghai"
```

使用 CA 的证书签发网站的证书：
```shell
openssl req -new -newkey rsa:2048 -nodes -keyout nginx.key -out nginx.csr \
  -subj "/C=CN/ST=Shanghai/L=Shanghai/OU=Logstash"
  
openssl x509 -req -in nginx.csr -CA ca.crt -CAkey ca.key -CAcreateserial \
  -out nginx.crt -days 3650 -sha256 \
  -extfile <(printf "basicConstraints=CA:FALSE\nkeyUsage=digitalSignature,keyEncipherment\nextendedKeyUsage=serverAuth\nsubjectAltName=DNS:localhost,IP:127.0.0.1,IP:xxxxxxxx")
```

在 Chrome 浏览器中导入 CA 的证书，然后访问网站即可。需要注意的是，网站的 `subjectAltName` 字段要包含访问网站的域名或者 IP。

## Nginx 配置
配置 Nginx
```
server {
        listen       80 ssl;
        server_name  localhost;

		ssl_certificate ./ssl/client.crt;
		ssl_certificate_key ./ssl/client.key;
        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }
}
```


参考连接：[如何用 OpenSSL 创建自签名证书](https://support.azure.cn/docs/azure-operations-guide/application-gateway/aog-application-gateway-howto-create-self-signed-cert-via-openssl.html)