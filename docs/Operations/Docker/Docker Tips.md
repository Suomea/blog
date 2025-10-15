## 安装
```shell
curl -fsSL https://get.docker.com -o get-docker.sh 
sh get-docker.sh
```

## 镜像操作
查看镜像详细信息
```shell
docker image inspect image_name
```

导出镜像
```shell
docker image save mysql -o mysql.tar
```

导入镜像
```
docker image load -i mysql.tar
```

镜像拉取代理配置，虚拟机走宿主机的科学上网。
```
# 服务文件 /usr/lib/systemd/system/docker.service 添加环境变量
[Service]
Environment="HTTP_PROXY=http://192.168.3.209:7897"
Environment="HTTPS_PROXY=http://192.168.3.209:7897"

# 重启服务
systemctl daemon-reload
systemctl restart docker
```

## 网络模式
`Docker` 安装时默认会创建三个网络，同时在宿主机上创建一个虚拟网卡 `docker0`。
```shell
# docker network list
NETWORK ID     NAME      DRIVER    SCOPE
b092ef563276   bridge    bridge    local
e8eb78a7d5ff   host      host      local
e1290a9d96db   none      null      local
```

- `host` 使用是宿主机的网络。  
- `bridge` 容器使用独立的 network namespace，并连接到 `docker0` 虚拟网卡。默认。  
- `none` 容器有独立的 network namespace，但是不进行任何网络设置。

docker compose up 会默认创建一个 bridge 网络，名称是目录名称加上 `_default`。

docker compose restart container_name 只会重启某个容器，不会重新加载修改的 yml 配置文件。

如果多个容器位于同一个网络下面，一个容器一个使用另一个容器的名称来访问连接。`curl c_name:80`。