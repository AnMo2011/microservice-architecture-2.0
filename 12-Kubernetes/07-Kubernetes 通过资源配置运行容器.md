### 概述

我们知道通过 **run** 命令启动容器非常麻烦，Docker 提供了 Compose 为我们解决了这个问题。那 Kubernetes 是如何解决这个问题的呢？其实很简单，使用 **kubectl** **create** 命令就可以做到和 Compose 一样的效果了，该命令可以通过配置文件快速创建一个集群资源对象。

# 创建 YAML 配置文件

### 部署 Deployment

创建一个名为 **nginx-deployment.yml** 的配置文件

```
# API 版本号
apiVersion: apps/v1
# 类型，如：Pod/ReplicationController/Deployment/Service/Ingress
kind: Deployment
# 元数据
metadata:
  # Kind 的名称
  name: nginx-app
spec:
  selector:
    matchLabels:
      app: nginx
  # 部署的实例数量
  replicas: 2
  template:
    metadata:
      labels:
        # 容器标签的名字，发布 Service 时，selector 需要和这里对应
        app: nginx
    spec:
      # 配置容器，数组类型，说明可以配置多个容器
      containers:
      # 容器名称
      - name: nginx
        # 容器镜像
        image: nginx:1.17
        # 只有镜像不存在时，才会进行镜像拉取
        imagePullPolicy: IfNotPresent
        # 暴露端口
        ports:
        # Pod 端口
        - containerPort: 80
```

```
# 部署
kubectl create -f nginx-deployment.yml

# 删除
kubectl delete -f nginx-deployment.yml
```

### 发布 Service

创建一个名为 **nginx-service.yml** 的配置文件

```
# API 版本号
apiVersion: v1
# 类型，如：Pod/ReplicationController/Deployment/Service/Ingress
kind: Service
# 元数据
metadata:
  # Kind 的名称
  name: nginx-http
spec:
  # 暴露端口
  ports:
    ## Service 暴露的端口
    - port: 80
      ## Pod 上的端口，这里是将 Service 暴露的端口转发到 Pod 端口上
      targetPort: 80
  # 类型
  type: LoadBalancer
  # 标签选择器
  selector:
    # 需要和上面部署的 Deployment 标签名对应
    app: nginx
```

```
# 部署
kubectl create -f nginx-service.yml
# 删除
kubectl delete -f nginx-service.yml
```

### 镜像拉取策略

支持三种 ImagePullPolicy

- **Always**： 不管镜像是否存在都会进行一次拉取

- **Never**： 不管镜像是否存在都不会进行拉取

- **IfNotPresent**： 只有镜像不存在时，才会进行镜像拉取

注意

- 默认为 **IfNotPresent**，但 **:latest** 标签的镜像默认为 **Always**

- 拉取镜像时 Docker 会进行校验，如果镜像中的 MD5 码没有变，则不会拉取镜像数据

- 生产环境中应该尽量避免使用 **:latest** 标签，而开发环境中可以借助 **:latest** 标签自动拉取最新的镜像

# 验证是否生效

### 查看 Pod 列表

```
kubectl get pods

# 输出如下
NAME                         READY   STATUS    RESTARTS   AGE
nginx-app-64bb598779-2pplx   1/1     Running   0          25m
nginx-app-64bb598779-824lc   1/1     Running   0          25m
```

### 查看 Deployment 列表

```
kubectl get deployment

# 输出如下
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
nginx-app   2/2     2            2           25m
```

### 查看 Service 列表

```
kubectl get service

# 输出如下
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP      10.96.0.1      <none>        443/TCP        20h
nginx-http    LoadBalancer   10.98.49.142   <pending>     80:31631/TCP   14m
```

### 查看 Service 详情

```
kubectl describe service nginx-app

# 输出如下
Name:                     nginx-http
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 name=nginx
Type:                     LoadBalancer
IP:                       10.98.49.142
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31631/TCP
Endpoints:                10.244.141.205:80,10.244.2.4:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

### 通过浏览器访问

通过浏览器访问 http://192.168.141.120:31631/ ，出现 Nginx 欢迎页即表示成功

### 集成环境部署

也可以不区分配置文件，一次性部署 Deployment 和 Service，创建一个名为 **nginx.yml** 的配置文件，配置内容如下：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-http
spec:
  ports:
    - port: 80
      targetPort: 80
      # 可以指定 NodePort 端口，默认范围是：30000-32767
      # nodePort: 30080
  type: LoadBalancer
  selector:
    app: nginx
```

```
# 部署
kubectl create -f nginx.yml

# 删除
kubectl delete -f nginx.yml
```

# 附：扩展阅读

### 修改默认的端口范围

Kubernetes 服务的 NodePort 默认端口范围是 30000-32767，在某些场合下，这个限制不太适用，我们可以自定义它的端口范围，操作步骤如下：

编辑 **vi /etc/kubernetes/manifests/kube-apiserver.yaml** 配置文件，增加配置 **--service-node-port-range=2-65535**

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    # 在这里增加配置即可
    - --service-node-port-range=2-65535
    - --advertise-address=192.168.141.150
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
// 以下配置省略...
```

使用 **docker ps** 命令找到 **kube-apiserver** 容器，再使用 docker restart **<ApiServer 容器 ID>** 即可生效。
