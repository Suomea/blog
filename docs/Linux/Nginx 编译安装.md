## 准备环境
### 安装编译环境
```shell
apt install gcc make
```

### 安装必要的支持库
pcre 提供 http rewrite 支持
```shell
apt install  libpcre3-dev zlib1g-dev libssl-dev libgd-dev
```
## 安装 Nginx
### 下载 
```shell
wget https://nginx.org/download/nginx-1.22.1.tar.gz

tar -zxvf nginx-1.22.1.tar.gz -C /usr/local/src
```

### 编译安装
完整的编译参数参考：http://nginx.org/en/docs/configure.html
```shell
cd /usr/local/srv/nginx-1.22.1

./configure --prefix=/usr/local/nginx \
--with-stream \
--with-stream_ssl_module \
--with-http_ssl_module

 实现 TCP 和 UDP 流量的转发和负载，实现四层代理
 实现 TCP/UDP 流量的加密
 实现 HTTPS 协议的支持

make && make install 
```

--with-http_ssl_module                              # 启用ssl支持
### 启动命令
```shell
cd /usr/local/nginx

# 启动
./sbin/nginx 

# 验证配置文件是否正确
./sbin/nginx -t

# 重启
./sbin/nginx -s reload

# 停止 nginx
./sbin/nginx -s stop

# 查看版本
./sbin/nginx -v
```

### 配置为服务
编辑配置文件 `/etc/systemd/system/nginx.service`：
```text
[Unit]
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs/
After=network.target

[Service]
Type=forking
LimitNOFILE=65535
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
ExecReload= /usr/local/nginx/sbin/nginx -s reload
ExecStop= /usr/local/nginx/sbin/nginx -s stop
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

配置服务开机启动：
```shell
systemctl daemon-reload

# 配置开机启动
systemctl enable nginx 

# 启动
systemctl start nginx

# 修改配置后重新加载
systemctl reload nginx

# 查看状态
systemctl status nginx
```

## 解释
### 编译命令的解释
```text
# 模块参数具体功能 
--with-cc-opt='-g -O2 -fPIE -fstack-protector'   # 设置额外的参数将被添加到CFLAGS变量。（FreeBSD或者ubuntu使用）
--param=ssp-buffer-size=4 -Wformat -Werror=format-security -D_FORTIFY_SOURCE=2' 
--with-ld-opt='-Wl,-Bsymbolic-functions -fPIE -pie -Wl,-z,relro -Wl,-z,now' 

--prefix=/usr/share/nginx                        # 指向安装目录
--conf-path=/etc/nginx/nginx.conf                # 指定配置文件
--http-log-path=/var/log/nginx/access.log        # 指定访问日志
--error-log-path=/var/log/nginx/error.log        # 指定错误日志
--lock-path=/var/lock/nginx.lock                 # 指定lock文件
--pid-path=/run/nginx.pid                        # 指定pid文件

--http-client-body-temp-path=/var/lib/nginx/body    # 设定http客户端请求临时文件路径
--http-fastcgi-temp-path=/var/lib/nginx/fastcgi     # 设定http fastcgi临时文件路径
--http-proxy-temp-path=/var/lib/nginx/proxy         # 设定http代理临时文件路径
--http-scgi-temp-path=/var/lib/nginx/scgi           # 设定http scgi临时文件路径
--http-uwsgi-temp-path=/var/lib/nginx/uwsgi         # 设定http uwsgi临时文件路径

--with-debug                                        # 启用debug日志
--with-pcre-jit                                     # 编译PCRE包含“just-in-time compilation”
--with-ipv6                                         # 启用ipv6支持
--with-http_ssl_module                              # 启用ssl支持
--with-http_stub_status_module                      # 获取nginx自上次启动以来的状态
--with-http_realip_module                 # 允许从请求标头更改客户端的IP地址值，默认为关
--with-http_auth_request_module           # 实现基于一个子请求的结果的客户端授权。如果该子请求返回的2xx响应代码，所述接入是允许的。如果它返回401或403中，访问被拒绝与相应的错误代码。由子请求返回的任何其他响应代码被认为是一个错误。
--with-http_addition_module               # 作为一个输出过滤器，支持不完全缓冲，分部分响应请求
--with-http_dav_module                    # 增加PUT,DELETE,MKCOL：创建集合,COPY和MOVE方法 默认关闭，需编译开启
--with-http_geoip_module                  # 使用预编译的MaxMind数据库解析客户端IP地址，得到变量值
--with-http_gunzip_module                 # 它为不支持“gzip”编码方法的客户端解压具有“Content-Encoding: gzip”头的响应。
--with-http_gzip_static_module            # 在线实时压缩输出数据流
--with-http_image_filter_module           # 传输JPEG/GIF/PNG 图片的一个过滤器）（默认为不启用。gd库要用到）
--with-http_spdy_module                   # SPDY可以缩短网页的加载时间
--with-http_sub_module                    # 允许用一些其他文本替换nginx响应中的一些文本
--with-http_xslt_module                   # 过滤转换XML请求
--with-mail                               # 启用POP3/IMAP4/SMTP代理模块支持
--with-mail_ssl_module                    # 启用ngx_mail_ssl_module支持启用外部模块支持
```

### Nginx 配置文件的解释
```text
# 全局参数设置 
worker_processes  1;          # 设置nginx启动进程的数量，一般设置成与逻辑cpu数量相同 
error_log  logs/error.log;    # 指定错误日志 
worker_rlimit_nofile 65535;   # 设置一个nginx进程能打开的最大文件数 
pid        /var/run/nginx.pid; 
events {                      # 事件配置
    worker_connections  10240; # 设置一个进程的最大并发连接数
    use epoll;                # 事件驱动类型
} 
# http 服务相关设置 
http {  
    log_format  main  'remote_addr - remote_user [time_local] "request" '
                      'status body_bytes_sent "$http_referer" '
                      '"http_user_agent" "http_x_forwarded_for"'; 
    access_log  /var/log/nginx/access.log  main;    #设置访问日志的位置和格式 
    sendfile          on;      # 用于开启文件高效传输模式，一般设置为on，若nginx是用来进行磁盘IO负载应用时，可以设置为off，降低系统负载
    tcp_nopush        on;      # 减少网络报文段数量，当有数据时，先别着急发送, 确保数据包已经装满数据, 避免了网络拥塞
    tcp_nodelay       on;      # 提高I/O性能，确保数据尽快发送, 提高可数据传输效率                           
    gzip              on;      # 是否开启 gzip 压缩 
    keepalive_timeout  65;     # 设置长连接的超时时间，请求完成之后还要保持连接多久，不是请求时间多久，目的是保持长连接，减少创建连接过程给系统带来的性能损耗，类似于线程池，数据库连接池
    types_hash_max_size 2048;  # 影响散列表的冲突率。types_hash_max_size 越大，就会消耗更多的内存，但散列key的冲突率会降低，检索速度就更快。types_hash_max_size越小，消耗的内存就越小，但散列key的冲突率可能上升
    include             /etc/nginx/mime.types;  # 关联mime类型，关联资源的媒体类型(不同的媒体类型的打开方式)
    default_type        application/octet-stream;  # 根据文件的后缀来匹配相应的MIME类型，并写入Response header，导致浏览器播放文件而不是下载
    # 虚拟服务器的相关设置 
    server { 
        listen      80;        # 设置监听的端口 
        server_name  localhost;        # 设置绑定的主机名、域名或ip地址 
        charset koi8-r;        # 设置编码字符 
        location / { 
            root  /var/www/nginx;           # 设置服务器默认网站的根目录位置 
            index  index.html index.htm;    # 设置默认打开的文档 
        }
         
        error_page  500 502 503 504  /50x.html; # 设置错误信息返回页面 
            location = /50x.html { 
            root  html;        # 这里的绝对位置是/var/www/nginx/html 
        } 
    } 
 }
```
