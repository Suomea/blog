## **准备环境**
官方的安装要求如下：

- 一台或多台运行 `deb/rmp` 兼容的 Linux 操作系统。
- 每台机器 `2GiB` 或者更多的内存。
- 运行 `control-plane` 的节点至少需要 2CPUs。（补充：官网就是这么写的，我的理解可能是两个核心。具体配置为 Hyper-V 中“虚拟处理器的数量”。）
- 集群节点之间的网络需要完全联通。

### 配置信息
我准备安装在 Hyper-V 管理的三台 Linux 虚拟机上，虚拟机均为 Debian 操作系统。具体配置如下表格，如无特别说明，后续所有的安装配置
在三台机器上都要进行。

| HOSTNAME | IP | RAM | CPUs | DESCRIPTION |
| :-- | :-- | :-- | :-- | :-- |
| suomea01 | 192.168.31.11 | 16GiB | 2 | 主节点，角色为 control-plane。 |
| suomea02 | 192.168.31.12 | 16GiB | 1 | 工作节点。 |
| suomea03 | 192.168.31.13 | 16GiB | 1 | 工作节点。|

### 配置 Host 信息
编辑 `/etc/hosts` 文件，添加如下配置：
```text
192.168.31.11     suomea01
192.168.31.12     suomea02
192.168.31.13     suomea03
```

## **安装容器运行时（CRI）**
每个节点都要安装容器运行时，这样 Pod 才能在节点上运行，所以下面的操作需要在每个机器上执行。

### 配置网络
需要做两件事情： 

一、加载 br_netfilter 模块
```shell
cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter
```
二、配置 iptables，能够让 iptables 检测到桥接的网络流量。
```shell
# sysctl params required by setup, params persist across reboots
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sysctl --system
```
### 安装 Containerd
安装命令：
```shell
apt install containerd
```

生成 `Containerd` 的配置文件：
```shell
containerd config default | tee /etc/containerd/config.toml
```

编辑配置文件，修改或者添加的地方有两处，如下：
```toml
[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    ···
    sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.2"
          ···
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true
```

重启 `Containerd`：
```shell
systemctl restart containerd
```

## **安装 K8S 集群**

### 关闭 swap
Kubernetes 1.8开始要求关闭系统的Swap，如果不关闭，默认配置下kubelet将无法启动。

> 没有关闭 swap 分区的情况下，kubeadm init 集群时的告警信息：
> swap is enabled; production deployments should disable swap unless testing the NodeSwap feature gate of the kubelet
> 
> 没有关闭 swap 分区的情况下，kubelet 启动的错误信息（默认配置下会启动 kubelet 失败，进而导致 control-plane 节点初始化失败）：
> "command failed" err="failed to run Kubelet: running with swap on is not supported, please disable swap! or set --fail-swap-on flag to false. ···

临时关闭 swap 分区使用 `swapoff -a` 命令；注释掉 `/etc/fstab` 文件中挂载 swap 分区的行，这样下次开机时就不会自动挂载 swap 分区了。

### 关闭 SELinux

### 安装 kubelet、kubeadm 和 kubectl
kubeadm：用来初始化启动集群。  
kubelet：集群的组件，运行在所有的节点上面并负责 Pod 和容器的管理。  
kubectl：和集群交互的命令行工具。  

[下载公钥](../files/apt-key.gpg)：
```shell
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
```

配置镜像：
```shell
deb https://mirrors.tuna.tsinghua.edu.cn/kubernetes/apt kubernetes-xenial main
```

安装：
```shell
apt install kubeadm kubectl kubelet
```



编辑 ~/.bashrc，配置相关命令的自动补全，然后使用 `source ~/.bashrc` 使配置生效。
```text
source <(kubectl completion bash)
source <(kubeadm completion bash)
source <(crictl completion bash)
```

### 配置 crictl
编辑配置文件 /etc/crictl.yaml：
```yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
```

### 初始化集群配置（仅在 control-plane 节点）
使用 `kubeadm version` 命令查看即将要安装 K8S 的版本；用 `kubeadm config images list` 命令查看初始化集群需要的镜像列表；

生成集群初始化配置文件：
```shell
kubeadm config print init-defaults > k8s.yaml
```

编辑配置文件，主要修改以下几个地方：
```yaml
apiVersion: kubeadm.k8s.io/v1beta3
···
kind: InitConfiguration
localAPIEndpoint:
  # 1. 修改为 control-plane 所在机器的 IP
  advertiseAddress: 192.168.31.11
  bindPort: 6443
nodeRegistration:
  ···
  # 2. 修改为 control-plane 所在机器的 hostname
  name: suomea01
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
···
# 3. 使用阿里云的镜像
imageRepository: registry.aliyuncs.com/google_containers
kind: ClusterConfiguration
···
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
```

初始化集群：
```shell
kubeadm init --config=k8s.yaml
```

如果初始化成功的话就可以看到如下信息，然后按照指示操作即可。
```text
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.31.11:6443 --token abcdef.0123456789abcdef \
--discovery-token-ca-cert-hash sha256:*****************************************
```

### 安装 Calico 网络插件（仅在 control-plane 节点）
下载配置文件：
```shell
curl https://projectcalico.docs.tigera.io/manifests/calico.yaml -O
```

编辑配置文件，`value` 的值为 `k8s.yaml` 配置文件 `networking.serviceSubnet` 属性的值。
```yaml
- name: CALICO_IPV4POOL_CIDR
  value: 10.96.0.0/12
```

安装网络插件：
```shell
kubectl apply -f calico.yaml
```

然后使用如下命令查看所有 Pod 的状态，当 Status 全部为 Running 时，即 K8S 集群安装成功。
```shell
kubectl get pod --all-namespaces
```

## 一些可能用到的命令

### 重置集群
集群初始化过程中可能出现错误导致失败，可以使用如下命令重置：
```shell
kubeadm reset
```

### 强制删除 Pod
使用 kubectl delete 清除资源的使用，可能导致 Pod 一只处于 Terminating 状态，无法删除，使用如下命令强制删除 Pod：
```shell
kubectl -n ns_name delete pod pod_name --force --grace-period=0
```
