## Java Jar 包部署脚本

```shell
#!/bin/bash

APP_NAME=<jar_file>
APP_JVM_OPTS=<jvm_options>

usage() {
    echo "Usage: sh service.sh [start|stop|restart|status]"
    exit 1
}

is_exist() {
    pid=`ps -ef | grep ${APP_NAME} | grep -v grep | awk '{print $2}'`
    if [ -z "${pid}" ]; then 
        return 1
    else 
        return 0
    fi
}

start() {
	is_exist
	if [ $? -eq "0" ]; then
		echo "${APP_NAME} is already running, pid=${pid}"
	else 
		nohup java $APP_JVM_OPTS -jar $APP_NAME --spring.profiles.active=stage >/dev/null 2>&1 &
	fi
}

stop() {
	is_exist
	if [ $? -eq "0" ]; then 
		kill -9 $pid
	else 
		echo "$APP_NAME is not running"
	fi
}

status() {
	is_exist
	if [ $? -eq "0" ]; then
		echo "${APP_NAME} is running. pid is ${pid}"
	else 
		echo "$APP_NAME is not running"
	fi
}

restart() {
	stop
	start
}

case "$1" in
	"start") start ;;
	"stop") stop ;;
	"status") status ;;
	"restart") restart ;;
	*) usage;;
esac
```

## 如何简化布尔表达式
原文链接：https://testing.googleblog.com/2024/04/isbooleantoolongandcomplex.html
原始表达式：
```java
if ((!pepperoniService.empty() || sausages.size() > 0)
	&& (useOnionFlag.get() || hasMushroom(ENOKI, PORTOBELLO)) && hasCheese()) {
	...
}
```

将表达式进行逻辑拆分：
```java
boolean hasGoodMeat = !pepperoniService.empty() || sausages.size() > 0;
boolean hasGoodVeggies = useOnionFlag.get() || hasMushroom(ENOKI, PORTOBELLO);
boolean isPizzaFantastic = hasGoodMeat && hasGoodVeggies && hasCheese();
if (isPizzaFantastic) {
  ...
}
```

将判断逻辑单独拆分为一个方法。
```java
boolean isPizzaFantastic() {
  if (!hasCheese()) {
    return false;
  }
  if (pepperoniService.empty() && sausages.size() == 0) {
    return false;
  }
  return useOnionFlag.get() || hasMushroom(ENOKI, PORTOBELLO);
}
```

## 文本文件转码
将目录下的所有以 `.cue` 结尾的文件，从 `GBK` 转到 `UTF-8`。

Java 版本：
```java
    private static final String DIR = "";

    public static void main(String[] args) throws IOException {
        Files.walk(Paths.get(DIR)).forEach(path -> {
            String fileName = path.toAbsolutePath().toString();
            if (fileName.endsWith(".cue") || fileName.endsWith(".CUE")) {
                try {
                    Files.writeString(path, Files.readString(path, Charset.forName("GBK")), StandardCharsets.UTF_8);
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
        });
    }
```

Python 版本：
```python
import os

for root, dirs, files in os.walk(""):
    for file in files:
        if file.endswith(".cue"):
            current_path = os.path.join(root, file)
            try:
                # 使用 GBK 编码尝试读取
                with open(current_path, "r", encoding="GBK") as f:
                    content = f.read()
                    print(f"Content from {current_path}:\n{content}\n{'=' * 50}\n")

                    # 将内容保存为 UTF-8 编码
                with open(current_path, "w", encoding="UTF-8") as f:
                    f.write(content)
                print(f"Successfully converted {current_path} from GBK to UTF-8.")
            except UnicodeDecodeError:
                print(f"Failed to decode {current_path} with GBK encoding. Trying GB18030...\n")
                try:
                    with open(current_path, "r", encoding="GB18030") as f:
                        content = f.read()
                        print(f"Content from {current_path}:\n{content}\n{'=' * 50}\n")

                    # 将内容保存为 UTF-8 编码
                    with open(current_path, "w", encoding="UTF-8") as f:
                        f.write(content)
                    print(f"Successfully converted {current_path} from GB18030 to UTF-8.")
                except UnicodeDecodeError:
                    print(f"Failed to decode {current_path} with GB18030 encoding.\n")
```
## HTTP 401 和 403 的区别
401 Unauthorized 表示客户端需要进行身份认证才能获取请求的响应。

403 Forbidden 表示客户端没有访问内容的权限，也就是说它是未经授权的，因此服务器拒绝提供请求的资源。

403 表示服务器知道客户端的身份，而 401 服务器不知道客户端的身份。

## 配置允许跨域
Spring Boot 后端配置处理：
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**") // 允许所有路径
                .allowedOriginPatterns("*") // 允许所有来源
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS") // 允许的 HTTP 方法
                .allowedHeaders("*") // 允许所有请求头
                .allowCredentials(true) // 允许携带凭证（如 cookies）
                .maxAge(3600); // 预检请求的缓存时间（秒）
    }
}
```

如果有 Filter 拦截请求需要在 Filter 中放行：
```java
// 如果是 OPTIONS 请求，直接放行  
if ("OPTIONS".equalsIgnoreCase(request.getMethod())) {  
    filterChain.doFilter(servletRequest, servletResponse);  
    return;  
}
```

## MybatisPlus 配置 SQL 输出到控制台
```properties
# myabtis  
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

## Java 日期时间
获取指定的时间：
```java
// 时间截取
LocalDateTime now = LocalDateTime.now();
// 2025-03-21 21:42:31.062
System.out.println(now.format(dtf));
// 获取当前秒的起始时间 2025-03-21 21:42:31.000
System.out.println(now.truncatedTo(ChronoUnit.SECONDS).format(dtf));
// 获取当前分钟的起始时间 2025-03-21 21:42:00.000
System.out.println(now.truncatedTo(ChronoUnit.MINUTES).format(dtf));
// 获取当前小时的起始时间 2025-03-21 21:00:00.000
System.out.println(now.truncatedTo(ChronoUnit.HOURS).format(dtf));
// 获取当天中午的起始时间 2025-03-21 12:00:00.000
System.out.println(now.truncatedTo(ChronoUnit.HALF_DAYS).format(dtf));
// 获取当天的起始时间 2025-03-21 00:00:00.000
System.out.println(now.truncatedTo(ChronoUnit.DAYS).format(dtf));
// 获取当月第一天的起始时间 2025-03-01 00:00:00.000
System.out.println(now
        .with(TemporalAdjusters.firstDayOfMonth())
        .truncatedTo(ChronoUnit.DAYS)
        .format(dtf)
);
// 获取当年第一天的起始时间 2025-01-01 00:00:00.000
System.out.println(now
        .with(TemporalAdjusters.firstDayOfYear())
        .truncatedTo(ChronoUnit.DAYS)
        .format(dtf)
);
```

获取两个 LocalDateTime 之间的间隔：
```java
Duration duration = Duration.between(startTime, endTime);
duration.toDays();
duration.toHours();
duration.toMinutes();
duration.getSeconds();
```

## 防止接口重复请求
前端没有做防抖，或者说因为网络抖动问题，同一时间重复请求多次接口。比如用户下单接口，同一时间参数一样请求了两次。

前段防抖，禁用按钮。

后端处理
1. 后段基于业务逻辑判断，同一个用户 5 秒之内只能下一单。Redis 里面缓存一个 key 就好了。
2. 前端生成一个 requestId，前段需要保证下单界面多次点击下单按钮 requestId 是同样的。后端就先判断是否处理过这个 requestId，已经处理过就不再处理，否则 Redis 缓存 requestId 然后处理请求。
3. 如果是修改业务数据，那么需要使用分布式锁进行处理，防止多个用户同时修改业务数据，导致数据不一致。