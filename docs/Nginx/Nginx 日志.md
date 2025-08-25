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
   '"remote_user":"$remote_user",'
   '"request":"$request",'
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

## 日志归集存储分析
**Doris 存储**
如果并发不是很高，可以使用 logstash/filebeat 进行采集日志，然后 Stream Load 导入到 Doris。  
如果并发很高，那么 logstash/filebeat 直接推送到 Kafka，然后 Routine Load 导入到 Doris。

**Loki 存储**
可以使用 promtail 进行采集并推送到 Loki 进行存储，Grafana 集成 Loki 进行可视化分析。