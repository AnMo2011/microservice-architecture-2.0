> **小提示：** 所有命令前都可以加上 `watch` 命令来观察状态的实时变化，如：`watch kubectl get pods --all-namespaces`

- 查看组件状态

```
kubectl get cs
```

- 查看环境信息

```
kubectl cluster-info
```

- 查看 Node

```
kubectl get nodes -o wide
```

- 查看集群配置

```
kubectl -n kube-system get cm kubeadm-config -oyaml
```

- 运行容器

```
kubectl run nginx --image=nginx --replicas=2 --port=80
```

- 暴露服务

```
kubectl expose deployment nginx --port=80 --type=LoadBalancer
```

- 查看命名空间

```
kubectl get ns
kubectl get namespace
```

- 创建命名空间

```
apiVersion: v1
kind: Namespace
metadata:
  name: development
```

- 查看容器

```
kubectl get pods -o wide
kubectl get deployment -o wide
```

- 查看服务

```
kubectl get svc -o wide
kubectl get service -o wide
```

- 查看详情，追踪事件，可用于定位问题

```
kubectl describe pod <Pod Name>
kubectl describe deployment <Deployment Name>
kubectl describe service <Service Name>
```

- 查看日志

```
kubectl logs -f <Pod Name>
```

- 删除容器和服务

```
kubectl delete deployment <Deployment Name>
kubectl delete service <Service Name>
```

- 配置方式运行

```
kubectl apply -f <YAML>
kubectl create -f <YAML>
```

- 配置方式删除

```
kubectl delete -f <YAML>
```

- 查看配置

```
kubeadm config view
kubectl config view
```

- 查看 Ingress

```
kubectl get ingress
```

- 查看持久卷

```
kubectl get pv
```

- 查看持久卷消费者

```
kubectl get pvc
```

- 查看 ConfigMap

```
kubectl get cm <ConfigMap Name>
```

- 修改 ConfigMap

```
kubectl edit cm <ConfigMap Name>
```

- 查看认证与授权

```
kubectl get secret [-n namespace]
```

- Port Forward 方式暴露服务到主机

```
kubectl port-forward svc/<Service Name> <Host Port>:<Service Port> --namespace=<Namespace> --address 0.0.0.0
```