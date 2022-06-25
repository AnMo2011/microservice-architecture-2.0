### 什么是 Nginx

Nginx 是一款高性能的 HTTP 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器。由俄罗斯的程序设计师 Igor Sysoev 所开发，官方测试 Nginx 能够支支撑 5 万并发链接，并且 CPU、内存等资源消耗却非常低，运行非常稳定。

### Nginx 的应用场景

- HTTP 服务器：Nginx 是一个 HTTP 服务可以独立提供 HTTP 服务。可以做网页静态服务器。

- 虚拟主机：可以实现在一台服务器虚拟出多个网站。例如个人网站使用的虚拟主机。

- 反向代理，负载均衡：当网站的访问量达到一定程度后，单台服务器不能满足用户的请求时，需要用多台服务器集群可以使用 Nginx 做反向代理。并且多台服务器可以平均分担负载，不会因为某台服务器负载高宕机而某台服务器闲置的情况。

# Nginx 虚拟主机

### 什么是虚拟主机

虚拟主机是一种特殊的软硬件技术，它可以将网络上的每一台计算机分成多个虚拟主机，每个虚拟主机可以独立对外提供 www 服务，这样就可以实现一台主机对外提供多个 web 服务，每个虚拟主机之间是独立的，互不影响的。

通过 Nginx 可以实现虚拟主机的配置，Nginx 支持三种类型的虚拟主机配置

- 基于 IP 的虚拟主机

- 基于域名的虚拟主机

- 基于端口的虚拟主机

### Nginx 配置文件的结构 

```
# ...
events {
    # ...
}

http {
    # ...
    server{
        # ...
    }

    # ...
    server{
        # ...
    }
}
```

> **注意**： 每个 server 就是一个虚拟主机

### 基于 Docker 部署 Nginx

```
version: '3.1'
services:
  nginx:
    restart: always
    image: nginx
    container_name: nginx
    ports:
      - 80:80
    volumes:
      - ./conf/nginx.conf:/etc/nginx/nginx.conf
      - ./html:/usr/share/nginx/html
```

### 基于端口的虚拟主机配置

#### 需求

- Nginx 对外提供 80 和 8080 两个端口监听服务

- 请求 80 端口则请求 html80 目录下的 html

- 请求 8080 端口则请求 html8080 目录下的 html

#### 操作流程

- 创建目录及文件，在 **/usr/local/docker/nginx/html** 目录下创建 **html80** 和 **html8080** 两个目录，并分别创建两个 **index.html** 文件

- 配置虚拟主机，创建并修改 **/usr/local/docker/nginx/conf** 目录下的 **nginx.conf**

```
# 启动进程,通常设置成和 CPU 的数量相等
worker_processes  1;

events {
    # epoll 是多路复用 IO(I/O Multiplexing) 中的一种方式
    # 但是仅用于 linux2.6 以上内核,可以大大提高 nginx 的性能
    use epoll;
    # 单个后台 worker process 进程的最大并发链接数
    worker_connections  1024;
}

http {
    # 设定 mime 类型,类型由 mime.type 文件定义
    include       mime.types;
    default_type  application/octet-stream;

    # sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用，
    # 必须设为 on，如果用来进行下载等应用磁盘 IO 重负载应用，可设置为 off，以平衡磁盘与网络 I/O 处理速度，降低系统的 uptime.
    sendfile        on;

    # 连接超时时间
    keepalive_timeout  65;
    # 设定请求缓冲
    client_header_buffer_size 2k;

    # 配置虚拟主机 192.168.141.121
    server {
    # 监听的 IP 和端口，配置 192.168.141.121:80
        listen       80;
    # 虚拟主机名称这里配置 IP 地址
        server_name  192.168.141.121;
    # 所有的请求都以 / 开始，所有的请求都可以匹配此 location
        location / {
        # 使用 root 指令指定虚拟主机目录即网页存放目录
        # 比如访问 http://ip/index.html 将找到 /usr/local/docker/nginx/html/html80/index.html
        # 比如访问 http://ip/item/index.html 将找到 /usr/local/docker/nginx/html/html80/item/index.html

            root   /usr/share/nginx/html/html80;
        # 指定欢迎页面，按从左到右顺序查找
            index  index.html index.htm;
        }

    }
    # 配置虚拟主机 192.168.141.121
    server {
        listen       8080;
        server_name  192.168.141.121;
        location / {
            root   /usr/share/nginx/html/html8080;
            index  index.html index.htm;
        }
    }
}
```

### 基于域名的虚拟主机配置

#### 需求

- 两个域名指向同一台 Nginx 服务器，用户访问不同的域名显示不同的网页内容

- 两个域名是 ingress.funtl.com 和 service.funtl.com

- Nginx 服务器使用虚拟机 192.168.141.121

#### 操作流程

- 配置 Hosts，通过 host 文件指定 service1.funtl.com 和 service2.funtl.com 对应 192.168.141.121 虚拟机，修改 window 的 hosts 文件：（C:\Windows\System32\drivers\etc）

- 创建目录及文件，在 **/usr/local/docker/nginx/html** 目录下创建 **service1** 和 **service1** 两个目录，并分别创建两个 index.html 文件

- 配置虚拟主机，修改 **/usr/local/docker/nginx/conf** 目录下的 **nginx.conf**

```
user  nginx;
worker_processes  1;

events {
    use epoll;
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    
    sendfile        on;
    
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  service1.funtl.com;
        location / {
            root   /usr/share/nginx/html/service1;
            index  index.html index.htm;
        }
        
    }

    server {
        listen       80;
        server_name  service2.funtl.com;
        
        location / {
            root   /usr/share/nginx/html/service2;
            index  index.html index.htm;
        }
    }
}
```
