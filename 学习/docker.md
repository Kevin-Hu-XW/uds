# docker

## 为什么要使用docker？

- **更高效的利用系统资源**

  由于容器不需要硬件的虚拟以及运行完整的操作系统等额外开销，docker对系统资源的利用率登高

- **更快速的启动时间**

  由于docker直接运行在宿主内核，无需启动完整的操作系统，可以做到毫秒级启动甚至更快，大大的节约了开发、测试、部署的时间。

- **一致的运行环境**

  docker提供了出内核外完整的运行环境，确保了运行环境的一致性，避免了由于开发环境、测试环境、生产环境不一致出现的bug

- **持续交付和部署**

  对开发和运维（[DevOps](https://zh.wikipedia.org/wiki/DevOps)）人员来说，最希望的就是一次创建或配置，可以在任意地方正常运行,使用 `Docker` 可以通过定制应用镜像来实现持续集成、持续交付、部署。

- **更轻松的迁移**

  由于 `Docker` 确保了执行环境的一致性，使得应用的迁移更加容易。`Docker` 可以在很多平台上运行，无论是物理机、虚拟机、公有云、私有云，甚至是笔记本，其运行结果是一致的。因此用户可以很轻易的将在一个平台上运行的应用，迁移到另一个平台上，而不用担心运行环境的变化导致应用无法正常运行的情况。

- **更轻松的维护和扩展**

​		`Docker` 使用的分层存储以及镜像的技术，使得应用重复部分的复用更为容易，也使得应用的维护更新更加简单，基		于基础镜像进一步扩展镜像也变得非常简单。此外，`Docker` 团队同各个开源项目团队一起维护了一大批高质量的 [官		方镜像](https://hub.docker.com/search/?type=image&image_filter=official)，既可以直接在生产环境使用，又可以作为基础进一步定制，大大的降低了应用服务的镜像制作成本。

![](C:\Users\Admin\Desktop\工作\uds\学习\docker作用.PNG)
## docker架构原理

![](C:\Users\Admin\Desktop\工作\uds\学习\docker运行原理图.jpg)

**三大核心要素：**

1. 仓库——专门存放我们的镜像文件，类似于APP软件市场

2. 镜像——类似于安装包，描述运行所需要的环境配置和依赖，Redis镜像、Tomcat镜像等

   **镜像来源：**

   自己做镜像，SpringBoot项目（自己创建一个镜像文件）

   拉取别人制作好的镜像（从docker hub仓库下载），例如 Nginx、Mysql、Redis等

3. 容器——专门运行镜像文件，自己独立的ip和网络信息，虚拟化出一个轻量级的Linux操作系统精简版本

   容器就是镜像运行的实例，容器的状态可以分为：初创建、运行、停止、暂停、删除

   一个镜像可以创建多个不同的容器

   阿里云加速配置：https://liiv47lf.mirror.aliyuncs.com

   

![](C:\Users\Admin\Desktop\工作\uds\学习\docker运行原理.PNG)
## 容器与虚拟机

什么是虚拟机：在一台物理机器上利用虚拟化技术，虚拟出来多个操作系统，每个操作系统之间是隔离的

<img src="C:\Users\Admin\Desktop\工作\uds\学习\虚拟机.png" style="zoom:50%;" />

Docker ：Docker是开源的应用容器引擎

<img src="C:\Users\Admin\Desktop\工作\uds\学习\容器.png" style="zoom:50%;" />

区别：

1. 从两者的架构图上看，==虚拟机是在硬件级别进行虚拟化，模拟硬件搭建操作系统；而Docker是在操作系统的层面虚拟化，复用操作系统，运行Docker容器==。

2. Docker的速度很快，秒级，而虚拟机的速度通常要按分钟计算。

3. Docker所用的资源更少，性能更高。同样一个物理机器，Docker运行的镜像数量远多于虚拟机的数量。

4. ==虚拟机实现了操作系统之间的隔离，Docker是进程之间的隔离，虚拟机隔离级别更高、安全性方面也更强。==

5. 虚拟机和Docker各有优势，不存在谁替代掉谁的问题，很多企业都采用物理机上做虚拟机，虚拟机中跑Docker的方式。

| 特性       | 容器               | 虚拟机     |
| ---------- | ------------------ | ---------- |
| 启动速度   | 秒级               | 分钟级别   |
| 硬盘使用   | 一般为MB           | 一般GB     |
| 性能       | 接近原生           | 弱于       |
| 系统支持量 | 单机支持上千个容器 | 一般几十个 |
| 隔离性     | 完全隔离           | 完全隔离   |
## docker安装

Docker 要求 CentOS7系统的内核版本在 3.10以上 ，查看本页面的前提条件来验证你的CentOS 版本是否支持 Docker 

1、通过 ==uname -r==命令查看你当前的内核版本

2、使用 root 权限登录 Centos，通过==yum -y update==确保 yum 包更新到最新。

3、卸载旧版本（如果安装版本过久的话）==yum remove docker docker-common docker-selinux docker-engine==

4、安装需要的软件包==yum install -y yum-utils device-mapper-persistent-data lvm2==

5、设置yum源yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

6、可以查看所有仓库中所有docker版本，并选择特定版本安装==yum list docker-ce --showduplicates | sort -r==

7、安装docker ==sudo yum install -y docker-ce==

8、启动并加入开机启动==systemctl start docker==  、  ==systemctl enable docker==

9、验证安装是否成功(有client和service两部分表示docker安装启动都成功了) ==docker version==

## docker常用命令

```shell
## 查看本地images 镜像缓存
docker images
## 搜索
docker search
##删除镜像
docker rmi image id
##
docker bulid
## 下载，不写版本号，默认下载最新的
docker pull name:version
## 指定容器运行镜像文件
docker run --name *** tomcat
##查看正在运行的容器
docker ps  获取容器id
docker inspect 容器id
## 查看已经存在的容器
docker ps -a
##进入容器
docker exec -it 容器ID bash
##停止rqi
docker stop 容器id
#删除容器
docker rm 容器ID
## 供外部访问的启动方式 8081:容器外部访问的端口号 8080：容器内部的端口号，后台启动（不会打印日志信息）
docker run --name my-tomcat -p 8081:8080 tomcat

## 前台启动会打印日志信息
docker run --name my-tomcat -p 8081:8080 -d tomcat

## commit 根据当前容器制作镜像文件
docker commit -m=“提交的描述信息” -a=“作者” 容器ID 要创建的目标镜像名:[标签名]

```

**docker run原理**

![](C:\Users\Admin\Desktop\工作\uds\学习\docker run运行原理.PNG)

## Docker 数据卷

数据卷就是宿主机上的一个文件或目录，当容器目录和数据卷（宿主机）目录绑定，双方修改会立即同步操作，一个数据卷可以被多个容器同时挂载

**数据卷的作用：**容器数据持久化、外部机器和容器间通信、容器之间数据交换使用-v命令

**数据卷的添加方式：** 

1. 直接命令形式添加 docker run -it -v 宿主机绝对路径目录:容器内目录 镜像文件名称

   ```shell
   docker run --name nginx81 -d -p 80:80 -v /data/nginx/html:/usr/share/nginx/html nginx
   ```

2. Dockerfile方式添加

## Docker 运行底层原理

![](C:\Users\Admin\Desktop\工作\uds\学习\docker底层原理.PNG)

**如何封装配置环境依赖？**

DockerFile --文件

## Dockerfile

**Dockerfile是什么**

Dockerfile是一个创建镜像所有命令的文本文件, 包含了一条条指令和说明, 每条指令构建一层, 通过docker build命令,根据Dockerfile的内容构建镜像,因此每一条指令的内容, 就是描述该层如何构建.有了Dockefile, 就可以制定自己的docker镜像规则,只需要在Dockerfile上添加或者修改指令, 就可生成docker 镜像.

docker build命令用于从Dockerfile构建映像。可以在docker build命令中使用-f标志指向文件系统中任何位置的Dockerfile。

```shell
docker build -f /path/to/a/Dockerfile
```

**Dockerfile指令**

```yaml
from:指定父镜像，基于哪个镜像image构建，指定基础镜像，必须为第一个命令
MAINTAINER:维护者
RUN: 容器创建的时候执行一段命令，构建镜像时执行的命令
ADD: 将本地文件添加到容器中，tar类型文件会自动解压(网络压缩资源不会被解压)，可以访问网络资源，类似wget
COPY:功能类似ADD，但是是不会自动解压文件，也不能访问网络资源
CMD:构建容器后调用，也就是在容器启动时才进行调用。 .sh执行文件
ENV: 设置环境变量
EXPOSE: 指定于外界交互的端口
VOLUME:用于指定持久化目录
WORKDIR:设置进入容器时的路径，类似于cd命令
```

**基于Dockerfile构建自己的centos**

1. 需要自己制作一个dockerfile文件

2. 继承docker hub中的centos

3. 在docker hubcentos 上加入以下两个功能

  	A.进入容器中 默认访问目录/usr
  	
  	B.实现支持vim插件

4. 需要将该dockerfile文件打包成一个镜像文件 交给我们容器执行



