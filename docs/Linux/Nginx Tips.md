## 关闭 Server 响应头
默认 `Nginx` 会在响应头 `Server` 中携带版本信息，在 `http` 域中添加如下配置隐藏 `Server` 响应头中的版本号。
参考：https://nginx.org/en/docs/http/ngx_http_core_module.html
```
server_tokens off;
```

## Nginx location 的匹配用法

location 的使用语法：
```
location [=|~|~*|^~|@] pattern {...}
```

### 完全匹配 =
假设：
```
location /test {
	...
}
```

请求 path /test ✅
请求 path /TEST 取决与操作系统的文件系统是否大小写敏感。
请求 path /test?param=1 匹配，忽略 querstring
请求 path /test/ 不匹配
请求 path /testabc 不匹配

### 区分大小写的正则匹配 ~
假设：
```
location ~ ^/test {
	...
}
```

### 不分区大小写的正则匹配 ~*

### 

## Nginx location 和 proxy_pass

### 如果 proxy_pass 仅包含 schema、host 和 port
- ✅ http://localhost:8080 
- ❌ http://localhost:8080/
- ❌ http://localhost:8080/api

那么 location 匹配到的 path 会追加到 proxy_pass 的 schema、host 和 port 后面。

假设：
```
location /test1 {
	proxy_pass http://localhost:8080;
}
```

请求 path /test1/abc 将被转发到：http://localhost:8080/test1/abc。

### 如果 proxy_pass 除了 schema、host 和 port 外，还包含 path
即使 path 仅仅只是一个 /，那么转发规则就变成：将匹配的 path 删除 location 匹配的部分，然后将得到的字符串拼接到 proxy_pass 后面。

```
location /test1 {
	proxy_pass http://localhost:8080/;
}
```

请求 path /test1/abc 将被转发到：http://localhost:8080//abc。