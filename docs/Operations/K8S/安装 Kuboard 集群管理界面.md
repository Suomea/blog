在 K8S 中安装 Kuboard，参考 https://kuboard.cn/install/v3/install-in-k8s.html。安装环境参考：[安装 Kubeneters 集群](../安装 Kubernetes 集群)。

之所以单独记录 Kuboard 的安装，是因为根据官方的安装文档进行操作会遇到两个问题：  

- 拉取镜像非常慢，官方有解决方案：采用华为云镜像代替。
- kuboard pod 启动失败。

## 安装前的准备

### 下载 yaml 清单文件
```shell
wget https://addons.kuboard.cn/kuboard/kuboard-v3-swr.yaml
```

### 修改 Kuboard 的启动参数
删除 ConfigMap kuboard-v3-config 的配置项 `KUBOARD_SERVER_NODE_PORT: '30080'`，然后新增如下配置项：
```text
KUBOARD_ENDPOINT: 'http://192.168.31.11:30080'
```

## 启动 Kuboard

### 启动命令
```shell
kubectl apply -f kuboard-v3-swr.yaml
```

### 访问地址
http://192.168.31.11:30080

