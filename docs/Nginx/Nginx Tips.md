## URI
参考 **RFC 3986**，完整结构如下：
```
foo://example.com:8042/over/there?name=ferret#nose
\_/   \______________/\_________/ \_________/ \__/
 |          |             |           |       |
scheme  authority        path       query  fragment

scheme:[//authority]path[?query][#fragment]
```

**authority** 部分的格式：`[userinfo@]host[:port]`  

- `userinfo` 部分是可选的，例如：`admin:123456@`。
- `host` 域名或 IP 地址。
- `port` 部分也是可选的，默认根据协议推断。

**fragment** 以 `#` 开头，用于标识资源内的某个部分（如 HTML 锚点），不会发送到服务器，仅由客户端处理。
## Nginx 关闭 Server 响应头
默认 `Nginx` 会在响应头 `Server` 中携带版本信息，在 `http` 域中添加如下配置隐藏 `Server` 响应头中的版本号。
参考：https://nginx.org/en/docs/http/ngx_http_core_module.html
```
server_tokens off;
```


## Nginx 配置允许上传的文件大小
```
server {
    listen       8088 ;
    client_max_body_size 100m; 

    location / { 
        root   /data/frontend/service/dist;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
    }
}   
```
## Nginx location 的匹配用法

`location` 如果不指定修饰符，则默认为前缀匹配。使用语法如下：
```
location [=|~|~*|^~|@] pattern {...}
```
### 完全匹配 `=`
假设 `location` 配置如下：
```
location /test {
	...
}
```

请求路径 `/test` 匹配。  
请求路径 `/TEST` 取决与操作系统的文件系统是否大小写敏感。  
请求路径 `/test?param=1` 匹配，查询参数的部分不参与匹配。  
请求路径 `/test/` 不匹配。  
请求路径 `/testabc` 不匹配。  
### 区分大小写的正则匹配 `~`
假设 `location` 的配置如下 ：
```
location ~ ^/test {
	...
}
```

### 不分区大小写的正则匹配 ~*


## Nginx 接口转发 `location` 和 `proxy_pass`

### 如果 `proxy_pass` 不包含 `path`
```
✅ http://localhost:8080 
❌ http://localhost:8080/
❌ http://localhost:8080/api
```
那么 `location` 匹配到的 `path` 会追加到 `proxy_pass` 的 `schema`、`host` 和 `port` 后面。

假设前缀匹配的配置如下：
```
location /test1 {
	proxy_pass http://localhost:8080;
}
```
请求 `path` `/test1/abc` 将被转发到：`http://localhost:8080/test1/abc`。
### 如果 `proxy_pass` 包含 `path`
即使 `path` 仅仅只是一个 `/`，那么转发规则就变成：将匹配的 `path` 删除 `location` 匹配的部分，然后将得到的字符串拼接到 `proxy_pass` 后面。
```
location /test1 {
	proxy_pass http://localhost:8080/;
}
```
请求 `path` `/test1/abc` 将被转发到：`http://localhost:8080//abc`。

## Nginx 配置静态文件路径 root 和 alias 
root 将 location 匹配的部分直接拼接在 root 指定的目录后面。

alias 将 location 匹配的部分删掉，然后将剩余的部分拼接在 alias 指定的目录后面。alias 一般使用时，location 匹配路径和 alias 指定的目录最后都要带上字符 “/”。 

如果是正则匹配，可以使用 $ 符号加上数字，来获取匹配组进行拼接。