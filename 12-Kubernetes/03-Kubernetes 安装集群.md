### 概述

kubeadm 是 kubernetes 的集群安装工具，能够快速安装 kubernetes 集群，安装 kubernetes 主要是安装它的各个镜像，而 kubeadm 已经为我们集成好了运行 kubernetes 所需的基本镜像。但由于国内的网络原因，在搭建环境时，无法拉取到这些镜像。此时我们只需要修改为阿里云提供的镜像服务即可解决该问题。

### 创建并修改配置

```
# 导出配置文件
kubeadm config print init-defaults --kubeconfig ClusterConfiguration > kubeadm.yml
```

```
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  # 修改为主节点 IP
  advertiseAddress: 192.168.141.110
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: kubernetes-master
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
# 国内不能访问 Google，修改为阿里云
imageRepository: registry.aliyuncs.com/google_containers
kind: ClusterConfiguration
# 修改版本号
kubernetesVersion: v1.15.0
networking:
  dnsDomain: cluster.local
  # 配置 POD 所在网段为我们虚拟机不重叠的网段（这里用的是 Flannel 默认网段）
  podSubnet: "10.244.0.0/16"
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

### 查看所需镜像

```
kubeadm config images list --config kubeadm.yml

# 输出如下
registry.aliyuncs.com/google_containers/kube-apiserver:v1.15.0
registry.aliyuncs.com/google_containers/kube-controller-manager:v1.15.0
registry.aliyuncs.com/google_containers/kube-scheduler:v1.15.0
registry.aliyuncs.com/google_containers/kube-proxy:v1.15.0
registry.aliyuncs.com/google_containers/pause:3.1
registry.aliyuncs.com/google_containers/etcd:3.3.10
registry.aliyuncs.com/google_containers/coredns:1.3.1
```

### 拉取所需镜像

```
kubeadm config images pull --config kubeadm.yml

# 输出如下
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-apiserver:v1.15.0
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-controller-manager:v1.15.0
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-scheduler:v1.15.0
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-proxy:v1.15.0
[config/images] Pulled registry.aliyuncs.com/google_containers/pause:3.1
[config/images] Pulled registry.aliyuncs.com/google_containers/etcd:3.3.10
[config/images] Pulled registry.aliyuncs.com/google_containers/coredns:1.3.1
```

### 安装主节点

执行以下命令初始化主节点，该命令指定了初始化时需要使用的配置文件，其中添加 **--upload-certs** 参数可以在后续执行加入节点时自动分发证书文件。追加的 **tee kubeadm-init.log** 用以输出日志。

> 注意： 如果安装 kubernetes 版本和下载的镜像版本不统一则会出现 timed out waiting for the condition 错误。中途失败或是想修改配置可以使用 kubeadm reset 命令重置配置，再做初始化操作即可。

```
kubeadm init --config=kubeadm.yml --upload-certs | tee kubeadm-init.log

# 输出如下
Flag --experimental-upload-certs has been deprecated, use --upload-certs instead
[init] Using Kubernetes version: v1.15.0
[preflight] Running pre-flight checks
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Activating the kubelet service
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.141.110]
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [kubernetes-master localhost] and IPs [192.168.141.110 127.0.0.1 ::1]
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [kubernetes-master localhost] and IPs [192.168.141.110 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 23.006217 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.15" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
4a351218cf34c15b04b08b7d916c27a625418145d8d3b281ec125fd2b2ac7ab5
[mark-control-plane] Marking the node kubernetes-master as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node kubernetes-master as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: abcdef.0123456789abcdef
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.141.110:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:f0759e0d352c1a5de4444782b4a676460b2ea7a2876fa0accab572b8629b72c8 
```

### 配置 kubectl

```
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

# 非 ROOT 用户执行
chown $(id -u):$(id -g) $HOME/.kube/config
```

### 验证是否成功

```
kubectl get node

# 输出如下
NAME                STATUS     ROLES    AGE     VERSION
kubernetes-master   NotReady   master   4m38s   v1.15.0
```

### 安装从节点

将 Node 节点加入到集群中很简单，只需要在 Node 服务器上安装 **kubeadm**，**kubectl**，**kubelet** 三个工具，然后使用 ==kubeadm join== 命令加入即可

```
kubeadm join 192.168.141.110:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:f0759e0d352c1a5de4444782b4a676460b2ea7a2876fa0accab572b8629b72c8

# 输出如下
[preflight] Running pre-flight checks
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.15" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Activating the kubelet service
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

### 验证是否成功

回到 Master 主节点查看是否安装成功

> **注意**： 如果 Node 节点加入 Master 时配置有问题可以在 Node 节点上使用 **kubeadm reset** 重置配置再使用 **kubeadm join** 命令重新加入即可。希望在 master 节点删除 node ，可以使用 **kubectl delete nodes <NAME>** 删除。

```
kubectl get node

# 输出如下
NAME                 STATUS     ROLES    AGE   VERSION
kubernetes-master    NotReady   master   20m   v1.15.0
kubernetes-node-01   NotReady   <none>   16s   v1.15.0
kubernetes-node-02   NotReady   <none>   6s    v1.15.0
```

### 查看 Pod 状态

coredns 尚未运行，此时我们还需要安装网络插件

```
kubectl get pod -n kube-system -o wide

# 输出如下
NAME                                        READY   STATUS    RESTARTS   AGE     IP                NODE                 NOMINATED NODE   READINESS GATES
coredns-bccdc95cf-9s4bm                     0/1     Pending   0          24m     <none>            <none>               <none>           <none>
>
coredns-bccdc95cf-s8ggd                     0/1     Pending   0          24m     <none>            <none>               <none>           <none>
>
etcd-kubernetes-master                      1/1     Running   0          24m     192.168.141.110   kubernetes-master    <none>           <none>
>
kube-apiserver-kubernetes-master            1/1     Running   0          24m     192.168.141.110   kubernetes-master    <none>           <none>
>
kube-controller-manager-kubernetes-master   1/1     Running   0          23m     192.168.141.110   kubernetes-master    <none>           <none>
>
kube-proxy-8s87d                            1/1     Running   0          4m56s   192.168.141.120   kubernetes-node-01   <none>           <none>
>
kube-proxy-cbnlb                            1/1     Running   0          4m46s   192.168.141.121   kubernetes-node-02   <none>           <none>
>
kube-proxy-vwhxj                            1/1     Running   0          24m     192.168.141.110   kubernetes-master    <none>           <none>
>
kube-scheduler-kubernetes-master            1/1     Running   0          24m     192.168.141.110   kubernetes-master    <none>           <none>
>
```

### kubeadm init 的执行过程

- **init**： 指定版本进行初始化操作

- **preflight**： 初始化前的检查和下载所需要的 Docker 镜像文件

- **kubelet-start**： 生成 kubelet 的配置文件 **var/lib/kubelet/config.yaml**，没有这个文件 kubelet 无法启动，所以初始化之前的 kubelet 实际上启动不会成功

- **certificates**： 生成 Kubernetes 使用的证书，存放在 **/etc/kubernetes/pki** 目录中

- **kubeconfig**： 生成 KubeConfig 文件，存放在 **/etc/kubernetes** 目录中，组件之间通信需要使用对应文件

- **control-plane**： 使用 **/etc/kubernetes/manifest** 目录下的 YAML 文件，安装 Master 组件

- **etcd**： 使用 **/etc/kubernetes/manifest/etcd.yaml** 安装 Etcd 服务

- **wait-control-plane**： 等待 control-plan 部署的 Master 组件启动

- **apiclient**： 检查 Master 组件服务状态。

- **uploadconfig**： 更新配置

- **kubelet**： 使用 configMap 配置 kubelet

- **patchnode**： 更新 CNI 信息到 Node 上，通过注释的方式记录

- **mark-control-plane**： 为当前节点打标签，打了角色 Master，和不可调度标签，这样默认就不会使用 Master 节点来运行 Pod

- **bootstrap-token**： 生成 token 记录下来，后边使用 **kubeadm join** 往集群中添加节点时会用到

- **addons**： 安装附加组件 CoreDNS 和 kube-proxy
