# Docker与Docker-compose


Docker与Docker-compose详解

<!--more-->


1、Docker是什么？

在计算机中，虚拟化(英语: Virtualization) 是一种资源管理技术，是将计算机的各种实体资源，如服务器、网络、内存及存储等，予以抽象、转换后呈现出来，打破实体结构间的不可切割的障碍，使用户可以比原本的组态更好的方式来应用这些资源。这些资源的新虚拟部份是不受现有资源的架设方式，地域或物理组态所限制。一般所指的虚拟化资源包括计算能力和资料存储。

在实际的生产环境中。虚拟化技术主要用来解决高性能的物理硬件产能过利和老的旧的硬件产能过低的重组重用，透明化底层物理硬件，从而最大化的利用物理硬件资源的充分利用。

虚拟化技术种类很多，例如:软件虚拟化、硬件虚拟化、内存虚拟化、网络虚拟化、桌面虚拟化、服务虚拟化、虚拟机等等。.

**Docker和传统虚拟机的区别**


2、Docker的安装

2.1、Windows下的安装

直接在官网下载windows包双击运行即可，对于win10来说需要开启Hype-v，直接百度打开即可。

2.2、Linux下的安装

```java
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun

# 安装报错 Problem: problem with installed package buildah
# 执行语句 yum erase podman buildah 
# 再进行安装

systemctl status docker

systemctl restart docker

docker info

systemctl enable docker

# 建立docker组
sudo groupadd docker
sudo usermod -aG docker $USER

# 重启服务
systemctl restart docker
```

2.3、核心概念

1. 仓库 
 <ul> 
  1. 远程仓库：开发者镜像及官方镜像 
  1. 本地仓库：只保存当前自己使用过的镜像及自定义镜像 
  1. 作用：用来存放docker镜像位置 
 </ul>  
5. 镜像 
 <ul> 
  5. 作用：一个镜像就代表一个软件 
 </ul>  
7. 容器 
 <ul> 
  7. 作用：一个幢像运行一次就会生成一个实例就是生成一个容器 
 </ul> 

2.4、Aliyun服务加速

docker提供了一个远程仓库，主要是用来存放镜像的，而我们所需要的镜像都需要去远程仓库进行拉取，dockerHub 地址： [https://registry.hub.docker.com/_/mysql?tab=tags](https://registry.hub.docker.com/_/mysql?tab=tags)，这里以mysql镜像为例，然后直接在虚拟机当中执行命令

```java
# 获取最新版本的mysql
docker pull mysql

# 获取指定版本的mysql 8.0.18版
docker pull mysql:8.0.18
```

在这里的dockerhub是为全球服务的，速度难免会有点慢，这里可以配置阿里的镜像来进行获取docker远程仓库的镜像。阿里云服务加速配置 [https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)，按照官网进行配置即可：


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/958ddeba6bde480e9f915719f74bab9c.png) 3、Docker的操作

3.1、Hello World

在安装docker之后，直接使用命令：docker run hello-world 表示直接运行hello-world这个镜像。而他的执行基本步骤如下：


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/086988e779294fccbeedfd9570d0b903.png) 3.2、Docker 的基本命令

**3.2.1、docker引擎及帮助操作：**

```java
# 查看docker信息
docker info

# 查看docker版本
docker version

# 帮助命令
docker --help
```

**3.2.2、镜像相关操作：**

```java
# 查看镜像
docker images
docker images -a #展示所有镜像
docker images -q #只展示镜像的ID
docker images mysql #只展示mysql镜像

# 下载镜像
docker pull 镜像名称:版本号  # 如 docker pull mysql:8.0.27
docker pull 镜像名称:@DIGEST
#如：docker pull mysql:DIGEST:sha256:975b3b1a6df6bf66221d1702b76c4141a4cd09f93f22f70c32edc99a6c256fe8

# 搜索镜像
docker search 镜像
# docker search mysql
# 搜索stars数在3000以上的image
docker search mysql --filter=stars=3000


# 删除镜像
docker image rm 镜像名:版本或者id标识 # docker image rm mysql:8.0.27
docker image rm -f 镜像名:版本或者id标识 # 强制删除
# 简化删除
docker rmi 镜像名:版本

# 组合运用
# 清空本地仓库所有镜像
docker rmi -f $(docker images -q)
```

**3.2.3、容器相关操作：**

```java
# 导入本地镜像
docker load -i 镜像文件

# 运行一个容器
docker run 镜像名称:版本号
# 运行容器与宿主机进行映射
docker run -p 8080:8080 镜像名称:版本号
# 启动容器映射端口，后台启动
docker run -p 8080:8080 -d 镜像名称:版本号
# 启动容器映射端口，后台启动，指定名称
docker run -p 8080:8080 --name 容器名称 -d 镜像名称:版本号

# 查看正在运行的容器
docker ps
# 查看运行容器的历史记录
docker ps -a
# 查看最近运行的两个容器
docker ps -a -n=2
# 查看正在运行的容器id
docker ps -q
# 查看所有容器的id
docker ps -aq

# 容器的启动和停止
docker start 容器名称或者容器id 
docker restart 容器名称或者容器id 
docker stop 容器名称或者容器id 
docker kill 容器名称或者容器id 

# 容器的删除
docker rm 容器的id或者名称
docker rm -f 容器的id或者名称
docker rm -f $(docker ps -aq)

# 查看日志
docker logs 容器id或名称
# 实时展示日志
docker logs -f 容器id或名称
# 加入时间戳展示实时展示日志
docker logs -tf 容器id或名称
# 查看最后n行日志
docker logs --tail 5 容器id或名称

# 查看容器的内部进程
docker top 容器id或名称

# 与容器内部进行交互
docker exec -it 容器id或名称 bash

# 从容器复制文件到操作系统
docker cp 容器id:路径 操作系统下的路径
# 从操作系统复制文件到容器当中
docker cp 操作系统下的路径 容器id:路径
```

在这里的文件复制主要还是运用到本地项目打包后的部署，比如说这里一个项目开发完成之后，打成一个jar包或者war包，丢给tomcat进行启动部署，而后直接将这个包给到tomcat镜像下的webapps目录下，重新启动tomcat或者重启容器，最后进行访问项目。

```java
# 查看容器中的元数据
docker inspect 容器id

# 数据卷（Volume）：实现宿主机系统和容器之间的文件共享
# 数据卷的使用：
docker run -d -p 8080:8080 --name 容器名称 -v 操作系统下路径:容器下路径 镜像名称:版本

# aa代表一个数据卷的名字，名称可以自定义，docker在不存在时自动创建这个数据卷，并且同时自动映射宿主机当中的某个目录
# 同时在启动容器的时候会将aa对应容器目录中全部内容复制到aa映射目录当中
docker run -d -p 8080:8080 --name 容器名称  -v aa:容器下路径 镜像名称:版本

find / -name aa

# 将容器打成一个新的镜像
docker commit -m "描述信息" -a "作者信息" 容器id或名称 打包的镜像名称:标签版本

# 将镜像备份出来
docker save 镜像名称:版本 -o 文件名
# 重新加载镜像
docker load -i 镜像文件
```

3.3、Docker 镜像分层原理

镜像是一种轻量级的，可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码、运行时所需的库、环境变量和配置文件。

**Docker当中的镜像为什么这莫大？**

Docker的设计：一个软件镜像不仅仅是原来软件包，镜像当中还包含软件包所需的操作系统依赖软件自身依赖以及自身软件包组成。

**分层原理**

很显然，在这里docker容器的设计简单来说，对于不同的环境都给抽离出来进行分层，就比如说很多的软件服务（比如说：Naocs、ES、Hadoop等等）都需要jdk的环境，那再进行拉取镜像的时候，这些镜像都会先检验jdk的环境，再进行后续的安装，那这里装个Naocs、ES、Hadoop要下载三次JDK，这显然浪费了很多的内存，所以在这里Docker采用了分层的原理，这里每一层的环境依赖都给分开了，再一次安装了jdk环境之后，后续安装的服务也要jdk依赖就不会再去拉取了，回直接使用本地有的jdk环境。

3.4、Docker 网络

在docker当中容器和容器之间也是可以进行通信的。就好比Linux中我们使用 ip addr 可以看到当前虚拟机的ip地址，在这里可以查看一下容器中的IP，docker exec -it 容器名 ip addr 会发现有一个对应的映射另一个映射。说明docker容器网络是通过veth-pair技术实现的。

并且在这里还可以通过docker inspect 容器名称或id 命令查看容器的元数据，这里也有该容器随机分配的ip地址。

而当我们启动多个容器之后，可以查看多个容器的ip地址，可以看到容器的ip地址都在同一个网段上，这就有点似曾相识的感觉了，在linux当中我们配置多台机器进行互相通信，那这里的容器通信那也是一样的不，直接进入到一台容器之内，使用ping命令，ping另外一个容器的ip。

再就是在启动容器之后，默认为分配的ip地址都同一个网桥上，而这里容器当中需要对网桥进行分割开又要如何操作呢？我们需要创建一个网桥，而后在启动容器的时候指定对应的网桥即可。

```java
# 查看网桥
docker network ls

# 创建网桥
docker network create 网桥名称

# 容器指定网桥挂载
docker run -p -d 8080:8080 --network 网桥名称 --name 容器名称 镜像:版本

# 在启动容器，生成的ip地址会和容器名称进行映射，这里除了使用ip进行访问，还可以使用容器名称进行访问

# 删除
docker network rm 网桥名称

# 网桥细节
docker inspect 网桥名称
```

3.5、Docker 数据卷

**3.5.1、作用**

是用来实现容器和宿主机之间的数据共享

**3.5.2：特点**

- 数据卷可以在容器之间进行共享和重用 
- 对数据卷的修改会立即影响到对应的容器 
- 对数据卷的修改不会影响镜像 
- 数据卷默认一直存在，即使容器被删除

**3.5.3、数据卷操作**

```java
# 自定义数据卷目录
docker run -v 绝对路径:容器内路径
# 自动创建数据卷
docker run -v 卷名:容器内路径

# 查看数据卷
docker volume ls
# 查看数据卷的细节
docker volume inspect 卷名
# 创建数据卷
docker volume create 卷名
# 删除数据卷(没有使用的数据卷)
docker volume prune
# 删除指定的数据卷
docker volume rm 卷名
```

3.6、Docker 核心架构


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/9e53d301e0c547d39ecb22c04fc13685.png) 4、Docker安装服务

4.1、mysql 的安装

首先我们需要确定服务的版本，拉取镜像到本地： [dockerHub 拉取镜像描述文件](https://registry.hub.docker.com/_/mysql?tab=description)，在镜像的描述文件当中，会对服务的启动、查看服务日志、服务配置等等都有进行描述。

```java
# 先获取mysql服务
docker pull mysql:8.0.18
```

服务的启动：这里需要指定运行的环境

```java
# 基本启动
docker run -e MYSQL_ROOT_PASSWORD=root -d mysql:8.0.18

# 启动服务 后台运行 指定root用户账号密码(设置root账户的密码为root) 指定容器名称
docker run -d -p 3307:3306 --name mysql8.0 -e MYSQL_ROOT_PASSWORD=root -d mysql:8.0.18

# 启动服务 后台运行 指定root用户账号密码 指定容器名称 使用数据卷将数据持久化
# mysql 容器默认存储位置：/var/lib/mysql
docker run -d -p 3307:3306 --name mysql8.0 -e MYSQL_ROOT_PASSWORD=root -d  -v mysqldata:/var/lib/mysql mysql:8.0.18

# 启动服务 后台运行 指定root用户账号密码 指定容器名称 使用数据卷将数据持久化 已修改之后的配置文件启动
docker run -d -p 3307:3306 --name mysql8.0 -e MYSQL_ROOT_PASSWORD=root -d  -v mysqldata:/var/lib/mysql -v mysqlconfig:/etc/mysql mysql:8.0.18
```

4.2、Tomcat 的安装

```java
# 先获取镜像
docker pull tomcat:9.0-jdk8

# 服务启动
docker run -d -p 8080:8080 --name tomcat tomcat:9.0-jdk8

# 项目的部署目录 /usr/local/tomcat/webapps
docker run -d -p 8080:8080 -v apps:/usr/local/tomcat/webapps --name tomcat tomcat:9.0-jdk8

# 配置文件目录 /usr/local/tomcat/conf
docker run -d -p 8080:8080 -v apps:/usr/local/tomcat/webapps -v confs:/usr/local/tomcat/conf --name tomcat tomcat:9.0-jdk8
```

4.3、Redis的安装

```java
# 拉取镜像
docker pull redis:6.2.6

# 启动服务
docker run -d -p 6379:6379 --name redis6 redis:6.2.6

# redis 持久化
docker run -d --name redis6 redis:6.2.6 redis-server --appendonly yes
```

4.4、ElasticSearch 的安装

5、Dockerfile

5.1、Dockerfile 概述

**5.1.1、Dockerfile是什么？**

Dockerfile是用来帮助自己构建一个自定义镜像

**5.1.2、为什么会存在Dockerfile？**

日常用户可以将自己应用进行打包成镜像，这样就可以让我们自己的应用在容器当中运行

**5.1.3、Dockerfile构建镜像原理**

5.2、Dockerfile 语法


FROM：构建一个自定义的镜像

```java
# 新建一个dockerfile文件
vim Dockerfile
# 写入内容
FROM centos:8
# 进行build
docker build -t mycentos8:01 .

# RUN : 对镜像进行扩展
docker run -it centos:7
# 不支持vim，对于vim的扩展，在原本的dockerfile文件当中加入
RUN yum install -y vim
# 或者使用这种语法
RUN ["yum","install","-y","vim"]

# EXPORT : 镜像暴露端口
EXPOSE 8888

#指定工作目录
WORKDIR /data

# 复制文件
COPY aa.txt /data/aa

# 添加内容
ADD bb.txt /data/bb
ADD 下载地址 /data/tomcat
```

5.3、idea对Dockerfile支持

打开idea的settings，在以来当中找到docker的依赖，进行安装该依赖，安装之后重启idea就可以对Dockerfile文件进行编辑了。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/0d661e36b5004fcc8ba1c0125e9e0e8e.png) 第二个就是，Dockerfile文件都在linux上，在idea当中怎么进行编辑呢，可以选择Tools下的deployment的browse remote host 进行连接远程虚拟机，这里直接连接上去之后在右侧就会有虚拟机上的文件目录信息，并且可以直接在idea当中进行打开了。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/dccbe4f1dec946758f16ab24d2d65d5b.png)

6、Docker compose

6.1、Docker compose 概述

**6.1.1、compose的作用**

用来负责对Docker容器集群的快速编排

**6.1.2、compose的定位**

是用来定义和运行多个docker容器的应用 同时可以对多个容器进行编排

**6.1.3、compose的核心概念**

- 服务：一个应用的容器，服务可以存在多个 
- 项目：由一组关联的应用容器组成的一个完整业务单元，在docker-compose.yml文件当中进行定义

**6.1.4、compose的安装**

github下载地址： [https://github.com/docker/compose/releases](https://github.com/docker/compose/releases)

首先在github上面下载对应的版本包，下载之后将包上传到linux服务器上。将文件进行重命名并且复制到/usr/local/bin目录下，并且给该目录赋予权限。最后直接使用docker-compose -v命令查看版本进行校验是否安装成功

```java
mv docker-compose-linux-x86_64 /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose -v
```

**6.1.5、docker和docker-compose直接的版本对应**

docker官网地址： [https://docs.docker.com/compose/compose-file/](https://docs.docker.com/compose/compose-file/)

使用命令 docker -v 查看docker的版本，可以在官网当中看到compose对docker版本的支持


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/5776b11dbdbb4c37ad862c1eed65b806.png) 6.2、Docker compose —— HelloWorld

在前面有说到compose当中的组成部分，分别是服务和项目，这里首先创建一个目录用来表示这第一个helloworld项目，在项目当中添加docker-compose.yml文件用来编写compose。

```java
# 创建目录
mkdir hello
cd hello

# 新建docker-compose.yml文件
vim docker-compose.yml

# 写入内容（这里在vim当中编辑yml文件挺难受的，可以在idea当中编辑远程主机的文件）
version: "3.0"    # 指定compose的版本
services:         # 指定服务
  tomcat:         # 单个服务
    image: tomcat:9.0.27-jdk8     # 服务镜像
    ports:
      - 8081:8080                 # 暴露对应的端口

# 保持文件内容后进行启动compose
docker-compose up

# 服务启动之后，可以直接进行访问8081端口
http://远程主机ip/8081
```

6.3、Docker compose —— 命令模板

```java
version: "3.0"		# compose版本
services:
  user:
    build:
      context: user					# dockerfile的镜像
      dockerfile: Dockerfile		# 读取dockerfile文件进行打包获取镜像
    container_name: user
    ports:
      - "8888:8888"
    networks:
      - hello
    depends_on:
      - tomcat

  tomcat:			# 单个服务标识
    container_name: tomcat			# 启动后的容器名称 相当于 --name 指定的名称
    image: tomcat:9.0.27-jdk8		# 镜像
    ports:
      - 8081:8080					# 端口映射
    volumes:
      - tomcatwebapps:/usr/local/tomcat/webapps		# 指定对应的数据卷
    networks:
      - hello										# 指定网桥
    depends_on:										# 服务启动依赖
      - tomcat1										# 服务标识
      - mysql
    healthcheck:									# 健康检查
      test: ['CMD','curl','-f','http://localhost']
      interval: 1m30s
      timeout: 10s
      retries: 3
    sysctls:										# 修改内核参数
      - net.core.somaxconn=1024
      - net.ipv4.tcp_syncookies=0
	ulimits:										# 修改最大进程数
      nproc: 65335
      nofile:
        soft: 20000
        hard: 40000
        
  tomcat1:
    container_name: tomcat2
    image: tomcat:9.0.27-jdk8
    ports:
    - 8082:8080
    volumes:
    - tomcatwebapps1:/usr/local/tomcat/webapps
    networks:
    - hello

  mysql:
    container_name: mysql8
    image: mysql:8.0.18
    ports:
      - 3307:3306
#    environment:							# 指定启动的环境
#      - MYSQL_ROOT_PASSWORD=root
    env_file:								# 使用文件进行代替
      - ./mysql.env							# mysql.evn文件内容就是 MYSQL_ROOT_PASSWORD=root
    volumes:
      - mysqldata:/var/lib/mysql
      - mysqlconfig:/etc/mysql
    networks:
      - hello

# 数据卷都要在这统计管理
volumes:
  tomcatwebapps:
  tomcatwebapps1:
#    external:		# 使用自定义数据卷名称 默认命名为 项目名_数据卷名  自定义后为 数据卷名
#      true
  mysqldata:
  mysqlconfig:

# 统一管理网桥
networks:
  hello:
```

6.4、Docker compose 指令

**6.4.1、模板指令与指令**

- 模板指令：用来书写在docker-compose.yml文件当中的指令，是用来为服务进行服务的 
- 指令：用来对整个docker-compose.yml对应的这个项目进行操作

**6.4.2、常用指令**


6.5、Docker 可视化工具 —— portainer

直接在dockerHub上面拉取镜像启动服务

```java
docker pull portainer/portainer:1.24.2

docker run -d 
	-p 8000:8000 
	-p 9000:9000 
	--name=portainer 
	--restart=always 
	-v /var/run/docker.sock:/var/run/docker.sock 
	-v portainer_data:/data 
	portainer/portainer:1.24.2
```

直接访问远程虚拟机的9000端口，注册一个账号，链接到本地虚拟机的服务，就可以看到所提供的web可视化页面了。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/19afb828408f4d589efa1ff754c86c88.png) 同样的我们还可以将这个服务的启动加到docker-componse当中进行启动：

```java
# 加入服务
  portainer:
    container_name: portainer
    image: portainer/portainer:1.24.2
    ports:
      - 8000:8000
      - 9000:9000
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
      
# 数据卷
volumes:
  portainer_data:
```



---

> 作者: wang  
> URL: https://codingroam.github.io/post/docker%E4%B8%8Edocker-compose%E8%AF%A6%E8%A7%A3/  

