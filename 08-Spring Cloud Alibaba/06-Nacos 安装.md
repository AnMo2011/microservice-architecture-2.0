### 概述

Nacos 官方提供了 Docker 版本，方便我们快速部署，操作步骤如下

### 操作步骤

##### Clone 项目

```
git clone https://github.com/nacos-group/nacos-docker.git
cd nacos-docker
```

##### 单机模式

```
docker-compose -f example/standalone-mysql.yaml up -d
```

##### 查看日志

```
docker-compose -f example/standalone-mysql.yaml logs -f
```

##### Nacos 控制台

```
http://ip:8848/nacos
```

##### 登录

![](../img/08-00000003.png)

- 账号： nacos

- 密码： nacos

##### 访问

![](../img/08-00000004.png)
