# Docker
[原文连接](https://www.runoob.com/docker/docker-container-connection.html)
[toc]

## 2、Docker 架构
&emsp;&emsp;Docker 包括三个基本概念：
* 镜像（Image）：Docker 镜像，就相当于是一个 root 文件系统。
* 容器（Container）：镜像和容器的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
* 仓库（Repository）：仓库可看成一个代码控制中心，用来保存镜像。


&emsp;&emsp;Docker 使用客户端-服务器 (C/S) 架构模式，使用远程API来管理和创建Docker容器。
&emsp;&emsp;Docker 容器通过 Docker 镜像来创建。



| 概念                   | 说明                                                         |
| ---------------------- | ------------------------------------------------------------ |
| Docker 镜像(Images)    | Docker 镜像是用于创建 Docker 容器的模板，比如 Ubuntu 系统。  |
| Docker 容器(Container) | 容器是独立运行的一个或一组应用，是镜像运行时的实体。         |
| Docker 客户端(Client)  | Docker 客户端通过命令行或者其他工具使用 Docker SDK (https://docs.docker.com/develop/sdk/) 与 Docker 的守护进程通信。 |
| Docker 主机(Host)      | 一个物理或者虚拟的机器用于执行 Docker 守护进程和容器。       |
| Docker Registry        | Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。<br>Docker Hub([https://hub.docker.com](https://hub.docker.com/))提供了庞大的镜像集合供使用。一个 Docker Registry 中可以包含多个仓库；每个仓库可以包含多个标签（tag）；每个标签对应一个镜像。<br>通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 **<仓库名>:<标签>** 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 **latest** 作为默认标签。 |
| Docker Machine         | Docker Machine是一个简化Docker安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装Docker，比如VirtualBox、 Digital Ocean、Microsoft Azure。 |


## 3、Docker 安装
### 3.1 Ubuntu Docker 安装
#### 3.1.1 使用官方安装脚本自动安装
&emsp;&emsp;安装命令如下：
```shell
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```
&emsp;&emsp;也可以使用国内 daocloud 一键安装命令：
```shell
curl -sSL https://get.daocloud.io/docker | sh
```

#### 3.1.2 手动安装
Docker 的旧版本被称为 docker，docker.io 或 docker-engine 。如果已安装，请卸载它们：
```shell
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```
&emsp;&emsp;当前称为 Docker Engine-Community 软件包 docker-ce 。
&emsp;&emsp;安装 Docker Engine-Community，以下介绍两种方式。

#### 3.1.3 使用 shell 脚本进行安装


### 3.2 卸载 docker
&emsp;&emsp;删除安装包：
```shell
sudo apt-get purge docker-ce
```
&emsp;&emsp;删除镜像、容器、配置文件等内容：
```shell
sudo rm -rf /var/lib/docker
```

## 4、Docker 使用
### 4.1 Docker Hello world
&emsp;&emsp;Docker 允许你在容器内运行应用程序， 使用`docker run`命令来在容器内运行一个应用程序。
&emsp;&emsp;输出 Hello world：
```shell
runoob@runoob:~$ docker run ubuntu:15.10 /bin/echo "Hello world"
Hello world
```
&emsp;&emsp;各个参数解析：
* docker: Docker 的二进制执行文件。
* run: 与前面的 docker 组合来运行一个容器。
* ubuntu:15.10 指定要运行的镜像，Docker 首先从本地主机上查找镜像是否存在，如果不存在，Docker 就会从镜像仓库 Docker Hub 下载公共镜像。
* /bin/echo "Hello world": 在启动的容器里执行的命令

&emsp;&emsp;以上命令完整的意思可以解释为：Docker 以 ubuntu15.10 镜像创建一个新容器，然后在容器里执行 bin/echo "Hello world"，然后输出结果。

#### 4.1.1 运行交互式的容器
&emsp;&emsp;我们通过 docker 的两个参数 -i -t，让 docker 运行的容器实现“对话”的能力：
```shell
runoob@runoob:~$ docker run -i -t ubuntu:15.10 /bin/bash
root@0123ce188bd8:/#
```
&emsp;&emsp;各个参数解析：
* -t: 在新容器内指定一个伪终端或终端。
* -i: 允许你对容器内的标准输入 (STDIN) 进行交互。
&emsp;&emsp;注意第二行 root@0123ce188bd8:/#，此时我们已进入一个 ubuntu15.10 系统的容器。
&emsp;&emsp;我们尝试在容器中运行命令 cat /proc/version和ls分别查看当前系统的版本信息和当前目录下的文件列表。
&emsp;&emsp;我们可以通过运行 exit 命令或者使用 CTRL + D 来退出容器。

#### 4.1.2 启动容器（后台模式）
&emsp;&emsp;使用以下命令创建一个以进程方式运行的容器：
```shell
runoob@runoob:~$ docker run -d ubuntu:15.10 /bin/sh -c "while true; do echo hello world; sleep 1; done"
2b1b7a428627c51ab8810d541d759f072b4fc75487eed05812646b8534a2fe63
```
&emsp;&emsp;在输出中，我们没有看到期望的 "hello world"，而是一串长字符
&emsp;&emsp;2b1b7a428627c51ab8810d541d759f072b4fc75487eed05812646b8534a2fe63
&emsp;&emsp;这个长字符串叫做容器 ID，对每个容器来说都是唯一的，我们可以通过容器 ID 来查看对应的容器发生了什么。
&emsp;&emsp;首先，我们需要确认容器有在运行，可以通过 docker ps 来查看：
```shell
$ docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED         STATUS         PORTS     NAMES
0a524448dd1d   ubuntu:15.10    "/bin/sh -c 'while t…"   5 seconds ago   Up 4 seconds             wizardly_ritchie
017d01aa34fe   ubuntu:latest   "/bin/bash"              2 months ago    Up 2 months              sad_chatterjee
```
&emsp;&emsp;输出详情介绍：
* CONTAINER ID: 容器 ID。
* IMAGE: 使用的镜像。
* COMMAND: 启动容器时运行的命令。
* CREATED: 容器的创建时间。
* STATUS: 容器状态。
&emsp;&emsp;状态有7种：
    * created（已创建）
    * restarting（重启中）
    * running 或 Up（运行中）
    * removing（迁移中）
    * paused（暂停）
    * exited（停止）
    * dead（死亡）
* PORTS: 容器的端口信息和使用的连接类型（tcp\udp）。
* NAMES: 自动分配的容器名称。

&emsp;&emsp;在宿主主机内使用 docker logs 命令，查看容器内的标准输出：
```shell
runoob@runoob:~$ docker logs 2b1b7a428627
```

#### 4.1.3 停止容器
&emsp;&emsp;我们使用`docker stop`命令来停止容器。
&emsp;&emsp;通过`docker ps`查看，容器已经停止工作。
&emsp;&emsp;也可以使用下面的命令来停止：
```shell
docker stop [docker_names]
```


### 4.2 Docker 容器使用
#### 4.2.1 Docker 客户端
&emsp;&emsp;docker 客户端非常简单，我们可以直接输入 docker 命令来查看 Docker 客户端的所有命令选项。
#### 4.2.2 容器使用
##### 4.2.2.1 获取镜像
&emsp;&emsp;如果我们本地没有 ubuntu 镜像，我们可以使用 docker pull 命令来载入 ubuntu 镜像：
```shell
docker pull ubuntu
```
##### 4.2.2.2 启动容器
&emsp;&emsp;以下命令使用 ubuntu 镜像启动一个容器，参数为以命令行模式进入该容器：
```shell
docker run -it ubuntu /bin/bash
```
&emsp;&emsp;参数说明：
* -i: 交互式操作。
* -t: 终端。
* ubuntu: ubuntu 镜像。
* /bin/bash：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。
&emsp;&emsp;要退出终端，直接输入 exit.

##### 4.2.2.3 启动已停止运行的容器
&emsp;&emsp;查看所有的容器命令如下：
```shell
docker ps -a
```
&emsp;&emsp;使用 docker start 启动一个已停止的容器：
```shell
docker start [docker id]
```

##### 4.2.2.4 后台运行
&emsp;&emsp;在大部分场景下，我们希望 docker 的服务是在后台运行的，我们可以通过`-d`指定容器的运行模式。
```shell
$ docker run -itd --name ubuntu-test ubuntu /bin/bash
```
&emsp;&emsp;加了 -d 参数默认不会进入容器，想要进入容器需要使用指令 docker exec。

##### 4.2.2.5 停止一个容器
&emsp;&emsp;停止容器命令如下：
```shell
$ docker stop <容器 ID>
```
&emsp;&emsp;停止的容器可以通过 docker restart 重启：
```shell
$ docker restart <容器 ID>
```

##### 4.2.2.6 进入容器
&emsp;&emsp;在使用 -d 参数时，容器启动后会进入后台。此时想要进入容器，可以通过以下指令进入：
* docker attach
* docker exec：推荐大家使用 docker exec 命令，因为此命令会退出容器终端，但不会导致容器的停止。

###### attach 命令
&emsp;&emsp;下面演示了使用`docker attach`命令
```shell
docker attach 1e560fca3906 
```
&emsp;&emsp;注意：如果从这个容器退出，会导致容器的停止。

###### exec 命令
&emsp;&emsp;下面演示了使用 docker exec 命令。
```shell
docker exec -it 243c32535da7 /bin/bash
```
&emsp;&emsp;如果从这个容器退出，容器不会停止，这就是为什么推荐大家使用 docker exec 的原因。
&emsp;&emsp;更多参数说明请使用 docker exec --help 命令查看。

##### 4.2.2.7 导出和导入容器
###### 导出容器
&emsp;&emsp;如果要导出本地某个容器，可以使用 docker export 命令。
```shell
docker export 1e560fca3906 > ubuntu.tar
```
&emsp;&emsp;导出容器 1e560fca3906 快照到本地文件 ubuntu.tar。

###### 导入容器快照
&emsp;&emsp;可以使用 docker import 从容器快照文件中再导入为镜像，以下实例将快照文件 ubuntu.tar 导入到镜像 test/ubuntu:v1:
```shell
$ cat docker/ubuntu.tar | docker import - test/ubuntu:v1
```
**上面这个命令我没看懂是在干嘛**
&emsp;&emsp;此外，也可以通过指定 URL 或者某个目录来导入，例如：
```shell
$ docker import http://example.com/exampleimage.tgz example/imagerepo
```

##### 4.2.2.8 删除容器
&emsp;&emsp;删除容器使用 docker rm 命令：
```shell
$ docker rm -f 1e560fca3906
```
&emsp;&emsp;下面的命令可以清理掉所有处于终止状态的容器：
```shell
$ docker container prune
```

### 4.3 Docker 镜像使用
&emsp;&emsp;当运行容器时，使用的镜像如果在本地中不存在，docker 就会自动从 docker 镜像仓库中下载，默认是从 Docker Hub 公共镜像源下载。
&emsp;&emsp;下面我们来学习：
* 管理和使用本地 Docker 主机镜像
* 创建镜像

#### 4.3.1 列出镜像列表
&emsp;&emsp;我们可以使用 docker images 来列出本地主机上的镜像
```shell
$ docker images
REPOSITORY        TAG       IMAGE ID       CREATED          SIZE
<none>            <none>    7ff3d6b19811   12 minutes ago   119MB
test/ubuntu       v1        47595d59a8c0   17 minutes ago   119MB
ubuntu            latest    27941809078c   5 days ago       77.8MB
ubuntu            <none>    825d55fb6340   2 months ago     72.8MB
hello-world       latest    feb5d9fea6a5   8 months ago     13.3kB
ubuntu            15.10     9b9cb95443b5   5 years ago      137MB
training/webapp   latest    6fae60ef3446   7 years ago      349MB
```
&emsp;&emsp;各个选项说明:
* REPOSITORY：表示镜像的仓库源
* TAG：镜像的标签
* IMAGE ID：镜像ID
* CREATED：镜像创建时间
* SIZE：镜像大小

&emsp;&emsp;同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本，如 ubuntu 仓库源里，有 15.10、14.04 等多个不同的版本，我们使用 REPOSITORY:TAG 来定义不同的镜像。
&emsp;&emsp;所以，我们如果要使用版本为15.10的ubuntu系统镜像来运行容器时，命令如下：
```shell
runoob@runoob:~$ docker run -t -i ubuntu:15.10 /bin/bash 
root@d77ccb2e5cca:/#
```
&emsp;&emsp;参数说明：
* -i: 交互式操作。
* -t: 终端。
* ubuntu:15.10: 这是指用 ubuntu 15.10 版本镜像为基础来启动容器。
* /bin/bash：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。

&emsp;&emsp;如果要使用版本为 14.04 的 ubuntu 系统镜像来运行容器时，命令如下：
```shell
runoob@runoob:~$ docker run -t -i ubuntu:14.04 /bin/bash 
root@39e968165990:/# 
```
&emsp;&emsp;如果你不指定一个镜像的版本标签，例如你只使用 ubuntu，docker 将默认使用 ubuntu:latest 镜像。

#### 4.3.2 获取一个新的镜像
&emsp;&emsp;当我们在本地主机上使用一个不存在的镜像时 Docker 就会自动下载这个镜像（会先在本地查找，找不到会从镜像源下载，如果镜像源也没有或输出错误信息）。如果我们想预先下载这个镜像，我们可以使用 docker pull 命令来下载它。
```shell
Crunoob@runoob:~$ docker pull ubuntu:13.10
```
&emsp;&emsp;下载完成后，我们可以直接使用这个镜像来运行容器。

#### 4.3.3 查找镜像
&emsp;&emsp;我们可以从 Docker Hub 网站来搜索镜像，Docker Hub 网址为： https://hub.docker.com/
&emsp;&emsp;我们也可以使用 docker search 命令来搜索镜像。比如我们需要一个 httpd 的镜像来作为我们的 web 服务。我们可以通过 docker search 命令搜索 httpd 来寻找适合我们的镜像。
```shell
docker search httpd
```
&emsp;&emsp;结果如下所示：
```shell
$ docker search httpd
NAME                                 DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
httpd                                The Apache HTTP Server Project                  4046      [OK]       
centos/httpd-24-centos7              Platform for running Apache httpd 2.4 or bui…   44                   
centos/httpd                                                                         35                   [OK]
hypoport/httpd-cgi                   httpd-cgi                                       2                    [OK]
solsson/httpd-openidc                mod_auth_openidc on official httpd image, ve…   2                    [OK]
dockerpinata/httpd                                                                   1                    
lead4good/httpd-fpm                  httpd server which connects via fcgi proxy h…   1                    [OK]
centos/httpd-24-centos8                                                              1                    
manageiq/httpd                       Container with httpd, built on CentOS for Ma…   1                    [OK]
nnasaki/httpd-ssi                    SSI enabled Apache 2.4 on Alpine Linux          1                    
inanimate/httpd-ssl                  A play container with httpd, ssl enabled, an…   1                    [OK]
dariko/httpd-rproxy-ldap             Apache httpd reverse proxy with LDAP authent…   1                    [OK]
clearlinux/httpd                     httpd HyperText Transfer Protocol (HTTP) ser…   1                    
publici/httpd                        httpd:latest                                    1                    [OK]
```
* NAME: 镜像仓库源的名称
* DESCRIPTION: 镜像的描述
* OFFICIAL: 是否 docker 官方发布
* stars: 类似 Github 里面的 star，表示点赞、喜欢的意思。
* AUTOMATED: 自动构建。

#### 4.3.4 拖取镜像
&emsp;&emsp;如果要使用上面 httpd 官方版本的镜像，使用命令 docker pull 来下载镜像
```shell
docker pull httpd
```

#### 4.3.5 删除镜像
&emsp;&emsp;镜像删除使用 docker rmi 命令，例如：
```shell
docker rmi [docker id/docker name:version]
```
&emsp;&emsp;如果有容器正在使用对应的镜像，会删除不成功，此时停止掉对应的容器，再删除即可。

#### 4.3.6 创建镜像
&emsp;&emsp;当我们从 docker 镜像仓库中下载的镜像不能满足我们的需求时，我们可以通过以下两种方式对镜像进行更改：
* 从已经创建的容器中更新镜像，并且提交这个镜像。
* 使用 Dockerfile 指令来创建一个新的镜像。

##### 4.3.6.1 更新镜像
&emsp;&emsp;更新镜像之前，我们需要使用镜像来创建一个容器：
```shell
runoob@runoob:~$ docker run -t -i ubuntu:15.10 /bin/bash
root@e218edb10161:/# 
```
&emsp;&emsp;在运行的容器内使用 apt-get update 命令进行更新。
&emsp;&emsp;在完成操作之后，输入 exit 命令来退出这个容器。
&emsp;&emsp;此时 ID 为 e218edb10161 的容器，是按我们的需求更改的容器。我们可以通过命令 docker commit 来提交容器副本。
```shell
runoob@runoob:~$ docker commit -m="has update" -a="runoob" e218edb10161 runoob/ubuntu:v2
sha256:70bf1840fd7c0d2d8ef0a42a817eb29f854c1af8f7c59fc03ac7bdee9545aff8
```
&emsp;&emsp;各个参数说明：
* -m: 提交的描述信息
* -a: 指定镜像作者
* e218edb10161：容器 ID
* runoob/ubuntu:v2: 指定要创建的目标镜像名
&emsp;&emsp;我们可以使用 docker images 命令来查看我们的新镜像 runoob/ubuntu:v2。
&emsp;&emsp;使用新镜像 runoob/ubuntu 来启动一个容器：
```shell
runoob@runoob:~$ docker run -t -i runoob/ubuntu:v2 /bin/bash                            
root@1a9fbdeb5da3:/#
```

##### 4.3.6.2 构建镜像
&emsp;&emsp;我们使用命令 docker build，从零开始创建一个新的镜像。为此，我们需要创建一个 Dockerfile 文件，其中包含一组指令来告诉 Docker 如何构建我们的镜像。
```shell
runoob@runoob:~$ cat Dockerfile 
FROM    centos:6.7
MAINTAINER      Fisher "fisher@sudops.com"

RUN     /bin/echo 'root:123456' |chpasswd
RUN     useradd runoob
RUN     /bin/echo 'runoob:123456' |chpasswd
RUN     /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
EXPOSE  22
EXPOSE  80
CMD     /usr/sbin/sshd -D
```
&emsp;&emsp;每一个指令都会在镜像上创建一个新的层，每一个指令的前缀都必须是大写的。
&emsp;&emsp;第一条FROM，指定使用哪个镜像源。
&emsp;&emsp;RUN 指令告诉docker 在镜像内执行命令，安装了什么。
&emsp;&emsp;然后，我们使用 Dockerfile 文件，通过 docker build 命令来构建一个镜像。
```shell
docker build -t runoob/centos:6.7 .
```
&emsp;&emsp;参数说明：
* -t ：指定要创建的目标镜像名
8 . ：Dockerfile 文件所在目录，可以指定Dockerfile 的绝对路径
&emsp;&emsp;使用docker images 查看创建的镜像已经在列表中存在。
&emsp;&emsp;我们可以使用新的镜像来创建容器。
```shell
docker run -t -i runoob/centos:6.7  /bin/bash
```


##### 4.3.6.3 设置镜像标签
&emsp;&emsp;我们可以使用 docker tag 命令，为镜像添加一个新的标签。
```shell
runoob@runoob:~$ docker tag 860c279d2fec runoob/centos:dev
```
&emsp;&emsp;docker tag 镜像ID，这里是 860c279d2fec ,用户名称、镜像源名(repository name)和新的标签名(tag)。

### 4.4 Docker 容器连接
&emsp;&emsp;容器中可以运行一些网络应用，要让外部也可以访问这些应用，可以通过 -P 或 -p 参数来指定端口映射。下面我们来实现通过端口连接到一个 docker 容器。
#### 4.4.1 网络端口映射
&emsp;&emsp;我们创建了一个 python 应用的容器
```shell
runoob@runoob:~$ docker run -d -P training/webapp python app.py
fce072cc88cee71b1cdceb57c2821d054a4a59f67da6b416fceb5593f059fc6d
```
&emsp;&emsp;可以使用 -P 或 -p 标识来指定容器端口绑定到主机端口。
&emsp;&emsp;两种方式的区别是：
* -P：是容器内部端口随机映射到主机的端口
* -p：是容器内部端口绑定到指定的主机的端口

```shell
runoob@runoob:~$ docker run -d -p 5000:5000 training/webapp python app.py
```

&emsp;&emsp;另外，我们可以指定容器绑定的网络地址，比如绑定 127.0.0.1
```shell
runoob@runoob:~$ docker run -d -p 127.0.0.1:5001:5000 training/webapp python app.py
```
&emsp;&emsp;这样我们就可以通过访问 127.0.0.1:5001 来访问容器的 5000 端口。
&emsp;&emsp;上面的例子中，默认都是绑定 tcp 端口，如果要绑定 UDP 端口，可以在端口后面加上 /udp。
```shell
runoob@runoob:~$ docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py
```
&emsp;&emsp;docker port 命令可以让我们快捷的查看端口的绑定情况。
```shell
runoob@runoob:~$ docker port adoring_stonebraker 5000
127.0.0.1:5001
```
#### 4.4.2 Docker 容器互联
&emsp;&emsp;端口映射并不是唯一把 docker 连接到另一个容器的方法。
&emsp;&emsp;docker 有一个连接系统允许将多个容器连接在一起，共享连接信息。
&emsp;&emsp;docker 连接会创建一个父子关系，其中父容器可以看到子容器的信息。
##### 4.4.2.1 容器命名
&emsp;&emsp;当我们创建一个容器的时候，docker 会自动对它进行命名。另外，我们也可以使用 --name 标识来命名容器，例如：
```shell
runoob@runoob:~$  docker run -d -P --name runoob training/webapp python app.py
```
&emsp;&emsp;我们可以使用 docker ps 命令来查看容器名称。
```shell
runoob@runoob:~$ docker ps -l
CONTAINER ID     IMAGE            COMMAND           ...    PORTS                     NAMES
43780a6eabaa     training/webapp   "python app.py"  ...    0.0.0.0:32769->5000/tcp   runoob
```

##### 4.4.2.2 新建网络
&emsp;&emsp;先创建一个新的 Docker 网络
```shell
$ docker network create -d bridge test-net
```
&emsp;&emsp;参数说明：
* -d：参数指定 Docker 网络类型，有 bridge、overlay。
其中 overlay 网络类型用于 Swarm mode，在本小节中你可以忽略它。

##### 4.4.2.3 连接容器
&emsp;&emsp;运行一个容器并连接到新建的 test-net 网络：
```shell
$ docker run -itd --name test1 --network test-net ubuntu /bin/bash
```
&emsp;&emsp;打开新的终端，再运行一个容器并加入到 test-net 网络。
```shell
$ docker run -itd --name test2 --network test-net ubuntu /bin/bash
```
&emsp;&emsp;下面通过 ping 来证明 test1 容器和 test2 容器建立了互联关系。
&emsp;&emsp;如果 test1、test2 容器内无 ping 命令，则在容器内执行以下命令安装 ping。
```shell
apt-get update
apt install iputils-ping
```
&emsp;&emsp;在 test1 容器输入以下命令：
```shell
ping test2
```
&emsp;&emsp;可以看到成功 ping 通了。
&emsp;&emsp;这样，test1 容器和 test2 容器建立了互联关系。

#### 4.4.3 配置 DNS

### 4.5 Docker 仓库管理
&emsp;&emsp;仓库（Repository）是集中存放镜像的地方。以下介绍一下 Docker Hub。当然不止 docker hub，只是远程的服务商不一样，操作都是一样的。

#### 4.4.5.1 Docker Hub
&emsp;&emsp;目前 Docker 官方维护了一个公共仓库 Docker Hub.
&emsp;&emsp;大部分需求都可以通过在 Docker Hub 中直接下载镜像来实现。

......

### 4.6 Docker Dockerfile
&emsp;&emsp;Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。

#### 4.6.1 使用 Dockerfile 定制镜像
1. 下面以定制一个 nginx 镜像（构建好的镜像内会有一个 /usr/share/nginx/html/index.html 文件）
&emsp;&emsp;在一个空目录下，新建一个名为 Dockerfile 文件，并在文件内添加以下内容：
```shell
FROM nginx
RUN echo '这是一个本地构建的nginx镜像' > /usr/share/nginx/html/index.html
```

2. FROM 和 RUN 指令的作用
* FROM：定制的镜像都是基于 FROM 后的镜像，这里的 nginx 就是定制需要的基础镜像。后续的操作都是基于 nginx。
* RUN：用于执行后面跟着的命令行命令。

&emsp;&emsp;shell 格式：
```shell
RUN <命令行命令>
# <命令行命令> 等同于，在终端操作的 shell 命令。
```
&emsp;&emsp;exec 格式：
```shell
RUN ["可执行文件", "参数1", "参数2"]
# 例如：
# RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline
```
&emsp;&emsp;注意：Dockerfile 的指令每执行一次都会在 docker 上新建一层。所以过多无意义的层，会造成镜像膨胀过大。例如：
```shell
FROM centos
RUN yum -y install wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN tar -xvf redis.tar.gz
```
&emsp;&emsp;以上执行会创建 3 层镜像。可简化为以下格式：
```shell
FROM centos
RUN yum -y install wget \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && tar -xvf redis.tar.gz
```
&emsp;&emsp;如上，以 && 符号连接命令，这样执行后，只会创建 1 层镜像。

#### 4.6.2 开始构建镜像
&emsp;&emsp;在 Dockerfile 文件存放目录下，执行构建动作。
&emsp;&emsp;以下示例，通过目录下的 Dockerfile 构建一个 nginx:v3（镜像名称:镜像标签）。
&emsp;&emsp;注：最后的 . 代表本次执行的上下文路径，下一节会介绍。
```shell
$ docker build -t nginx:v3 .
```

#### 4.6.3 上下文路径
&emsp;&emsp;上一节中，有提到指令最后一个 . 是上下文路径，那么什么是上下文路径呢？
```shell
$ docker build -t nginx:v3 .
```
&emsp;&emsp;上下文路径，是指 docker 在构建镜像，有时候想要使用到本机的文件（比如复制），`docker build`命令得知这个路径后，会将路径下的所有内容打包。
&emsp;&emsp;解析：由于 docker 的运行模式是 C/S。我们本机是 C，docker 引擎是 S。实际的构建过程是在 docker 引擎下完成的，所以这个时候无法用到我们本机的文件。这就需要把我们本机的指定目录下的文件一起打包提供给 docker 引擎使用。
&emsp;&emsp;如果未说明最后一个参数，那么默认上下文路径就是 Dockerfile 所在的位置。
&emsp;&emsp;注意：**上下文路径下不要放无用的文件，因为会一起打包发送给 docker 引擎，如果文件过多会造成过程缓慢**。

#### 4.6.4 指令详解
##### 4.6.4.1 COPY
&emsp;&emsp;复制指令，从上下文目录中复制文件或者目录到容器里指定路径。
&emsp;&emsp;格式：
```shell
COPY [--chown=<user>:<group>] <源路径1>...  <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",...  "<目标路径>"]
```

* `[--chown=<user>:<group>]`：可选参数，用户改变复制到容器内文件的拥有者和属组。
* `<源路径>`：源文件或者源目录，这里可以是通配符表达式，其通配符规则要满足 Go 的 filepath.Match 规则。例如：
```shell
COPY hom* /mydir/
COPY hom?.txt /mydir/
```
* `<目标路径>`：容器内的指定路径，该路径不用事先建好，路径不存在的话，会自动创建。

##### 4.6.4.2 ADD
&emsp;&emsp;ADD 指令和 COPY 的使用格类似（同样需求下，官方推荐使用 COPY）。功能也类似，不同之处如下：
* ADD 的优点：在执行 <源文件> 为 tar 压缩文件的话，压缩格式为 gzip, bzip2 以及 xz 的情况下，会自动复制并解压到 <目标路径>。
* ADD 的缺点：在不解压的前提下，无法复制 tar 压缩文件。会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。**具体是否使用，可以根据是否需要自动解压来决定**。


##### 4.6.4.3 CMD
&emsp;&emsp;类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:
* CMD 在docker run 时运行。
* RUN 是在 docker build。

&emsp;&emsp;作用：为启动的容器指定默认要运行的程序，程序运行结束，容器也就结束。CMD 指令指定的程序可被 docker run 命令行参数中指定要运行的程序所覆盖。
&emsp;&emsp;注意：**如果 Dockerfile 中如果存在多个 CMD 指令，仅最后一个生效**。
&emsp;&emsp;格式：
```shell
CMD <shell 命令> 
CMD ["<可执行文件或命令>","<param1>","<param2>",...] 
CMD ["<param1>","<param2>",...]  # 该写法是为 ENTRYPOINT 指令指定的程序提供默认参数
```
&emsp;&emsp;推荐使用第二种格式，执行过程比较明确。第一种格式实际上在运行的过程中也会自动转换成第二种格式运行，并且默认可执行文件是 sh。

##### 4.6.4.3 ENTRYPOINT
&emsp;&emsp;类似于 CMD 指令，但其不会被 docker run 的命令行参数指定的指令所覆盖，而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序。
&emsp;&emsp;但是, 如果运行 docker run 时使用了 --entrypoint 选项，将覆盖 ENTRYPOINT 指令指定的程序。
&emsp;&emsp;优点：在执行 docker run 的时候可以指定 ENTRYPOINT 运行所需的参数。
&emsp;&emsp;注意：**如果 Dockerfile 中如果存在多个 ENTRYPOINT 指令，仅最后一个生效**。
&emsp;&emsp;格式：
```shell
ENTRYPOINT ["<executeable>","<param1>","<param2>",...]
```
&emsp;&emsp;可以搭配 CMD 命令使用：一般是变参才会使用 CMD ，这里的 CMD 等于是在给 ENTRYPOINT 传参，以下示例会提到。
&emsp;&emsp;示例：
&emsp;&emsp;假设已通过 Dockerfile 构建了 nginx:test 镜像：
```shell
FROM nginx

ENTRYPOINT ["nginx", "-c"] # 定参
CMD ["/etc/nginx/nginx.conf"] # 变参 
```
1、不传参运行
```shell
$ docker run  nginx:test
```
&emsp;&emsp;容器内会默认运行以下命令，启动主进程。
```shell
nginx -c /etc/nginx/nginx.conf
```
2、传参运行
```shell
$ docker run  nginx:test -c /etc/nginx/new.conf
```
&emsp;&emsp;容器内会默认运行以下命令，启动主进程(/etc/nginx/new.conf:假设容器内已有此文件)
```shell
nginx -c /etc/nginx/new.conf
```

......


### 4.7 Docker Compose
#### 4.7.1 Compose 简介


























































