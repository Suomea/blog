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

## 网络模式
`Docker` 安装时默认会创建三个网络，同时在宿主机上创建一个虚拟网卡 `docker0`。
```shell
# docker network list
NETWORK ID     NAME      DRIVER    SCOPE
b092ef563276   bridge    bridge    local
e8eb78a7d5ff   host      host      local
e1290a9d96db   none      null      local
```
- host 使用是宿主机的网络。  
- bridge 容器使用独立的 network namespace，并连接到 docker0 虚拟网卡。默认。  
- none 容器有独立的 network namespace，但是不进行任何网络设置。