
## 前置条件
### 安装编译安装软件

```shell
apt install gcc make
```

### 安装 PCRE

PCRE，Perl Compatible Regular Expressions。正则表达式库，Nginx 用来支持正则表达式处理。
```shell
apt install libpcre3-dev
```

### 安装 zlib 库

Nginx 用来支持 gzip 压缩。
```shell
apt install zlib1g-dev
```
## 安装

### 下载解压缩安装包
```shell
wget https://nginx.org/download/nginx-1.25.4.tar.gz
tar -zxvf nginx-1.25.4.tar.gz -C /usr/local/src/
```
### 配置编译选项

`--prefix` 选项配置安装目录。
```shell
cd /usr/local/src/nginx-1.25.4
./configure --prefix=/usr/local/nginx
```

### 编译安装

```shell
make & install
```

### 启动命令
```shell
sbin/nginx -h    // 查看 nginx 命令选项
sbin/nginx // 启动 
sbin/nginx -s reload // 重新加载配置文件 
sbin/nginx -s quit // 退出
```


## 配置问题

### Linux 对进程句柄的限制


### Worker 进程的数量设置

Nginx 默认是多进程模式，一个 Master 进程多个 Worker 进程，生产环境 Worker 进程的数量一般设置为 CPU 核心数量。

设置 Nginx Worker 进程的数量为 2，编辑 conf/nginx.conf 配置文件。
```
worker_processes  2;
```

重启 Nginx。
```
sbin/nginx -t
sbin/nginx -s reload
```