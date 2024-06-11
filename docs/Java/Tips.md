## 消息队列

### 消费异常处理

RabbitMQ 或者 Kafka 的异常处理机制。

该怎么控制消费逻辑。如果一直不停的重新消费可能会造成严重的问题，如果说有三步消费逻辑，第一步下载文件，第二步发生了异常，那么不停的重新消费肯能会导致磁盘问题。

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

第一步将表达式提取出来，并且按照涵义命名：
```java
boolean isPizzaFantastic = (!pepperoniService.empty() || sausages.size() > 0)
    && (useOnionFlag.get() || hasMushroom(ENOKI, PORTOBELLO)) && hasCheese();
if (isPizzaFantastic) {
  ...
}
```

第二步进一步将表达式进行逻辑拆分：
```java
boolean hasGoodMeat = !pepperoniService.empty() || sausages.size() > 0;
boolean hasGoodVeggies = useOnionFlag.get() || hasMushroom(ENOKI, PORTOBELLO);
boolean isPizzaFantastic = hasGoodMeat && hasGoodVeggies && hasCheese();
if (isPizzaFantastic) {
  ...
}
```

或者将判断逻辑单独拆分为一个方法。
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