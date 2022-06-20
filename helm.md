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

```sh
# helm repo remove [REPO1 [REPO2 ...]] [flags]
helm repo remove ali-stable stablecharts
```

### 3.1.3 列出库

```sh
helm repo list
```

### 3.1.4 其他

```sh
# 列出helm环境变量
helm env
# helm版本查看
helm version
```



# 4. helm常用命令

## 4.1 搜索镜像

```sh
# 从中央仓库搜索
helm search hub nginx
# 从自己添加的库中搜索
helm search repo nginx
```

## 4.2 安装与卸载

```sh
# 安装 -n代表要安装到k8s的哪个namespace下, helm-test要在k8s中先创建出来
helm install mynginx apphub/nginx -n helm-test
# 查看安装的内容
helm list -n helm-test

# 查看状态
helm status mynginx -n helm-test

# 使用 k8s命令查看安装的内容 
kubectl get deploy,svc,pod -n helm-test -o wide

# 卸载
helm uninstall mynginx -n helm-test
```

## 4.3 升级，查询历史与回滚

```sh
# 1.拉取离线配置到本地
helm pull apphub/nginx

# 1.1 命令参数升级
helm upgrade --set service.port=8080 mynginx . -n helm-test
# 1.2 修改values.yaml方式升级
helm upgrade -f values.yaml mynginx . -n helm-test

# 2.查看版本历史
helm history mynginx -n helm-test

# 3.回滚到指定版本
helm rollback mynginx 1 -n helm-test

# 4.在线查看在库中已存在的 chart配置
helm show chart apphub/nginx

```

## 4.4 离线修改安装

```sh
# 以redis为例子,由于别人制作的redis需要pv，而我们仅是测试，所以需要禁用pvc的配置
helm pull apphub/redis
tar -zxvf redis-10.5.3.tgz

# 修改无需密码连接 修改无需使用pv 修改无需使用主从配置
# 修改values.yaml的一些内容

# 安装
helm install my-edit-redis . -n helm-test

# 测试连接安装好的redis
# redis-cli -h host -p port -a password

```

## 4.5 自定义模板

```sh
# 使用命令创建helm模板文件,根据需要修改内容或新增yaml文件，删除yaml文件
helm create hyy-my-nginx-charts

# 简单验证文件编写是否存在较大的问题
helm lint ./hyy-my-nginx-charts

# 安装
helm install hyy-my-nginx-charts ./hyy-my-nginx-charts -n helm-test

# 查看部署的helm文件
helm get manifest hyy-my-nginx-charts -n helm-test

# 打包分发
helm package hyy-my-nginx-charts

# 上传到仓库 该命令未真实使用
helm push hyy-my-nginx-charts-0.1.0.tgz myRepoName
```

## 4.6 helm插件

```sh
# 比如插件地址 https://github.com/chartmuseum/helm-push；若无法正常连接，可下载下来后离线安装
# 查看已安装插件
helm plugin list

# 安装插件 helm plugin install [options] <path|url>... [flags]
helm plugin install https://github.com/chartmuseum/helm-push

# 卸载插件 
helm plugin uninstall pluginName

# 更新插件
helm plugin update pluginName
```



## 4.7 其他

```sh
# 查看说明信息
helm get notes hyy-myedit-nginx -n helm-test
# 查看values配置
helm get values hyy-myedit-nginx -n helm-test

```



## 4.x 参考文档

- https://www.hellodemos.com/hello-helm-command/helm-command-helm-repo-remove.html
- https://v2.helm.sh/docs/developing_charts/#predefined-values
- https://juejin.cn/post/6844904199818313735
- helm模板语法解释
  - https://blog.csdn.net/weixin_45015255/article/details/117217876
  - http://www.bubuko.com/infodetail-3666223.html
- helm常用函数
  - https://www.csdn.net/tags/Ntjagg2sNjUxMjYtYmxvZwO0O0OO0O0O.html