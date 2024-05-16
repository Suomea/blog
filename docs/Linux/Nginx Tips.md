## 关闭 Server 响应头
默认 `Nginx` 会在响应头 `Server` 中携带版本信息，在 `http` 域中添加如下配置隐藏 `Server` 响应头中的版本号。
参考：https://nginx.org/en/docs/http/ngx_http_core_module.html
```
server_tokens off;
```