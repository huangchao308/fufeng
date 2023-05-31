# 在 Mac 上安装 K8S


本篇文章将介绍如何在 Mac 上使用 minikube 搭建单机版的 Kubernetes。

## 安装步骤
### 安装 Docker 

安装 docker 主要是用于提供容器引擎。直接下载安装即可。

[下载地址](https://www.docker.com/products/docker-desktop/)

### 安装 Kubectl

推荐使用 home brew 安装

```bash
brew install kubectl
```

可以使用下面的命令查看是否已经安装完毕

```bash
kubectl version --client
```

### 安装 Minikube

依然推荐使用 home brew 安装

```bash
brew install minikube 
```

可以使用下面的命令查看是否已经安装完毕

```bash
minikube -h
```

如果能输出一下信息则安装成功

```bash
minikube provisions and manages local Kubernetes clusters optimized for development workflows.

Basic Commands:
  start            Starts a local Kubernetes cluster
  status           Gets the status of a local Kubernetes cluster
  stop             Stops a running local Kubernetes cluster
  delete           Deletes a local Kubernetes cluster
  dashboard        访问在 minikube 集群中运行的 kubernetes dashboard
  pause            pause Kubernetes
  unpause          恢复 Kubernetes

Images Commands:
  docker-env       Provides instructions to point your terminal's docker-cli to the Docker Engine inside minikube.
(Useful for building docker images directly inside minikube)
  podman-env       配置环境以使用 minikube's Podman service
  cache            Manage cache for images
  image            Manage images

Configuration and Management Commands:
  addons           Enable or disable a minikube addon
  config           Modify persistent configuration values
  profile          Get or list the current profiles (clusters)
  update-context   Update kubeconfig in case of an IP or port change

Networking and Connectivity Commands:
  service          Returns a URL to connect to a service
  tunnel           连接到 LoadBalancer 服务

Advanced Commands:
  mount            将指定的目录挂载到 minikube
  ssh              Log into the minikube environment (for debugging)
  kubectl          Run a kubectl binary matching the cluster version
  node             添加，删除或者列出其他的节点
  cp               将指定的文件复制到 minikube

Troubleshooting Commands:
  ssh-key          Retrieve the ssh identity key path of the specified node
  ssh-host         Retrieve the ssh host key of the specified node
  ip               Retrieves the IP address of the specified node
  logs             Returns logs to debug a local Kubernetes cluster
  update-check     打印当前和最新版本版本
  version          打印 minikube 版本
  options          显示全局命令行选项列表 (应用于所有命令)。

Other Commands:
  completion       Generate command completion for a shell
  license          Outputs the licenses of dependencies to a directory

Use "minikube <command> --help" for more information about a given command.
```
## 简单使用 minikube

### 启动 Minikube

```bash
minikube start
```

在启动的过程中，我遇到了如下错误

```bash
➜  workspace minikube start
😄  Darwin 12.6 (arm64) 上的 minikube v1.30.1
✨  自动选择 docker 驱动
📌  Using Docker Desktop driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.26.3 preload ...
    > preloaded-images-k8s-v18-v1...:  330.52 MiB / 330.52 MiB  100.00% 6.16 Mi
    > index.docker.io/kicbase/sta...:  336.39 MiB / 336.39 MiB  100.00% 3.35 Mi


❗  minikube was unable to download gcr.io/k8s-minikube/kicbase:v0.0.39, but successfully downloaded docker.io/kicbase/stable:v0.0.39 as a fallback image
🔥  Creating docker container (CPUs=2, Memory=4000MB) ...

❌  Exiting due to RSRC_DOCKER_STORAGE: Docker is out of disk space! (/var is at 100% of capacity). You can pass '--force' to skip this check.
💡  建议：

    Try one or more of the following to free up space on the device:

    1. Run "docker system prune" to remove unused Docker data (optionally with "-a")
    2. Increase the storage allocated to Docker for Desktop by clicking on:
    Docker icon > Preferences > Resources > Disk Image Size
    3. Run "minikube ssh -- docker system prune" if using the Docker container runtime
🍿  Related issue: https://github.com/kubernetes/minikube/issues/9024
```

解决办法也是按照其提示清理了一下 docker 的数据

```bash
docker system prune
```

万万没想到，这一下直接清理出了 2.561GB 的空间🐶

### 查看 Minikube 状态

```bash
minikube status

# 如果是如下输出，说明启动成功
# minikube
# type: Control Plane
# host: Running
# kubelet: Running
# apiserver: Running
# kubeconfig: Configured
```

### 打开 K8S Dashboard

```bash
minikube dashboard
```

运行上述命令后，会调起系统的默认浏览器，dashboard UI 如图所示

![kubernetes-dashboard](/fufeng/images/kubernetes-dashboard.png)


