### 检查组件运行状态

```
kubectl get cs

# 输出如下
NAME                 STATUS    MESSAGE             ERROR
# 调度服务，主要作用是将 POD 调度到 Node
scheduler            Healthy   ok
# 自动化修复服务，主要作用是 Node 宕机后自动修复 Node 回到正常的工作状态
controller-manager   Healthy   ok
# 服务注册与发现
etcd-0               Healthy   {"health":"true"}  
```

### 检查 Master 状态

```
kubectl cluster-info

# 输出如下
Kubernetes master is running at https://192.168.141.110:6443
KubeDNS is running at https://192.168.141.110:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

### 检查 Nodes 状态

```
kubectl get nodes

# 输出如下
NAME                 STATUS   ROLES    AGE   VERSION
kubernetes-master    Ready    master   72m   v1.15.0
kubernetes-node-01   Ready    <none>   52m   v1.15.0
kubernetes-node-02   Ready    <none>   51m   v1.15.0
```

### 运行第一个容器实例

```
# 使用 kubectl 命令创建两个监听 80 端口的 Nginx Pod（Kubernetes 运行容器的最小单元）
kubectl run nginx --image=nginx --replicas=2 --port=80

# 输出如下
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/nginx created
```

### 查看全部 Pods 的状态

```
kubectl get pods

# 输出如下
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7c45b84548-mv8n8   1/1     Running   0          36s
nginx-7c45b84548-vp2x6   1/1     Running   0          36s
```

### 查看已部署的服务

```
kubectl get deployment

# 输出如下
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   2/2     2            2           70s
```

### 发布服务

```
# 使用负载均衡模式发布服务，让用户可以访问
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# 输出如下
service/nginx exposed
```

### 查看已发布的服务

```
kubectl get services

# 输出如下
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP        81m
nginx        LoadBalancer   10.102.37.240   <pending>     80:31406/TCP   115s
```

### 查看服务详情

```
kubectl describe service nginx

# 输出如下
Name:                     nginx
Namespace:                default
Labels:                   run=nginx
Annotations:              <none>
Selector:                 run=nginx
Type:                     LoadBalancer
IP:                       10.102.37.240
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
# 发布到外网的服务端口
NodePort:                 <unset>  31406/TCP
Endpoints:                192.168.140.66:80,192.168.141.194:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

### 验证是否成功

通过浏览器访问 Node 服务器，此时 Kubernetes 会以负载均衡的方式访问部署的 Nginx 服务，能够正常看到 Nginx 的欢迎页即表示成功

    http://192.168.141.120:31406/

### 停止服务

- 删除已部署的服务

```
kubectl delete deployment nginx

# 输出如下
deployment.extensions "nginx" deleted
```

- 删除已发布的服务

```
kubectl delete service nginx

# 输出如下
service "nginx" deleted
```
