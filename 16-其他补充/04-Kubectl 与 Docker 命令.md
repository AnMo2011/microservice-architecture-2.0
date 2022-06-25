## 概述

Docker 命令和 Kubectl 命令有很多相似的地方，Docker 操作容器，Kubectl 操作 Pod（容器的集合）等

## 运行容器

- docker：`docker run -d --restart=always -e DOMAIN=cluster --name nginx-app -p 80:80 nginx`

- kubectl：

  - `kubectl run --image=nginx nginx-app --port=80 --env="DOMAIN=cluster"`

  - `kubectl expose deployment nginx-app --port=80 --name=nginx-http`

> **注意：** `kubectl run` 会创建一个 **Deployment** 并且默认会在后台运行，以上面的代码为例它的名称为 **nginx-app**。默认情况 Deployment 并不会将端口暴露出去，所以我们还需要使用 `kubectl expose` 暴露端口以供访问，此时还会创建一个同名的 **Service**

## 查看已运行的容器

- docker：`docker ps`

- kubectl：
 
    - `kubectl get pods`
    
    - `kubectl get deployment`
    
    - `kubectl get service`

## 交互式进入容器

- docker：`docker exec -it <容器 ID/NAME> /bin/bash`

- kubectl：`kubectl exec -it <容器名> -- /bin/bash`

## 打印日志

- docker：`docker logs -f <容器 ID/NAME>`

- kubectl：`kubectl logs -f <容器名>`

## 停止和删除容器

- docker：

  - `docker stop <容器 ID/NAME>`

  - `docker rm <容器 ID/NAME>`

- kubectl：

  - `kubectl delete deployment `

  - `kubectl delete service `

> **注意：** 不要直接删除 Pod，使用 kubectl 请删除拥有该 Pod 的 Deployment。如果直接删除 Pod，则 Deployment 将会重新创建该 Pod。

## 查看版本

- docker：`docker version`

- kubectl：`kubectl version`

## 查看环境信息

- docker：`docker info`

- kubectl：`kubectl cluster-info`