## Helm

官网地址：https://helm.sh/

# 1. helm的概念

## 1.1 定义

Helm 是 Kubernetes 的包管理器，可以与Linux中的apt和yum包管理工具做类比

## 1.2 Helm的三大核心内容

- Chart：用于定义创建Kubernetes应用程序所必需的一组信息
- Repository：Charts仓库，是一个存储区域。
- Release：是一个与特定配置相结合的chart的运行实例，可以理解为运行的容器（使用不同的配置命令运行起来的docker镜像-容器）。

Chart(定义k8s的一些资源信息) -> Config(修改定义的资源信息) ->Release(生成一个可以运行的实例，也即是新的运行版本)

## 1.3 程序架构

- Helm客户端：客户端，管理本地的Chart仓库，管理Chart，与Tiller服务器交互，发送Chart，实例安装、查询、卸载等操作

- Helm库（tiller服务器）：服务端，接受helm发来的Charts与Config，合并生成Release；

  <img src="https://upload-images.jianshu.io/upload_images/1709776-539208286fd1bf09.jpg?imageMogr2/auto-orient/strip|imageView2/2/format/webp" style="zoom:50%;" />



## 1.4 参考文档

- https://helm.sh/docs/topics/architecture/
- https://www.jianshu.com/p/aa4efa87bc53
- https://www.jianshu.com/p/f4dd6e07b938

# 2. helm安装

## 2.1 使用脚本安装

不推荐，由于一些网络原因，可能无法下载成功

```sh
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```



## 2.2 离线二进制安装

```sh
# 从https://github.com/helm/helm/releases下载所需要的helm离线包
# 解压后 移动到执行目录 必要时赋予可执行权限 (chmod 命令)
mv linux-amd64/helm /usr/local/bin/helm
```



## 2.3 参考文档

- https://v3.helm.sh/zh/docs/intro/install/

# 3. helm简单配置

## 3.1 配置稳定仓库

### 3.1.1 添加库

```sh
# 主要是添加国内库
helm repo add microsoft http://mirror.azure.cn/kubernetes/charts   #微软的很全推荐
helm repo add bitnami https://charts.bitnami.com/bitnami        #大部分都有
helm repo add harbor https://helm.goharbor.io                   #harbor的
helm repo add nvidia-dcgm https://nvidia.github.io/gpu-monitoring-tools/helm-charts    #NVIDIA DCGM的
helm repo add elastic https://helm.elastic.co                   #elastic的 elasticsearch
helm repo add stablecharts https://charts.helm.sh/stable              #不更新了还是有些东西的
helm repo add ali-stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts # 内容部分陈旧，与新版k8s存在诸多不兼容
helm repo add apphub https://apphub.aliyuncs.com # 这个适合新版k8s配置
# 更新刚配置的库
helm repo update
```

### 3.1.2 删除库

```
# helm repo remove [REPO1 [REPO2 ...]] [flags]
helm repo remove ali-stable stablecharts
```

# 4. helm基础命令



## 4.x 参考文档

- https://www.hellodemos.com/hello-helm-command/helm-command-helm-repo-remove.html
- 