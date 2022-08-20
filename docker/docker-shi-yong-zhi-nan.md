# Docker 使用指南



## 1、Docker的安装

### Centos7安装并自启动

```
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io -y
systemctl enable docker
systemctl restart docker
```

### Ubuntu安装并自启动

```
sudo apt-get install ca-certificates curl gnupg lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

## 2、Docker的基础使用

### 搜索镜像

docker search `images`

### 查看镜像

docker images ls

### 拉取镜像

docker pull `images`

### 运行容器

```
docker run -it imagesname /bin/bash
```

```
-t	交互式操作
-t	终端
-d	指定在后台运行，带-d参数默认不进docker，需要使用exec命令
-p	将容器的端口映射到本地 前为宿主机端口 后为容器内部端口
--restart=always
    {
        docker run -d -p 9999:8080 imagesname python xxx.py
    }
imagesname	镜像名字
/bin/bash 交互式Shell
```

```
让docker在交互式容器后台运行：Ctrl+p 加上 Ctrl+q
进入后台运行的容器: 
1.docker attach container-id	不推荐使用，退出容器会导致容器停止运行
2.docker exec container-id	推荐使用，退出容器不会导致容器停止运行

example（exec）： docker exec -it container-id /bin/bash
```

### 查看容器

docker ps # 查看在运行的容器

docker ps -a # 查看所有的容器 包括停止的

### 进入容器

docker exec -it `iamges`

### 停止容器

docker stop `container`

## 删除镜像

docker rmi `container`

### 查看结束的容器并且终结

docker container ls -aq = docker container ls -a | awk '{print $1}' docker rm $(docker container ls -aq)

### 查看容器信息

docker inspect `container`

### 查看容器日志

docker logs `container`

### 上传容器镜像

```
docker login
输入账号密码回车即可
出现success就是成功了
```

```
docker push limitrinno/dockername:laster
就可以在docker hub上面到看到自己制作的镜像了
```

### 限制Docker的资源

* \--memory 内存加上swap的内存 一共可以用的是400M

docker run --memory=200M `images`

### 拷贝文件到docker

```
宿主机拷贝到docker里
docker ps /host/aaa.txt dockername:/dockerhost/aaa.txt
容器拷贝文件出来
docker ps dockername:/etc/bbb.txt /etc/bbb.txt
```

## 3、制作镜像

### 现有环境打包(不提倡)

将一个环境做好的容器打包commit

docker commit container-id limitrinno/centos-vim

### Docker build (推荐)

制作一个自己需要的镜像 首先创建一个目录 以创建vim的centos为例 mkdir centos-vim-build && cd centos-vim-build vim dockerfile

```
FROM centos
RUN yum -y install vim
```

```
docker build -t limitrinno/centos-vim .
```

## 4、Docker network

### 基本操作

### 创建网络

```
docker network create cumtom_network1

在创建的网络下边，可以使用docker内置的DNS server，就是通过主机名ping通
```

### 连接网络

```
docker network connect 网卡名字 容器名字
```

### 断开网络

```
docker network disconnect 网卡名字 容器名字
```

### 删除网络

```
docker network rm 网卡名字
```

### Bridge Network

给容器增加一个host解析

```
docker run -it --name=centos1 centos

docker run -it --name=centos2 --link=centos1 centos
这个时候 centos2的容器就会有一个centos1的hosts解析
```

### 查看容器网络列表

```
docker network list
```

### 新建网络

```
docker network create -d bridge limit
```

```
docker run -it --network limit --name=centos3
docekr inspect centos3
```

#### 旧容器添加新网络

```
docker network connect limit centos2
这个时候centos2就有limit这个网络的IP地址
```

> 自己新建的一个网络 比如limit，在连接新的容器和之前的容器的时候会自动创建一个link，也就是可以通过hosts来ping通主机

### Host

```
与宿主机共享IP地址
```

### None

```
只有宿主机可以exec到主机
```

## 5、Volume 存储

### Data Volume

```
docker run mysql:/var/lib/mysql
```

### Bind Mouting

```
docekr run /var/lib/mysql:/var/lib/mysql
```

### 创建Volume和查看

```
docker volume create volume-name

docker volume inspect volume-name
```

### 通过mount挂载内容到Docker

```
docekr run -itd --name name-container --mount src=volume src,dst=docker src containner-id
```

```
在创建的过程中，如果没有对应的Volume将会自动创建

[root@hkdocker]# docker volume create nginx-web
nginx-web
[root@hkdocker]# docker volume inspect nginx-web
[
    {
        "CreatedAt": "2020-02-08T17:08:53+08:00",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/nginx-web/_data",
        "Name": "nginx-web",
        "Options": {},
        "Scope": "local"
    }
]

[root@hkdocker]# docker volume ls
DRIVER              VOLUME NAME
local               nginx-web
```

### 文件夹挂载

```
docker run -it --name=nginx01 -p 80:80 -v /dataname/conf:/dockername/conf nginx
		    启动保留  容器名    宿主机端口：虚拟机端口  -v 宿主机目录:容器目录(无自动创建) image
```

#### 匿名挂载卷

```
docker run -it --name=nginx -v /usr/local/data nginx

文件就在 /var/lib/docker/volumes/***
```

#### 具名挂载卷

```
docker run -it --name=nginx -v docker_centos_data:/usr/local/data --name=nginx nginx
将 /usr/local/data目录挂载在docker_centos_data的卷下(/var/lib/docker/volumes/docker_centos_data)
```

#### 挂载卷的只读与读写

只读与读写都是针对与docker容器的目录设置的权限

```
docker run -it -v /主机路径:/docker的路径:ro imagename
这个就是挂载卷的只读

docker run -it -v /主机路径:/docker的路径:rw imagename
这个就是挂载卷的读写
```

#### 挂载卷继承

```
容器 centos7-01指定目录挂载
docker run --v /mydata/docker_centos/data:/usr/local/data --name centos7-01 centos: 7
容器 centos7-04和 centos7-05相当于继承 centos7-01音器白挂载目录
docker run -di --volumes-from centos7-01--name centos7-04 centos: 7
docker run -di --volumes-from centos7-01--name centos7-05 centos: 7
```

### 查看目录挂载关系

```
docker volume inspect volumename
```

## 6、Docker-compose

### Docker-compose 的安装

```
sudo curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```

### 启动Docker compose

```
sudo chmod +x /usr/local/bin/docker-compose
$ docker-compose --version
```

## 7、Dockerfile 语法

### FROM

* 尽量使用官方的image做的基本的镜像

```
FROM centos
FROM ubuntu
```

### LABEL

* 定义docker的作者、邮箱以及描述

```
LABEL maintainer="limitrinno@gmail.com"
LABEL version="1.0"
LABEL description="This is description"
```

### RUN

* 每run一次就会有一个新的层
* 所以运行run的时候保持美观并且代码过长使用 \ 来分行

```
RUN yum -y update && yum -y install vim \
    gcc
```

### WORKDIR

* 设置工作目录
* 不要用RUN cd,使用绝对目录

```
WORKDIR /root
```

```
WORKDIR /test  # 如果/下没有test就会自动创建一个test
WORKDIR demo   # 相当于cd /test/demo
```

### ADD 和 COPY

* ADD 不仅仅只有添加文件的功能还有解压的功能

```
ADD hello / # 将可执行文件hello放到根目录/底下
ADD test.tar.gz / # 将一个压缩包解压到根目录底下
```

```
WORKDIR /root
ADD hello test/  # 即文件工作在/root/test/hello
```

```
WORKDIR /root
COPY hello test/
```

```
添加远程软件的话使用curl或者wget
```

### ENV

* 设置变量或者常量

```
ENV MYSQL_VERSION 5.6
RUN apt-get install -y mysql-server="${MYSQL_VERSION}"
```

```
ENV <key>=<value>
配置环境变量 ENV JAVA_HOME /usr/local/java/jdk-11.0.6
```

### VOLUME 和 EXPOSE

```
VOLUME ["/var/lib/mysql"]
```

```
EXPOSE <port> [<port>/<protocol>...]

默认情况下，EXPOSE假定使用TCP。 您还可以指定UDP：
EXPOSE 80/udp

要同时在TCP和UDP上公开，请包括以下两行：
EXPOSE 80/tcp
EXPOSE 80/udp

EXPOSE 80 443 8080/tcp
```

### CMD 和 ENTRYPOINT

```
该CMD指令具有三种形式：

CMD ["executable","param1","param2"] （*exec*形式，这是首选形式）
CMD ["param1","param2"] （作为*ENTRYPOINT的默认参数*）
CMD command param1 param2 （Shell形式）

CMD指令中只能有一条指令Dockerfile。如果您列出多个，CMD 则只有最后一个CMD才会生效。
```

* 小案例

```
FROM ubuntu
RUN apt-get update && apt-get install stress
ENTRYPOINT ["/usr/bin/stress"]
CMD []

使用entrypoint定义使用的参数 cmd负责接收参数
```

## 10、遇到的问题 小Tips

### ipv4报错 转发问题

WARNING: IPv4 forwarding is disabled. Networking will not work.

* Centos 7 开启IPV4/6的转发

echo "net.ipv4.ip\_forward = 1" >>/usr/lib/sysctl.d/00-system.conf

echo "net.ipv6.ip\_forward = 1" >>/usr/lib/sysctl.d/00-system.conf

* 重启服务

```
systemctl restart network && systemctl restart docker
```

### 阿里云Docker加速源

* 基本说明

> 安装／升级Docker客户端-推荐安装1.10.0以上版本的Docker客户端，参考文档 [docker-ce](https://yq.aliyun.com/articles/110806)

> 配置镜像加速器-针对Docker客户端版本大于 1.10.0 的用户
>
> > 您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

```
1、 国内镜像源加速站点

https://registry.docker-cn.com

http://hub-mirror.c.163.com

https://3laho3y3.mirror.aliyuncs.com

http://f1361db2.m.daocloud.io

https://mirror.ccs.tencentyun.com
```

* Ubuntu

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://zj70ve2y.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

* Centos

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://zj70ve2y.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
