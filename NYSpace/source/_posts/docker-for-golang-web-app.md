title: Docker for Golang Web App
date: 2017/10/03
comments: true
tags:
- Docker 
- Golang
categories: 
- Dokcer
---

# 前言
我最近在做自己的个人项目，需要部署一个稳定的 Golang Web App 运行环境。然而之前使用的 supervisor 十分不稳定，导致服务经常跪掉。所以决定学习 Docker，用其来做 Golang 的环境部署工具。另外，学习新东西很有意思，借此机会也学习一下运维和后端的相关技术。

本文分为 Docker 基本概念和 Docker 实践操作两部分，如果只想学习 Docker 的基本操作，可点击 [Docker 实践](./#Docker-实践)

# 初识 Docker
Docker 是一个为开发、装载以及运行应用提供的开放平台。Docker 使能将应用从基础设施中分离，从而使我们能快速发布软件。通过利用 Docker 装载、测试以及快速部署代码的优势，在生产中我们能够显著地减少编写代码和运行程序间的时间延迟。

## Docker Engine

Docker 引擎是一个拥有以下组件的 C/S 应用：
- 服务端，一种被称谓后台进程 daemon process 的始终运行的程序（dockerd 命令）。
- REST API，那些程序能够用于和后台程序通信和指示它们做什么的特定接口。
- 命令行（CLI）客户端（docker 命令）。

![](https://docs.docker.com/engine/article-img/engine-components-flow.png)

CLI 用 Docker REST API 去经由脚本或者直接命令行指令，与 Docker 后台程序进行控制或者交互。很多其他 Docker 应用使用下层的 API 和 CLI。

后台程序（dockerd）创建和管理 Docker 对象，如镜像、容器、网络和卷。

## Docker architecture

Docker 是 C/S 架构的。Docker 客户端与 Docker 后台程序进行通讯，Docker 后台程序承担了 Docker 容器的创建、运行以及分发。Docker 客户端以及后台程序能够在相同系统上运行，或者我们可以用 Docker 客户端连接远程 Docker 后台。Docker 客户端和后台程序利用 REST API 在 UNIX sockets 或网络接口中进行通讯。

![](https://docs.docker.com/engine/article-img/architecture.svg)

### Docker daemon

Docker 后台程序（dockerd） 监听 Docker API 请求和管理 Docker 对象，例如，镜像、容器、网络和卷。一个后台程序也可以和其他后台程序进行通信，以管理 Docker 的服务。

### Docker client
Docker 客户端（docker）是 Docker 用户和 Docker 进行交互的主要方式。当我们使用命令行例如，**docker run** ，客户端向 dockerd 发送这些命令。docker 命令使用 Docker API。Docker 客户端能与不止一个后台程序进行通信。

### Docker registries
Docker 仓库存储 Docker 镜像。Docker Hub 以及 Docker 云是公用注册仓库，任何人都可以使用，并且 Docker 默认配置在 Docker Hub 查找镜像。我们甚至能共运行自己的私有仓库。

### Docker objects
当使用 Docker 时，我们正创建和使用使用镜像、容器、网络、卷、插件以及其他对象。这一部分是这些对象的简述。

#### IMAGES
**镜像是一个用指令来创建 Docker 容器的只读模板。**通常，一个镜像基于另外一个镜像，并有一些额外的自定义配置。例如，我们可能构建一个基于 ubuntu 的镜像，但是安装了 Apache Web 服务器和我们的应用，以及使应用程序运行所需的配置详细信息。

#### CONTAINERS
**容器是一个可运行的镜像的实例**。我们能够利用 Docker API 或者 CLI 创建、运行、停止、移动或者删除一个容器。

#### SERVICES

服务允许我们跨多个 Docker 守护程序扩展容器，这些守护进程与多个管理员和工作人员作为一个集群进行协作。 集群的每个成员都是 Docker 守护进程，守护进程使用 Docker API 进行通信。

# Docker 实践
以上是 Docker 的一些基本概念，但是这些概念都非常抽象，难以理解，所以，接下来具体实践下 Docker 是如何使用的。

## 安装 Docker
### Docker for macOS
对于 macOS 的 Docker 安装，是比较简单的，直接[下载 Docker](https://store.docker.com/editions/community/docker-ce-desktop-mac) 的客户端就可以。安装完成后，运行 Docker 就可以使用了。

### 阿里云 CentOS

输入下列命令行进行安装
```sh
yum install epel-release-y
yum clean all
yum list
yum install docker-io -y
docker info
```

或者运行

```sh
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
```

## 	镜像
在我使用 Docker 的过程中，接触到的镜像操作一般是列出镜像、获取镜像和删除镜像。

### 列出镜像 **docker images**
输入 **docker images** 命令后，控制台就输出本地所有的 Docker 镜像。

```docker
➜  Downloads docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
beego               latest              ac00984e1c84        45 hours ago        775MB
golang              latest              bba10fd6d576        6 days ago          733MB
mysql               latest              b4e78b89bcf3        3 weeks ago         412MB
nginx               latest              da5939581ac8        3 weeks ago         108MB

```


### 获取镜像
获取镜像有两种方式，一种是在 Docker Hub 等仓库拉取镜像，另一种是自己编写 Dockerfile 构建镜像。

#### Docker Hub 获取 **docker pull**
alpine 是 Linux 的体积最小的 Docker 镜像，用它来做演示。

```docker
docker pull alpine
```

**docker pull** 命令行会将 alpine 的镜像拉取到本地，再输入列出镜像命令，即可发现本地镜像中多了一个 alpine 的镜像。

```
➜  Downloads docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
beego               latest              ac00984e1c84        45 hours ago        775MB
golang              latest              bba10fd6d576        6 days ago          733MB
mysql               latest              b4e78b89bcf3        3 weeks ago         412MB
nginx               latest              da5939581ac8        3 weeks ago         108MB
alpine              latest              76da55c8019d        3 weeks ago         3.97MB
```

#### Dockerfile 构建 **docker build**
除了通过 Docker Hub 拉取镜像，还可以自己编写 Dockerfile 定制镜像。

```docker
FROM alpine
ADD helloworld /

```

然后在 Dockerfile 存储的位置构建自定义的 Docker 镜像。其中，本文所使用的项目是 [Simple Go Web App](https://github.com/niyaoyao/http-response-for-go-web-application) ，需要自己手动在 git 根目录创建 Dockerfile。

```docker
➜  http-response-for-go-web-application git:(master) ✗ docker build -t nyalpine .
Sending build context to Docker daemon  67.39MB
Step 1/2 : FROM alpine
 ---> 76da55c8019d
Step 2/2 : ADD helloworld /
 ---> 6cceb96a3904
Successfully built 6cceb96a3904
Successfully tagged nyalpine:latest
```

**docker build** 命令将我们所编写的 Dockerfile 构建到本定镜像，并按照对应指令，逐步运行。可以看到 **FROM** 指令先从 Docker Hub 拉取 alpine 镜像，然后 **ADD** 指令将项目中的 helloworld 二进制文件添加到根目录。

```docker
➜  http-response-for-go-web-application git:(master) ✗ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nyalpine            latest              6cceb96a3904        9 minutes ago       8.2MB
beego               latest              ac00984e1c84        46 hours ago        775MB
golang              latest              bba10fd6d576        6 days ago          733MB
mysql               latest              b4e78b89bcf3        3 weeks ago         412MB
nginx               latest              da5939581ac8        3 weeks ago         108MB
alpine              latest              76da55c8019d        3 weeks ago         3.97MB
```

输入 **docker images** 命令后，发现我们编写的 Docker 镜像构建成功。 运行一下测试这个 Docker 镜像是否可用。

```docker
➜  http-response-for-go-web-application git:(master) ✗ docker run -it --rm nyalpine
/ # ls
bin         etc         home        media       proc        run         srv         tmp         var
dev         helloworld  lib         mnt         root        sbin        sys         usr
/ # exit
```


**docker run** 是将我们构建的镜像运行成容器实例，其中 **-it** （interact）选项是生成可交互的终端，**--rm** 选项是当容器停止运行后，让 Docker 自动删除容器。

### 删除镜像 **docker rmi**

删除镜像的命令行是 **docker rmi**，即 remove image。例如，我们刚才创建的 nyalpine 这个镜像，我们可以直接用 nyalpine 名字 **docker rmi nyalpine** 删除，或者利用镜像对应的 IMAGE ID **docker rmi 6cce** 删除。

```docker
➜  http-response-for-go-web-application git:(master) ✗ docker rmi nyalpine
Untagged: nyalpine:latest
Deleted: sha256:6cceb96a3904a6b4e3aa50feb2665c9816b255e6418435326f8dc1b0ff0e4694
Deleted: sha256:23aedac71bd6a5bc86750590191aa0086319e3162c00be3d04b7ab8504f05f18
```

## 容器

刚才我们已经简单尝试过镜像的构建，并运行容器了。接下来，我们利用 [Simple Go Web App](https://github.com/niyaoyao/http-response-for-go-web-application) 项目来对容器的操作进行学习。

```docker
➜  http-response-for-go-web-application git:(master) docker run --rm -it -p 1234:9090 -v "$(pwd)":/go/src golang
root@2a9432e4806f:/go# cd src
root@2a9432e4806f:/go/src# sh run-go.sh
Service Started

```

在 git 根目录执行上面的命令，则可以启动 Go Web 服务，访问 http://localhost:1234/ 即可看到运行的 Web 网页。接下来就具体看一下操作容器经常会用到的命令。

### 列出容器 **docker ps** 

```sh
docker ps
```

**docker ps** 会列出所有运行中的容器，而 **docker ps -a** 会将已经停止但未被删除的容器一起列出。


### 运行容器 **docker run**
运行容器其实只需要 **docker run** 命令即可，但是，为了满足我们更多样的需求，我们需要使用很多选项参数进行设置，比如，设置网络端口以便宿主机能够访问等。 **docker run --help** 查看 **docker run** 命令的详细选项。

```docker
docker run --help
Usage:	docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
Run a command in a new containe
```

#### 交互终端
**-i** **--interactive** 选项使容器的标准输入始终打开，**-t** **--tty** 选项可以为容器创建一个 伪 TTY 终端。以 alpine 为例，我们运行一个有交互终端的容器输入下列命令。

```docker
docker run -it alpine
```

#### 指定端口
**-p** **--publish** 选项用来指定对宿主机开放的端口。比如，[Simple Go Web App](https://github.com/niyaoyao/http-response-for-go-web-application) 项目 helloworld.go 在容器中绑定的端口是 9090，而我们可以指定我们想要的主机访问的端口为 1234。

#### 随机端口
**-P** **--publish-all** 选项用来为主机开放所有暴露的端口，并且都为随机端口。

#### 绑定卷
**-v** **--volume** 选项用来绑定容器和宿主机的卷，比如， [Simple Go Web App](https://github.com/niyaoyao/http-response-for-go-web-application) 项目就将根目录绑定到 /go/src 目录下，以便编译运行 go 程序。

#### 停止容器后自动删除

#### 后台常驻
**-d** **--detach** 选项用来让容器在后台运行，并且打印容器的 ID。如下，就是让 MySQL 作为后台常驻容器运行，并且 MySQL 的密码应该填写在 MYSQL_ROOT_PASSWORD 后。

```docker
docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=mysecretpassword -d mysql
```

#### 容器名字
**--name** 选项可以指派容器一个易记的名字，如果这个选项没有被指定，那么 Docker 会给容器取一个随机的名字。

#### 容器链接
镜像的设计原则是使运行应用的容器尽量小且独立，那么可以运行多个容器以达到多个应用相互独立和隔离的目的，但是如果容器间想要进行通讯链接，就必须使用这个选项进行容器的互联。

比如，我们让 MySQL 作为后台常驻程序运行，如果需要修改 MySQL 的数据，其中一个方法就是可以通过运行另外一个容器，并使用 **--link** 选项链接到后台运行的 MySQL 容器。

```docker
docker run -it --link some-mysql:mysql --rm mysql sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD"'

```


### 停止容器 **docker stop**
```docker
➜  mvoice git:(master) ✗ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
00da63ecff60        golang              "bash"              About an hour ago   Up About an hour    0.0.0.0:1234->9090/tcp   jovial_franklin
➜  mvoice git:(master) ✗ docker stop 00da
00da

```

我们可以使用 **docker stop _container\_id_** 来终止运行的容器。对于我们刚才的例子，由于设置了 **--rm** 选项，那么容器再停止退出的时候会被自动删除。

### 删除容器 **docker rm**
如果在运行容器时，没有设置 **--rm** 选项，那么就必须对容器进行手动删除。**注意，--rm 选项 和 -d 选项不能同时使用。**

# 部署 MySQL + Golang + Nginx + Beego 的联合环境
## 官方镜像

执行 ```docker pull``` 命令，从官方下载最新镜像

```docker
➜  mvoice git:(master) ✗ docker pull golang        
➜  mvoice git:(master) ✗ docker pull mysql 
➜  mvoice git:(master) ✗ docker pull nginx
```

## 定制 Beego 镜像

### 编写 Dockerfile 文件

```docker
FROM golang:latest
MAINTAINER N.Y. <nycode.jn@gmail.com>

RUN go get -u github.com/go-sql-driver/mysql
RUN go get -u github.com/astaxie/beego
RUN go get -u github.com/beego/bee

EXPOSE 8080
```

### 编译定制的镜像

```docker
docker build -t beego .
```

## 运行 MySQL

### 运行容器

```docker
docker run --name nyspace-mysql -e MYSQL_ROOT_PASSWORD=root -d mysql
```

### 链接 MySQL

```docker
docker run -it --link nyspace-mysql:mysql --rm mysql sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD"'
```

### 初始化数据库和数据表

```sql
show databases;
create database mvoice;
use mvoice;
create table mvoice_user(
    user_id INT NOT NULL AUTO_INCREMENT,
    user_nick VARCHAR(50) NOT NULL,
    user_udid VARCHAR(500) NOT NULL default "",
    user_password VARCHAR(500) NOT NULL,
    user_created TIMESTAMP NOT NULL default CURRENT_TIMESTAMP,
    user_ip VARCHAR(20) NOT NULL default "",
    user_mobile VARCHAR(100) NOT NULL default "",
    PRIMARY KEY (user_id)
    );

insert into mvoice_user(user_nick, user_password) values("niyao", "25d55ad283aa400af464c76d713c07ad");

select * from mvoice_user;
```


## 运行 Beego API

### 生成 Beego Api 应用

```docker
docker run --rm --link db:mysql -v "$(pwd)":/go/src/ -w /go/src beego bee api api -conn="root:root@tcp(mysql:3306)/mvoice"
```

### 运行 Api 应用

```docker
docker run --rm --link db:mysql -v "$(pwd)/api":/go/src/api -w /go/src/api -p 8080:8080 --name api beego bee run -downdoc=true -gendoc=true
```

## 运行 Nginx

### 启动 Nginx，链接容器

```docker
docker run --name nginx --link api:beego -v "$(pwd)"/nginx:/etc/nginx/conf.d/ -p 80:80 -d nginx
```

绑定 80 端口后访问 http://localhost/swagger/


## 其他
### 利用 Docker 搭建直播服务环境
Docker 的好处就是部署环境非常方便，比如，我的 Mac 本地不需要安装 Golang 的编译运行环境，但是利用 Docker 就可以运行带有 Golang 应用的容器，对于应用部署环境的管理是非常方便的。

之前有一段时间在看直播相关的内容，直播的服务环境需要部署 Nginx + ffmpeg 的环境，但是如果直接安装到自己的电脑上也会出现很多问题，然而直接用 Docker 来部署环境就非常方便，如果不用了就可以直接删除掉，也不会污染自己本机的开发坏境。这里分享一个比较不错的直播 Docker 镜像。

```docker
docker pull alfg/nginx-rtmp
docker run -it -p 1935:1935 -p 8080:80 --rm alfg/nginx-rtmp
```

# 小结
本文主要介绍了 Docker 的相关概念、Docker 的基本使用，以及 Docker 部署 Golang Web App 的方法。

- Docker 是 C/S 架构的，包含服务端（dockerd 命令）、REST API和 Docker 客户端（CLI docker 命令）。
- **镜像是一个用指令来创建 Docker 容器的只读模板。**
- **容器是一个可运行的镜像的实例。**

# 参考资料
- Docker 概述 https://docs.docker.com/engine/docker-overview/
- Docker run reference https://docs.docker.com/engine/reference/run/
- Docker 从入门到实践 https://yeasy.gitbooks.io/docker_practice/
- 装在 Docker 里面的 Beego https://github.com/lei-cao/beego-in-action/blob/master/zh/beego-in-docker.md
- ECS上搭建Docker(CentOS7) https://help.aliyun.com/document_detail/51853.html?spm=5176.doc25426.6.724.TDd5eg
- Simple Go Web App https://github.com/niyaoyao/http-response-for-go-web-application