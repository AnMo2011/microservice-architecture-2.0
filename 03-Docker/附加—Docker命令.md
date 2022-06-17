# 命令：

- Docker 下载：

```
docker pull xxx
```

- Docker 查询镜像：

```
docker images
```

- Docker 

```
docker system df
```

- Docker

```
docker -a
```

- Docker 查询正在运行的容器

```
docker ps

# 输入如下内容：
root@ubuntu:/# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
1cfec0d2b5c7        tomcat              "catalina.sh run"   20 hours ago        Up 20 hours         0.0.0.0:8080->8080/tcp   stoic_fermat
```

- Docker 查询容器

```
docker ps -a
```

- Docker 启动容器

```
docker staer 容器ID
```

- Docker 停止容器

> **注**：**==容器ID==** 查询就是使用 **==docker ps==** 查询出来的，ID不会重复。

```
docker stop 容器ID
```

- Docker 删除容器

> 注：删除容器可以根据 **==容器ID==** 或者 **==容器名==** 删除。

```
docker rm 容器ID/容器名
```

> 注：docker rm 容器ID 命令是没办法删除再运行的容器，如果想删除再运行的容器，需要一下命令！

```
docker rm -f 容器ID/容器名
```

- Docker 山粗所有停止的容器

```
docker container prune

# 输入如下：
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
```

- Docker 交互

```
docker exec -it 1cfec0d2b5c7 /bin/bash
```

- Tomcat 前台运行：

```
docker run -p 8080:8080 tomcat
```

- Tomcat 守护运行：

```
docker run -p 8080:8080 -d tomcat
```

> 运行Tomcat容器并给Tomcat容器起个名字，容器名不得重复！

```
docker run -p 8080:8080 --namn tomcat -d tomcat
```
















































