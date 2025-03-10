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

## HTTP 401 和 403 的区别
401 Unauthorized 表示客户端需要进行身份认证才能获取请求的响应。

403 Forbidden 表示客户端没有访问内容的权限，也就是说它是未经授权的，因此服务器拒绝提供请求的资源。

403 表示服务器知道客户端的身份，而 401 服务器不知道客户端的身份。