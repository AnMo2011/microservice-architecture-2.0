### 概述

Kubernetes 已然是当下容器编排领域事实上的标准，各大云服务商都急于推出 Kubernetes 服务，互联网公司也纷纷跟进，将自己的应用容器化，并使用 Kubernetes 编排，在 Kubernetes 图形化工具方面，我们已经获得了极大的可选择空间：

- 各云服务商自己推出的 Kubernetes 服务所搭载的管理控制台，例如 阿里云的 Kubernetes 服务，青云推出的 KubeSphere，其他云服务商的 CaaS 类服务

- Kubernetes 官方的图形管理界面 Kubernetes Dashboard

- 面向企业私有化部署的 Rancher

### 什么是 Kuboard

Kuboard 类似于 Kubernetes Dashboard，是一款轻量级产品，按照 Kuboard 的文档，您可以很快地搭建一套学习 Kubernetes 用的集群环境，您也可以在自己已经有的集群上轻易地完成 Kuboard 的安装

### 安装 Kuboard

如果您已经有了 Kubernetes 集群，只需要一行命令即可安装 Kuboard：

```
kubectl apply -f https://kuboard.cn/install-script/kuboard.yaml
```

然后访问您集群中任意节点的 32567 端口（http://any-of-your-node-ip:32567） ，即可打开 Kuboard 界面，比如我的 Node 节点 IP 为：http://10.3.50.121:32567

### 获取 Token

此 Token 拥有 ClusterAdmin 的权限，可以执行所有操作

```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep kuboard-user | awk '{print $1}')

# 输出如下
Name:         kuboard-user-token-dsccx
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: kuboard-user
              kubernetes.io/service-account.uid: 08aad1b1-3d1b-4b6f-a379-3e320620a09f

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJvYXJkLXVzZXItdG9rZW4tZHNjY3giLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoia3Vib2FyZC11c2VyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMDhhYWQxYjEtM2QxYi00YjZmLWEzNzktM2UzMjA2MjBhMDlmIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmt1Ym9hcmQtdXNlciJ9.rndy-u-CxDF_yfaubq2HmIb7o04uLI3z7soqTf581VPLFo31dLmQvDPBJXWmFXHAfy0f-msCZjXWU35Cvwvlg3A5LZXgmfsJJi-kwYgpjnsOpFqquezAR6mDDnyXC1NLb8HD4PXgb4wbqnz6-jXlj7T0EXaDK0U5JvNtnkVy8y66mucz6tGnANqLYsHIYPT86P2VTOAWPrnz3pJHtoo_2AMI_y5Pc9RiVH1cro5I5D8Qf8_XDBE4oTsNXaqvWItnjg1O-nhxwsRpe-Szbe9wtFLbqjsO0_ZF5jPT1SdNrkuzoRJ4FpJsjBpbSpLB_DXtc0Y15oJVNZwpmJXKywCXRw
```

### 登录 Kuboard

使用刚才生成的 Token 直接登录即可

![](../img/12-00000024.png)

![](../img/12-00000025.png)

### 为什么使用 Kuboard

#### 非侵入式

Rancher 为了支持多云管理（MeSos，Docker Swarm，Kubernetes 等），引入了一系列复杂的概念，又需要新增安装 Rancher Server、Rancher Agent 等，本来学习 Kubernetes 已然不堪重负，Rancher 再把多云整合到一起来，无疑又增加了学习和使用的复杂性。如果您只是打算使用 Kubernetes 的话，也许并不需要选择 Rancher 这样重量级的产品。Kuboard 仅仅依赖于原生 Kubernetes，可以运行在各种公有云、私有云上，您也可以自己基于物理机、vsphere、vmware 等已经有的基础设施搭建 Kubernetes，轻松实现现有 Infrastructure 的容器化改造

#### 名称空间

在 Kuboard 中，名称空间的展示形式以微服务参考分层架构的形式，将所有的微服务分为如下几层：

- 展现层

    - 终端用户访问的 Web 应用

- API 网关层

    - Spring Cloud Gateway / Zuul / Kong 等接口网关

- 微服务层

    - Spring Boot 微服务，或 PHP / Python / Go 实现的微服务

- 持久层

    - MySQL 数据库等（开发及测试环境里，将 MySQL 部署于 Kubernetes 可以极大地降低环境维护的任务量）

- 中间件层

    - 消息队列
    
    - 服务注册 Eureka / Zookeeper / Consul 等

- 监控层

    - Prometheus + Grafana
    
    - Pinpooint 等

![](../img/12-00000026.png)

更为贴心的，当某一个工作负载存在部署方面的问题时， Kuboard 将以红色高亮出该工作负载，并配有合适的方式辅助用户定位错误信息

#### 无需手写 Yaml

学习 Kubernetes 时，需要花费许多的时间，在理解一个概念之后，模仿着写一个 **yaml** 文件，再使用 **kubectl** 应用该文件，使用 Kuboard 提供的工作负载编辑器，可以直观的完成应用的部署

![](../img/12-00000027.png)

#### 应用升级

Kuboard 提供批量修改容器镜像版本的功能，轻松升级应用程序的版本

![](../img/12-00000028.png)
