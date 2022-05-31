# Docker
[原文链接](https://www.runoob.com/docker/docker-tutorial.html)  
## Docker 架构
&emsp;&emsp;Docker 包括三个基本概念：
* 镜像（iamge）：Dockers 镜像，就相当于是一个 root 文件系统。
* 容器（container）：镜像和容器和关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
* 仓库（repository）：仓库可看成是也给代码控制中心，用来保存镜像。

&emsp;&emsp;容器与镜像的关系类似于面向对象编程中的对象与类。

| Docker | 面向对象 |
| ------ | -------- |
| 容器   | 对象     |
| 镜像   | 类       |

| 概念                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| Docker 镜像         | Docker 镜像时用于创建 Docker 容器的模板，比如 Ubuntu 系统。  |
| Docker 容器         | 容器是独立运行的一个或一组应用，是镜像运行时的实体。         |
| Docker 客户端       | Docker 客户端通过命令行或其他工具使用 Docker SDK 与 Docker 的守护进程通信。 |
| Docker 主机（Host） | 一个物理或者虚拟的机器用于执行 Docker 守护进程的容器。       |
| Docker Registry     | Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。    |
| Docker Machine      | Docker Machine 是一个简化 Docker 安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装 Docker。 |

