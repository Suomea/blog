## 日志格式
log_format 定义日志的格式，**必须在 http 域中定义**：
```
log_format name [escape=default|json|none] string;
```

escape 指定对输出日志的转义方式，对输出日志的特殊字符、引号、反斜杠如何处理。  
配置日志 JSON 格式输出，如果输出日志某个字段的值包含双引号，那么就会导致日志分析工具 JSON 解析失败，就要使用 json 转义来对输出日志进行处理。

默认配置示例：
```
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';
```
虽然配置文件是三行，但是实际输出日志是一行。如果想要多行输出日志，可以在 log_format 中添加换行符。

log_format 配置变量可以取 Nginx 的内置变量：客户端、服务器、请求、请求头、时间、响应等相关的变量都可以配置输出。如果 log_format 配置的变量值为空，默认输出 `-` 字符。

log_format 可以配置多个。下面实际配置示例，第一个加上了 token 认证请求头输出，第二个加上了 service_name 变量输出：
```
log_format main 
		   '$remote_addr - $remote_user [$time_local] '
           '"$request" $status $body_bytes_sent $request_time '
           '"$http_referer" "$http_user_agent" "$http_token" '
           '$upstream_response_time $upstream_addr';
           
log_format json escape=json
  '{"time":"$time_local",'
   '"remote_addr":"$remote_addr",'
   '"request_method":"$request_method",'
   '"uri":"$uri",'
   '"args":"$args",'
   '"server_protocol":"$server_protocol",'
   '"status":$status,'
   '"body_bytes_sent":$body_bytes_sent,'
   '"request_time":$request_time,'
   '"http_referer":"$http_referer",'
   '"http_user_agent":"$http_user_agent",'
   '"http_token":"$http_token",'
   '"upstream_response_time":"$upstream_response_time",'
   '"service_name":"$service_name",'
   '"upstream_addr":"$upstream_addr"}';
```

!!!note
	如果请求头是 A-b，那么 Nginx 会将请求头映射到变量 $http_a_b。  
	
	如果 log_format 定义中取了 service_name 变量输出，但是 service_name 并不是 Nginx 的内置变量。那么就要在使用 log_format 的域或者上层域中，定义 service_name 变量。
	
	$reuqest 这个变量，如果监听的是 http 用 https 请求，或者反过来，都会导致日志出现乱码。这样可能会导致一些 JSON 解析框架出错。所以建议使用 $uri + $args 替换 $request 变量。

## 访问日志
access_log 默认记录所有的请求，语法：
```
access_log path [format [buffer=size] [gzip[=level_]] [flush=time] [if=condition]];  
access_log off;
```

access_log 可以在 http、server、location 块进行设置，如果同时设置的话优先级依次是 location、server、http。

配置示例：
```
    server {
        listen       80;
        server_name  localhost;
        
        set $service_name "otto";
        
        access_log logs/access.log main;
        access_log logs/json.log json;

		……
	}
```

access_log 可以配置多个，写入不同的文件，使用不同的 log_format。

## goaccess 分析
goaccess 是一个本地的 Nginx 日志分析工具，能够快速的分析日志并且生成报告。
### 按天切割日志
编辑文件 /etc/logrotate.d/nginx-access
```
/usr/local/nginx/logs/access.log {
    daily                  # 按天切分日志
    missingok              # 文件不存在不报错
    rotate 180             # 日志保留天数
    notifempty             # 空文件不切分
    create 0640 nobody root  # 新日志文件权限和属主
    sharedscripts
    postrotate
        # 切分后通知 Nginx 重新打开日志文件
        [ -f /usr/local/nginx/logs/nginx.pid ] && kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`
    endscript
}
```

logrotate 本身只是一个工具，依靠 cron 定时任务来进行定时执行。所以要确保 crond 服务是启动的。

配置好之后强制切分一次，验证配置是否生效
```
logrotate -f /etc/logrotate.d/nginx-access
```

### 生成分析报告

生成分析报告，假设日志格式是上文定义的 main log_format：
```
goaccess logs/access.log \
    --log-format='%h - %e [%d:%t %^] "%r" %s %b %T "%R" "%u" "%^" %^ %^' \
    --date-format=%d/%b/%Y \
    --time-format=%T
```

解释：  
- %h → $remote_addr  
- %e → $remote_user  
- %d → 日（来自 $time_local）  
- %t → 时间（来自 $time_local）  
- %r → $request  
- %s → $status  
- %b → $body_bytes_sent  
- %T → $request_time  
- %R → $http_referer  
- %u → $http_user_agent  
- %^ → 忽略字段（比如 $http_token，$upstream_response_time，$upstream_addr）

注意：  
- GoAccess 默认不解析自定义 Header（`$http_token`）和 `$upstream_*` 字段，需要用 `%^` 占位。  
- 日期格式 `%d/%b/%Y` 对应 `[25/Aug/2025:23:00:00 +0800]` 的日期部分。  
- 时间格式 `%T` 对应 `23:00:00`。

## Doris 存储分析
如果并发不是很高，可以使用 logstash/filebeat 进行采集日志，然后 Stream Load 导入到 Doris。  
如果并发很高，那么 logstash/filebeat 直接推送到 Kafka，然后 Routine Load 导入到 Doris。  

Doris 针对 Beats/Logstash 有官方的 Output 插件，能够将数据通过 Http Stream Load 输出到 Drois 中。插件支持 Filebeat。

Nginx 配置日志存储：
```
log_format json escape=json
               '{"time":"$time_iso8601",'
               '"remote_addr":"$remote_addr",'
               '"remote_user":"$remote_user",'
               '"request":"$request",'
               '"scheme":"$scheme",'
               '"request_method":"$request_method",'
               '"uri":"$uri",'
               '"status":$status,'
               '"body_bytes_sent":$body_bytes_sent,'
               '"request_time":$request_time,'
               '"http_referer":"$http_referer",'
               '"http_host":"$http_host",'
               '"http_user_agent":"$http_user_agent",'
               '"http_token":"$http_token",'
               '"http_type":"$http_type",'
               '"upstream_response_time":"$upstream_response_time",'
               '"service_name":"$service_name",'
               '"upstream_addr":"$upstream_addr"}';
access_log /usr/local/nginx/logs/json_access.log json;
```

Doris 建表
```sql
CREATE TABLE `nginx_log` (
  `time` datetime NULL,
  `remote_addr` varchar(50) NULL,
  `uri` varchar(3000) NULL,
  `remote_user` varchar(100) NULL,
  `request` varchar(3000) NULL,
  `scheme` varchar(20) NULL,
  `request_method` varchar(20) NULL,
  `status` int NULL,
  `body_bytes_sent` bigint NULL,
  `request_time` float NULL,
  `http_referer` varchar(500) NULL,
  `http_user_agent` varchar(500) NULL,
  `http_type` varchar(50) NULL,
  `http_token` varchar(500) NULL,
  `upstream_response_time` varchar(50) NULL,
  `service_name` varchar(100) NULL,
  `upstream_addr` varchar(100) NULL,
  `create_time` datetime NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=OLAP
DUPLICATE KEY(`time`, `remote_addr`)
PARTITION BY RANGE(`time`)()
DISTRIBUTED BY RANDOM BUCKETS 10
PROPERTIES (
"replication_allocation" = "tag.location.default: 1",
"dynamic_partition.enable" = "true",
"dynamic_partition.time_unit" = "YEAR",
"dynamic_partition.time_zone" = "Asia/Shanghai",
"dynamic_partition.start" = "-2147483648",
"dynamic_partition.end" = "10",
"dynamic_partition.prefix" = "p",
"dynamic_partition.replication_allocation" = "tag.location.default: 1",
"dynamic_partition.buckets" = "8",
"dynamic_partition.create_history_partition" = "true",
"dynamic_partition.history_partition_num" = "2",
"dynamic_partition.hot_partition_num" = "0"
);
```

下载 Doris 官方的 Beats：https://doris.apache.org/zh-CN/docs/3.0/ecosystem/beats

Filebeat 配置文件：
```
# input 
# 逐行读取，每一行就是一个事件
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /usr/local/nginx/logs/json_access.log

# queue and batch
queue.mem:
  ## 内存最多缓存 100 万条事件
  events: 1000000
  ## 每次至少凑够 10 万条事件才会推送 Doris
  flush.min_events: 100000
  ## 如果 10 秒内没有凑够 10 万条，也会强制推送 Doris
  flush.timeout: 10s

# output
output.doris:
  fenodes: [ "http://fe_host:8030" ]
  user: "root"
  password: "xxxx"
  database: "log_db"
  table: "nginx_log"
  # output string format
  ## 直接把原始文件每一行的 message 原样输出，由于 headers 指定了 format: "json"，Stream Load 会自动解析 JSON 字段写入对应的 Doris 表的字段。
  codec_format_string: '%{[message]}'
  headers:
    format: "json"
    read_json_by_line: "true"
    ## 同一批次的数据写入到一个 tablet 里面去
    load_to_single_tablet: "true"
```

Filebeat 启动：
```shell
 ./filebeat-doris-2.0.0 -c nginx-log.yml
```

## 实战案例
Nginx 配置
```
    log_format j_format escape=json
               '{"time":"$time_iso8601",'
               '"remote_addr":"$remote_addr",'
               '"remote_user":"$remote_user",'
               '"request":"$request",'
               '"scheme":"$scheme",'
               '"request_method":"$request_method",'
               '"uri":"$uri",'
               '"status":$status,'
               '"body_bytes_sent":$body_bytes_sent,'
               '"request_time":$request_time,'
               '"http_referer":"$http_referer",'
               '"http_host":"$http_host",'
               '"http_user_agent":"$http_user_agent",'
               '"upstream_response_time":"$upstream_response_time",'
               '"service_name":"$service_name",'
               '"upstream_addr":"$upstream_addr"}';

    access_log  logs/access.json j_format;
```

Filebeat 采集日志，输出到 LogStash，采用 ssl 进行认证。

生成 CA 私钥和证书：
```shell
openssl genrsa -out ca.key 4096

openssl req -new -x509 -key ca.key -sha256 -days 3650 -out ca.crt \
  -subj "/C=CN/ST=Shanghai/L=Shanghai"
```

生成 Logstash 的私钥和证书：
```shell
openssl req -new -newkey rsa:2048 -nodes -keyout logstash.key -out logstash.csr \
  -subj "/C=CN/ST=Shanghai/L=Shanghai/OU=Logstash"

openssl x509 -req -in logstash.csr -CA ca.crt -CAkey ca.key -CAcreateserial \
  -out logstash.crt -days 3650 -sha256 \
  -extfile <(printf "basicConstraints=CA:FALSE\nkeyUsage=digitalSignature,keyEncipherment\nextendedKeyUsage=serverAuth\nsubjectAltName=DNS:localhost,IP:127.0.0.1,IP:192.168.31.157")
```
注意，服务端证书 SAN 需要指定 ip 地址或者 dns，否则客户端在进行证书校验的时候可能不过。filebeat 也可以配置跳过证书校验。


生成 Filebeat 的私钥和证书：
```shell
openssl req -new -newkey rsa:2048 -nodes -keyout filebeat.key -out filebeat.csr \
  -subj "/C=CN/ST=Shanghai/L=Shanghai/OU=Filebeat"

openssl x509 -req -in filebeat.csr -CA ca.crt -CAkey ca.key -CAcreateserial \
  -out filebeat.crt -days 3650 -sha256 \
  -extfile <(printf "basicConstraints=CA:FALSE\nkeyUsage=digitalSignature,keyEncipherment\nextendedKeyUsage=clientAuth")
```

logstash 配置文件
```
input {
  beats {
    port => 5044
    ssl_enabled => true
    ssl_certificate => "/home/jacky/logstash.crt"       # Logstash 服务端证书
    ssl_key => "/home/jacky/logstash.key"   # Logstash 私钥
    ssl_certificate_authorities => ["/home/jacky/ca.crt"]   # 用于验证 Filebeat 客户端证书
    ssl_client_authentication => "required"                  # 强制验证客户端证书
  }
}

filter {
}

output {
  http {
    url => "http://192.168.31.86:8080/filebeat/receive"  # 目标接口
    http_method => "post"
    format => "json"
    headers => {
      "Content-Type" => "application/json"
    }
    # 可选：超时和重试设置
    # content_type => "application/json"
    # http_proxy => "http://proxy.example.com:8080"
  }
}
```

filebeat 配置文件：
```
filebeat.inputs:
  - type: filestream
    enabled: true
    paths:
      - /usr/local/nginx/logs/access.json

output.logstash:
  # Logstash 地址
  hosts: ["localhost:5044"]

  # 启用 SSL/TLS
  ssl.enabled: true

  # CA 证书，用于验证 Logstash 服务端证书
  ssl.certificate_authorities: ["/home/jacky/ca.crt"]

  # 客户端证书和私钥，用于双向认证
  ssl.certificate: "/home/jacky/filebeat.crt"
  ssl.key: "/home/jacky/filebeat.key"

  # 可选：跳过验证（不推荐）
  ssl.verification_mode: none

```