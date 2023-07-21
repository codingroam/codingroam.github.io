# docker详解(尚硅谷阳哥)

docker详解(尚硅谷阳哥)
<!--more-->


一款产品从开发到上线，一般都会有开发环境，测试环境，运行环境。

如果有一个环境中某个软件或者依赖版本不同了，可能产品就会出现一些错误，甚至无法运行。比如开发人员在windows系统，但是最终要把项目部署到linux。如果存在不支持跨平台的软件，那项目肯定也无法部署成功。

这就产生了开发和运维人员之间的矛盾。开发人员在开发环境将代码跑通，但是到了上线的时候就崩了。还要重新检查操作系统，软件，依赖等版本，这大大降低了效率。造成了搭环境一两天，部署项目两分钟的事件。

docker的出现就能解决以上问题：

开发人员把环境配置好，将需要运行的程序包运行成功，然后把程序包和环境一起打包给运维人员，让运维人员部署就可以了。这大大提高了项目上线的效率。


Docker是基于Go语言实现的云开源项目。

Docker的主要目标是“Build，Ship and Run Any App,Anywhere”，也就是通过对应用组件的封装、分发、部署、运行等生命周期的管理，使用户的APP（可以是一个WEB应用或数据库应用等等）及其运行环境能够做到“一次镜像，处处运行”


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/b3ffe73fea1d51a52849159c521b3e84.png)

Linux容器技术的出现就解决了这样一个问题，而 Docker 就是在它的基础上发展过来的。将应用打成镜像，通过镜像成为运行在Docker容器上面的实例，而 Docker容器在任何操作系统上都是一致的，这就实现了跨平台、跨服务器。只需要一次配置好环境，换到别的机器上就可以一键部署好，大大简化了操作。

docker简介总结：

解决了运行环境和配置问题的软件容器，方便做持续集成并有助于整体发布的容器虚拟化技术。docker基于Linux内核，仅包含业务运行所需的runtime环境。


## 3.1虚拟机

虚拟机是可以在一种操作系统里面运行另一种操作系统，比如在Windows10系统里面运行Linux系统CentOS7。应用程序对此毫无感知，因为虚拟机看上去跟真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通文件。这类虚拟机完美的运行了另一套系统，能够使应用程序，操作系统和硬件三者之间的逻辑不变。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/8fe6bbaca83618a20aafb851cc5d5675.png)

虚拟机的缺点：

1. 启动慢 
2. 资源占用多 
3. 冗余步骤多

## 3.2容器虚拟化技术

由于前面虚拟机存在某些缺点，Linux发展出了另一种虚拟化技术：

Linux容器(Linux Containers，缩写为 LXC)

Linux容器是与系统其他部分隔离开的一系列进程，从另一个镜像运行，并由该镜像提供支持进程所需的全部文件。容器提供的镜像包含了应用的所有依赖项，因而在从开发到测试再到生产的整个过程中，它都具有可移植性和一致性。

Linux 容器不是模拟一个完整的操作系统而是对进程进行隔离。有了容器，就可以将软件运行所需的所有资源打包到一个隔离的容器中。容器与虚拟机不同，不需要捆绑一整套操作系统，只需要软件工作所需的库资源和设置。系统因此而变得高效轻量并保证部署在任何环境中的软件都能始终如一地运行。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/312a9a474a48c47162ee6ccf1bccbf34.png)=

## 3.3两者对比

-  传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；  
-  容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。  
-  每个容器之间互相隔离，每个容器有自己的文件系统 ，容器之间进程不会相互影响，能区分计算资源。 


一：更快速的应用交付和部署

传统的应用开发完成后，需要提供一堆安装程序和配置说明文档，安装部署后需根据配置文档进行繁杂的配置才能正常运行。Docker化之后只需要交付少量容器镜像文件，在正式生产环境加载镜像并运行即可，应用安装配置在镜像里已经内置好，大大节省部署配置和测试验证时间。

二：更便捷的升级和扩缩容

随着微服务架构和Docker的发展，大量的应用会通过微服务方式架构，应用的开发构建将变成搭乐高积木一样，每个Docker容器将变成一块“积木”，应用的升级将变得非常容易。当现有的容器不足以支撑业务处理时，可通过镜像运行新的容器进行快速扩容，使应用系统的扩容从原先的天级变成分钟级甚至秒级。

三：更简单的系统运维

应用容器化运行后，生产环境运行的应用可与开发、测试环境的应用高度一致，容器会将应用程序相关的环境和状态完全封装起来，不会因为底层基础架构和操作系统的不一致性给应用带来影响，产生新的BUG。当出现程序异常时，也可以通过测试环境的相同容器进行快速定位和修复。

四：更高效的计算资源利用

Docker是内核级虚拟化，其不像传统的虚拟化技术一样需要额外的Hypervisor支持，所以在一台物理机上可以运行很多个容器实例，可大大提升物理服务器的CPU和内存的利用率。


docker借鉴了标准集装箱的概念。标准集装箱是将货物运往世界各地，docker将这个模型运用到自己的设计当中，唯一不同的是：集装箱运送货物，而docker运输软件。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/61fce447bcfb25ba31b7e66ca97abddd.png)



![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/0588aa993350cb0b1a1bde81b9facbf2.png)

**一：镜像（Image）**

Docker 镜像（Image）就是一个只读的模板。镜像可以用来创建 Docker 容器，一个镜像可以创建很多容器。

**二：容器（Container）**

Docker 利用容器（Container）独立运行的一个或一组应用。容器是用镜像创建的运行实例。

它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。

可以把容器看做是一个简易版的 Linux 环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

容器的定义和镜像几乎一模一样，也是一堆层的统一视角，唯一区别在于容器的最上面那一层是可读可写的。

**三：仓库（Repository）**

仓库（Repository）是集中存放镜像文件的场所。仓库(Repository)和仓库注册服务器（Registry）是有区别的。仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）。


仓库分为公开仓库（Public）和私有仓库（Private）两种形式。最大的公开仓库是 Docker Hub(https://hub.docker.com/)，存放了数量庞大的镜像供用户下载。国内的公开仓库包括阿里云 、网易云 等 ![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/6c60df3b853c559c2bd2a9ae3bcb83ed.png)


Docker是一个Client-Server结构的系统，Docker守护进程运行在主机上， 然后通过Socket连接从客户端访问，守护进程从客户端接受命令并管理运行在主机上的容器。 容器，是一个运行时环境，就是我们前面说到的集装箱。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/212040299389d11971fdbfaa0d1c5214.png)


Docker 是一个 C/S 模式的架构，后端是一个松耦合架构，众多模块各司其职。

Docker运行的基本流程为:

- 用户是使用Docker Client与Docker Daemon建立通信，并发送请求给后者。 
- Docker Daemon作为Docker架构中的主体部分，首先提供Docker Server的功能使其可以接受Docker Client的请求。 
- Docker Engine执行Docker内部的一系列工作，每一项工作都是以一个Job的形式的存在。 
- Job的运行过程中，当需要容器镜像时，则从Docker Registry中下载镜像，并通过镜像管理驱动Graph driver将下载镜像以Graph的形式存储。 
- 当需要为Docker创建网络环境时，通过网络管理驱动Network driver创建并配置Docker容器网络环境。 
- 当需要限制Docker容器运行资源或执行用户指令等操作时，则通过Exec driver来完成。 
- Libcontainer是一项独立的容器管理包，Network driver以及Exec driver都是通过Libcontainer来实现具体对容器进行的操作。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/0314c55d593a4aa2c65ed7c24217b18f.png)


 [Docker官网](https://www.docker.com/)

Docker并非是一个通用的容器工具,它依赖于已存在并运行的Linux内核环境。

Docker实质上是在已经运行的Linux下制造了一个隔离的文件环境，因此它执行的效率几乎等同于所部署的Linux主机。

因此，Docker必须部署在Linux内核的系统上。如果其他系统想部署Docker就必须安装一个虚拟 Linux环境。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/542c68749bfdb5ecbf93788e4898db2c.png)

要求系统为64位、Linux系统内核版本为 3.8以上

查看自己虚拟机的内核：


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/be4a1bf7751f9dd3f775fa9dfa576f47.png)

开始安装：

**一：搭建gcc环境（gcc是编程语言译器）**

```java
yum -y install gcc
yum -y install gcc-c++
```

**二：安装需要的软件包**

```java
yum install -y yum-utils
```

**三：安装镜像仓库**

官网上的是


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/17906968f4763af1f86068c6b9c03b0e.png)

但是因为docker的服务器是在国外，所以有时候从仓库中下载镜像的时候会连接被拒绝或者连接超时的情况，所以可以使用阿里云镜像仓库

```java
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

**四：更新yum软件包索引**

```java
yum makecache fast
```

**五：安装docker引擎**

```java
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

**六：启动docker**

```java
systemctl start docker
```

查看docker服务


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/eb819c1bd181e7603d58752999d5551e.png)

查看docker版本信息

```java
docker version
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/1a8fc59d4d3d3eabc84b06480bec09ca.png)

```java
docker run hello-world
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/15277542dc58d1813347098367093586.png)


为了提高镜像的拉取、发布的速度，可以配置阿里云镜像加速


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/659545bb15fe5db8e045233c63e910fd.png)

查看加速器地址


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/325f98d2df03929449ea3ac44b4224e8.png)

在CentOS下配置镜像加速器


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/de6f5b048d1afa84eb98b7ba936b5915.png)

```java
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
   
  "registry-mirrors": ["https://8pfzlx7j.mirror.aliyuncs.com"]
}
EOF
systemctl daemon-reload
systemctl restart docker
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/79922a968ed85c45780481f4bfedaac1.png)

输出这段提示以后，hello world就会停止运行，容器自动终止。

run干了什么？


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/fd0e1dd69f1d17c6fb8a07db814293f6.png)


**(1)docker有着比虚拟机更少的抽象层**

由于docker不需要Hypervisor(虚拟机监视器)实现硬件资源虚拟化,运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。因此在CPU、内存利用率上docker将会在效率上有明显优势。

**(2)docker利用的是宿主机的内核,而不需要加载操作系统OS内核**

当新建一个容器时,docker不需要和虚拟机一样重新加载一个操作系统内核。进而避免引寻、加载操作系统内核返回等比较费时费资源的过程,当新建一个虚拟机时,虚拟机软件需要加载OS,返回新建过程是分钟级别的。而docker由于直接利用宿主机的操作系统,则省略了返回过程,因此新建一个docker容器只需要几秒钟。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/13087e15abc9ce56a7958938d9d0eb5b.png)


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/30b23baff24fe74b68df79756a9cb4f2.png)


## 13.1帮助启动类命令

-  
 <blockquote> 
  <p>启动docker： <code>systemctl start docker</code></p> 
 </blockquote> 

-  
 <blockquote> 
  <p>停止docker： <code>systemctl stop docker</code></p> 
 </blockquote>  
-  
 <blockquote> 
  <p>重启docker： <code>systemctl restart docker</code></p> 
 </blockquote>  
-  
 <blockquote> 
  <p>查看docker状态： <code>systemctl status docker</code></p> 
 </blockquote>  
-  
 <blockquote> 
  <p>开机启动： <code>systemctl enable docker</code></p> 
 </blockquote>  
-  
 <blockquote> 
  <p>查看docker概要信息： <code>docker info</code></p> 
 </blockquote>  
-  
 <blockquote> 
  <p>查看docker总体帮助文档： <code>docker --help</code></p> 
 </blockquote>  
-  
 <blockquote> 
  <p>查看docker命令帮助文档：<code> docker 具体命令 --help</code></p> 
 </blockquote> 

## 13.2镜像命令


-  
 <blockquote> 
  <p>列出主机上的所有镜像：<code>docker images</code></p> 
 </blockquote> 


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/31ef2ca7eb4f1771baae8f6baae62a62.png)

```java
各个选项说明:
REPOSITORY：表示镜像的仓库源
TAG：镜像的标签版本号
IMAGE ID：镜像ID
CREATED：镜像创建时间
SIZE：镜像大小
 同一仓库源可以有多个 TAG版本，代表这个仓库源的不同个版本，我们使用 REPOSITORY:TAG 来定义不同的镜像。
如果你不指定一个镜像的版本标签，例如你只使用 ubuntu，docker 将默认使用 ubuntu:latest 镜像
OPTIONS说明：
-a :列出本地所有的镜像（含历史映像层）
-q :只显示镜像ID。
```

-  
 <blockquote> 
  <p>搜索某个镜像： search 镜像名`</p> 
 </blockquote> 


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/43b627348d05f1998c0a189bf017c8d6.png)

```java
各个选项说明:
NAME：镜像名称
DESCRIPTION：镜像说明
STARS：点赞数量
OFFICAL：是否是官方的
AUTOMATED：是否是自动构建的
OPTIONS说明：
--limit : 只列出N个镜像，默认25个
docker search --limit 5 redis
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/967110448afb8a9593e6743fb2989c8d.png)

-  
 <blockquote> 
  <p>下载镜像：</p> 
  <p><code>docker pull 镜像名</code> 不指定版本，默认是最新版本</p> 
  <p><code>docker pull 镜像名 [:TAG] </code> 指定镜像版本</p> 
 </blockquote> 


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/92090a67aa7ec73e03f846d0de56f585.png)


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/09ba7c1222f57b0563ddee3db8bc9f07.png)

-  
 <blockquote> 
  <p>查看镜像/容器/数据卷所占的空间 <code>docker system df </code></p> 
 </blockquote> 

```java
各个选项说明:
TYPE：类型（镜像、容器、数据卷）
TOTAL：总数
SIZE：大小
RECLAIMABLE：伸缩性
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/ff5af730b7500262878b9fd37938c7d1.png)

-  
 <blockquote> 
  <p>删除镜像</p> 
  <p><code>docker rmi 镜像名/镜像ID</code></p> 
  <p>强制删除镜像</p> 
  <p><code>docker rmi -f 镜像名/镜像ID</code></p> 
  <p>删除多个镜像</p> 
  <p><code>docker rmi -f 镜像名1:TAG 镜像名2:TAG </code></p> 
  <p>删除docker引擎中的全部镜像</p> 
  <p><code>docker rmi -f $(docker images -qa)</code></p> 
 </blockquote> 


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/d8818f6c8d770910b5d16e9eaf0a8aba.png)

删除带版本的镜像


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/49766d4b79c0fae7339054ce1a7af2a9.png)

-  
 <blockquote> 
  <p>查看私有仓库指定镜像所有版本</p> 
  <p><code>curl -XGET http://私有仓库主机ip:端口/v2/镜像名称/tags/list</code></p> 
 </blockquote> 


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/e90583e92aa249f26683545231c81bda.png)

面试题：谈谈docker虚悬镜像是什么？

>仓库名、标签都是的镜像，俗称虚悬镜像dangling image


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/3554ec919265503fd07dab8619ef32be.png)

## 13.3容器命令


docker命令中/bin/bash的作用是：

docker中必须要保持一个进程的运行，要不然整个容器启动后就会马上kill itself，这个/bin/bash就表示启动容器后启动bash。

有镜像才能创建容器， 这是根本前提(下载一个CentOS或者ubuntu镜像演示)

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-2VIyzu0k-1667213295628)(Docker.assets/1658129060820-e3bbad90-56b9-44d7-971b-305557ab2ed9.png)]


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/0c825f5fd428ee0d6a8ca8df9a6c45fc.png)

-  
 <blockquote> 
  <p>新建/启动容器</p> 
  <p><code>docker run [OPTIONS] IMAGE [COMMAND] [ARG...]</code></p> 
  <p>启动交互式容器</p> 
  <p><code>docker run -it IMAGE [COMMAND] [ARG...] </code></p> 
  <p>新建指定名字的容器</p> 
  <p><code>docker run --name=容器名 IMAGE [COMMAND] [ARG...]</code></p> 
  <p>为容器开启守护进程</p> 
  <p><code>docker run -d IMAGE [COMMAND] [ARG...]</code></p> 
 </blockquote> 

```java
OPTIONS说明（常用）：有些是一个减号，有些是两个减号
--name="容器新名字"：为容器指定一个名称；
-d: 后台运行容器并返回容器ID，也即启动守护式容器(后台运行)；
-i：以交互模式运行容器，通常与 -t 同时使用；
-t：为容器重新分配一个伪输入终端，通常与 -i同时使用；也即启动交互式容器(前台有伪终端，等待交互)；
-P: 随机端口映射，大写P
-p: 指定端口映射，小写p
```

**启动交互式容器**（-it表示启动带命令行交互的伪终端）


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/e2352c5d57599776621d7524573c75b9.png)

**新建指定的名字的容器**


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/51b0ad5917b07c7cb278d825b3753279.png)


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/9c83f209c7501884483dbf94de082a72.png)

**启动守护式容器(让容器在后台运行)**

在大部分的场景下，我们希望 docker 的服务是在后台运行的， 我们可以过 -d 指定容器的后台运行模式

在后台开启一个容器后，再查询当前运行中的容器，发现并没有。

很重要的要说明的一点: Docker容器后台运行,就必须有一个前台进程。

容器运行的命令如果不是那些一直挂起的命令（比如运行top，tail），就是会自动退出的。

这个是docker的机制问题,比如你的web容器,我们以nginx为例，正常情况下,我们配置启动服务只需要启动响应的service即可。例如service nginx start。但是,这样做,nginx为后台进程模式运行,就导致docker前台没有运行的应用，这样的容器后台启动后，会立即自杀因为他觉得他没事可做了。所以，最佳的解决方案是将你要运行的程序以前台进程的形式运行，常见就是命令行模式，表示我还有交互操作，别中断.

前台启动redis服务


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/6983c037669b1167c425d9cf36e4a002.png)

后台启动redis服务


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/3c43ae3fc6a2e73ffc89549ae6fa7ae9.png)

-  
 <blockquote> 
  <p>列出当前正在运行的所有容器：<code>docker ps [OPTIONS]</code></p> 
 </blockquote> 

```java
OPTIONS说明（常用）：
-a :列出当前所有正在运行的容器+历史上运行过的
-l :显示最近创建的容器。
-n：显示最近n个创建的容器。
-q :静默模式，只显示容器编号。
```

创建一个新的虚拟机实例，查看当前容器的运行情况


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/d15a62ff0ca288aa6bb7746f1e19ffaf.png)


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/d1d857c2790dad8fe14a00b7d7ce676d.png)

-  
 <blockquote> 
  <p>退出容器，容器停止：</p> 
  <p><code>exit</code></p> 
  <p>退出容器，容器不停止</p> 
  <p><code>快捷键ctrl+p+q</code></p> 
 </blockquote>  
-  
 <blockquote> 
  <p>启动已停止运行的容器：<code>docker start 容器ID或者容器名</code></p> 
 </blockquote>  
-  
 <blockquote> 
  <p>重启容器：<code>docker restart 容器ID或者容器名</code></p> 
 </blockquote>  
-  
 <blockquote> 
  <p>停止容器：<code>docker stop 容器ID或者容器名</code></p> 
 </blockquote>  
-  
 <blockquote> 
  <p>强制停止容器：<code>docker kill 容器ID或容器名</code></p> 
 </blockquote>  
-  
 <blockquote> 
  <p>删除已停止的容器：</p> 
  <p><code>docker rm 容器ID</code></p> 
  <p>强制删除正在运行的容器</p> 
  <p><code>docker rm -f 容器名/容器ID</code></p> 
  <p>一次性删除多个容器实例</p> 
  <p><code>docker rm -f $(docker ps -a -q)</code></p> 
  <p><code>docker ps -a -q | xargs docker rm</code></p> 
 </blockquote>  
-  
 <blockquote> 
  <p>查看容器日志：<code>docker logs 容器ID</code></p> 
 </blockquote> 

查看redis的日志信息


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/afd2cdb2741945ac5bf1deb13f6bf226.png)

-  
 <blockquote> 
  <p>查看容器内运行的进程：<code>docker top 容器ID</code></p> 
 </blockquote> 

可以看到redis容器中当前只有一个进程：redis-sercer


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/67f1ab864e6730d3e905eb7eaab92f9b.png)

-  
 <blockquote> 
  <p>查看容器内部的细节：<code>docker inspect 容器ID</code></p> 
 </blockquote> 


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/fe8d625579afe74035920abff8308e80.png)

-  
 <blockquote> 
  <p>进入正在运行的容器并以命令行交互：</p> 
  <p>方式1（推荐方式）：</p> 
  <p><code>docker exec -it 容器ID bashShell</code></p> 
  <p>方式2：</p> 
  <p><code>docker attach 容器ID</code></p> 
 </blockquote> 

方式1：


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/4ba31dbaeb6c9650f5363b23c87507ab.png)

方式2：


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/47f75ad48352adb555a87f856e69bbeb.png)

-  
 <blockquote> 
  <p>从容器内的文件拷贝到主机上：<code>docker cp 容器ID:容器路径 主机路径</code></p> 
 </blockquote> 

将容器下/opt目录下的test.txt文件cp到主机上


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/5ec26f4cb07c698ec9e0fbebfd70fc4c.png)


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/8682f16522753c846f9c9aa2698a0fae.png)

-  
 <blockquote> 
  <p>将容器以压缩包的形式导出到当前路径下：<code>docker export 容器ID &gt; 压缩文件名.tar</code></p> 
 </blockquote> 


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/ddee1edae198885e8b8152f504037725.png)

-  
 <blockquote> 
  <p>从tar包中的内容创建一个新的文件系统再导入为镜像：</p> 
  <p><code>cat 文件名.tar | docker import - 镜像用户/镜像名:镜像版本</code></p> 
 </blockquote> 


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/c3cf559178db7a3b4d2cb3f33d45a41d.png)


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/feba8d9907c02e47366c5e7b3096e72e.png)

使用刚刚解压的镜像创建一个容器，查看容器中是否有test.txt文件。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/261c8f14f86b9a4bb9339296244a8e50.png)


## 14.1镜像的概念

是一种轻量级、可执行的独立软件包，它包含运行某个软件所需的所有内容，我们把应用程序和配置依赖打包好形成一个可交付的运行环境(包括代码、运行时需要的库、环境变量和配置文件等)，这个打包好的运行环境就是image镜像文件。

只有通过这个镜像文件才能生成Docker容器实例(类似Java中new出来一个对象)。

## 14.2分层的镜像

以我们的pull为例，在下载的过程中我们可以看到docker的镜像是在一层一层的下载


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/24cdf8cd559e2f7c087759b1401e1d9a.png)

## 14.3镜像的底层原理(联合文件系统)

UnionFS（联合文件系统）：Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录

## 14.4镜像加载原理

docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS(联合文件系统)。

bootfs(引导文件系统)主要包含bootloader和kernel(Linux内核),bootloader主要是引导加载kernel, Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是引导文件系统bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。

rootfs (root file system) ，在bootfs之上。包含的就是典型 Linux 系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu，Centos等等。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/ef59d530a69abeb3bec1ec080dbdeb7d.png)

平时我们安装进虚拟机的CentOS都是好几个G，为什么docker这里才200M？？


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/c402673c7fa9e2b99c9082af97c7b10a.png)

对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接用Host的kernel，自己只需要提供 rootfs 就行了。由此可见对于不同的linux发行版, bootfs基本是一致的, rootfs会有差别, 因此不同的发行版可以公用bootfs。

## 14.5为什么docker镜像使用分层结构

镜像分层最大的一个好处就是共享资源，方便复制迁移，就是为了复用。

比如说有多个镜像都从相同的 base 镜像构建而来，那么 Docker Host 只需在磁盘上保存一份 base 镜像；

同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。

Docker镜像层都是只读的，容器层是可写的。当容器启动时，一个新的可写层被加载到镜像的顶部。 这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。

所有对容器的改动 - 无论添加、删除、还是修改文件都只会发生在容器层中。只有容器层是可写的，容器层下面的所有镜像层都是只读的。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/927f76246a80bc326ca2354fe0d4259c.png)

Docker中的镜像分层，支持通过扩展现有镜像，创建新的镜像。类似Java继承于一个Base基础类，自己再按需扩展。

新镜像是从 base 镜像一层一层叠加生成的。每安装一个软件，就在现有镜像的基础上增加一层


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/1747d50e2329174242ec594e719b7566.png)

## 14.6docker镜像commit操作案例

-  
 <blockquote> 
  <p>docker commit 提交容器副本使之成为一个新的镜像：</p> 
  <p><code>docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]</code></p> 
 </blockquote> 

测试：在tomcat安装vim


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/b9f1601a52266e0e0fc61aa974974adc.png)

安装vim

```java
apt-get update
apt-get -y install vim
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/95553d094d1dc3b139c2ea67a65b407c.png)


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/de7310ad9d635f1d7fb09160b98674a2.png)

## 14.7在阿里云推送/拉取镜像


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/6bfec0889798a20c65908e077824ff9c.png)


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/7733b3335b52f0c25beec7a2c457b44a.png)


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/e6bfd554e4963a253322516018500693.png)


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/15c17b56043e58f8797aca1cfb840a7a.png)

阿里云镜像仓库提供的对应的命令：


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/a719e1162024ef8e6dd8091a18c8e5cb.png)


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/b1e5ec85c036c4b3d8b4ca6cd6a5669d.png)


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/708a5f6c77ae3c7d81806ef03f142ec8.png)

测试：

删除本地镜像，从阿里云仓库中拉取


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/98081f9cfe5263fd5dac0a894bc90997.png)

## 14.8阿里云私有库

Dockerhub、阿里云这样的公共镜像仓库可能不太方便，涉及机密的公司不可能提供镜像给公网，所以需要创建一个本地私人仓库供给团队使用，基于公司内部项目构建镜像。

Docker Registry是官方提供的工具，可以用于构建私有镜像仓库

## 14.9创建私有仓库

**一：下载镜像docker registry**

```java
docker pull registry
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/17a9b292693041ea8e2c7e834898789d.png)

**二：运行私有库Registry**

```java
docker run -d -p 5000:5000  -v /docker-xha/tomcat-vim/:/opt/registry --privileged=true registry
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/659a794f34c9257061183d8e7008e4e4.png)

**三：创建一个新镜像，tomcat安装ifconfig命令**


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/4a9cca2beaa59dc4a1512c6d389dd931.png)

```java
docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]
docker commit -m="ifconfig cmd add" -a="xha" 749a08193a53 tomcat-ifconfig:1.1
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/d9b9f7db1dcefff09a7fc0d746d3ff99.png)

已经提交的镜像：


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/b803f84df5ea4af69280157e4c2c83dd.png)

**四：验证提交的镜像能够使用ifconfig**


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/a4f7bbb4f464ded742fd9d7a8427c826.png)

**五：查看私服库上有什么镜像**

```java
curl -XGET http://主机ip:5000/v2/_catalog
```

当前私服库上没有任何镜像


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/522f99f167d68180015ae9bc4d2706f4.png)

**六：将新镜像修改符合私服规范的Tag**

```java
docker tag tomcat-ifconfig:1.1 192.168.26.135:5000/tomcat-ifconfig:1.1
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/51520d7b7c16ed4606491be5e21ba52a.png)

**七：修改进程守护配置文件，设置docker允许推动镜像**

```java
"insecure-registries": ["192.168.26.135:5000"]
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/13300de1f2c276086b421940bf1abc63.png)

重启docker服务


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/ee8b0f919bdfc563b52289d97133a911.png)

**八：重启私有仓库**

```java
docker run -d -p 5000:5000  -v /命名空间/仓库名称/:/opt/registry --privileged=true registry
```

**九：将镜像推送到私服库**


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/32673abbd5124e41357fe4ccfe06cd95.png)

**十：查看当前私服库中的镜像**

```java
curl -XGET http://主机ip:5000/v2/_catalog
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/d3fec939163a5b503d9fb98284bf68bb.png)

**十一：拉取私服库中的镜像**

```java
docker pull 主机ip:端口号/镜像名:版本号
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/f9852d8fcaaf28ede23b7fcd9af04002.png)

查看docker中的所有镜像


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/b2a2c55208654279150d019053f42f48.png)

利用此镜像新建容器，并测试ifconfig命令


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/e7125586ed62d806f73c57fb4d5e15dc.png)


## 15.1容器数据卷概述

卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于持续存储或共享数据的特性：数据卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷。

数据卷会将docker容器内的数据保存进宿主机的磁盘中，运行一个带有容器卷存储功能的容器实例。

## 15.2容器数据卷的特点

1. **数据卷可在容器之间共享或重用数据** 
2. **容器和宿主机之间数据共享** 
3. **卷中的更改可以直接实时生效** 
4. **数据卷中的更改不会包含在镜像的更新中** 
5. **数据卷的生命周期一直持续到没有容器使用它为止**

## 15.3为容器添加数据卷

```java
docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录 镜像名
```

>选项解释：


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/a559e7017d6b142275193fdf89b3faaa.png)

查看数据卷是否挂载成功：

```java
dokcer inspect 容器id
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/8c859e1045f4d1957f3ae893624561a2.png)

容器和宿主机之间数据共享：

在容器中编辑文件：


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/99592cf71b920289c6998b8cf0a92b3c.png)

在宿主机的同一路径下查看文件：


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/d06feff81d0397393dbc8f0853bbef77.png)

在宿主机中编辑文件：


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/924f29a3a74494c3f51b514b3c01e18b.png)

查看容器中查看文件：


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/7d44219d0ab79dc1daf4981bfe946dce.png)

实现了容器和宿主机之间的数据共享

即使容器停止了，在宿主机操作数据卷，等到容器重新启动了也能实现数据共享

## 15.4容器卷ro和rw的读写规则

一：ro即read only，容器只能读

```java
docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录:ro --name:容器名 镜像名
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/693cefdb0022347bd49ce9f224914445.png)

创建的实例：


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/1edcd04631afc5c1559ed8265fa4e0fa.png)

使用主机进行文件的修改和读取，并在容器中进行读写操作。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/a2ecc3d53676eb2006a0e12963d8e1e6.png)

## 15.5容器卷之间的继承

第一个容器1完成和宿主机的映射


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/7aed11373bf1c9d412a961ddbfd545fa.png)

容器2继承容器1的卷规则

```java
docker run -it  --privileged=true --volumes-from 父类  --name:容器名 镜像名
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/0922a89143ae5eb4e34e96fed66b8e55.png)


## 16.1Tomcat

说明：由于较新版的tomcat需要删除/usr/local/tomcat目录下的webapps文件，并将webapps.dist重命名为webapp。所以推荐下载旧版的tomcat。

步骤：

1.  docker hub上面查找tomcat镜像（这里采用8.0.53版本）  
2.  从docker hub上拉取tomcat镜像到本地  
3.  docker images查看是否有拉取到的tomcat  
4.  使用tomcat镜像创建容器实例 

报错就重启一下docker服务

```java
docker run -d -p 8080:8080 --name t1 tomcat:8.0.53
```

tomcat服务启动成功


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/43d9e3fba58ab0d630151cbdf5224d4f.png)

## 16.2MySQL

**一：简单版本**

1. docker hub上面查找mysql镜像(这里采用8.0.19版本) 
2. 使用mysql8.0.19镜像创建容器(也叫运行镜像)

```java
docker run -p 主机端口号:容器端口 -e MYSQL_ROOT_PASSWORD=密码 -d mysql:版本
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/b85f9e0e6c437e6b5c72dd275d02310a.png)

1. 进入mysql


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/56347af4c935aeda5ea7ae69584eee23.png)

4.测试：查看数据库，创建数据库


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/54625390a8b471be93d7a15e001683b5.png)

6.查看docker容器下mysql的字符集编码

```java
show variables like 'character%'
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/188a6239cb2f4dc42de10ed659d88af5.png)

5.在本机采用可视化工具连接测试


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/c9424e6e75f424578da531f1f9590afc.png)

**二：实战版本**

1. 创建mysql容器实例

 为了防止数据丢失和误删问题，采用数据卷的形式实现数据备份：

```java
docker run -d -p 3306:3306 --privileged=true \
-v /opt/mysql/log:/var/log/mysql \
-v /opt/mysql/data:/var/lib/mysql \
-v /opt/mysql/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=xu.123456 \
--name mysql \
mysql:8.0.19
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/cef750218ed5e0890002833c543695ec.png)

1. 在主机的配置文件目录下新建mysql的配置文件my.cnf


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/f7835e80cad00c2f460256e50e54020a.png)

设置mysql字符编码为utf-8

```java
[client]
default_character_set=utf8
[mysqld]
collation_server = utf8_general_ci
character_set_server = utf8
```

1. 重启mysql容器实例

```java
docker restart 容器ID
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/d17969708df9e7902950f0cf9c69ed12.png)

1.  重新启动容器，重新查看mysql的字符集编码 show variables like 'character%' 


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/13056daee8f55d072a5920c5a3d4ca2e.png)

## 16.3Redis

1. 拉取redis镜像


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/afb772af75e0a7834d07b93d7a2c2850.png)

1. 获取redis的配置文件

 redis配置文件官网 [Redis configuration | Redis](https://redis.io/docs/manual/config/)

1. 创建存放redis配置文件的目录，创建redis.conf文件，写入配置信息


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/8a5fccbaae841518f9b1104cae3cff12.png)

1.  修改配置文件内容 
 <ul> 
  1. 添加redis密码（requirepass） 
  1. 修改bind为0.0.0.0（任何机器都能够访问） 
  1. 为了避免和docker中的-d参数冲突，将后台启动设置为no（daemonize no） 
  1. 关闭保护模式(protected-mode no) 
  1. 开启AOF持久化(appendonly yes) 
 </ul>  
7.  使用redis镜像创建容器实例 docker run -d -p 6379:6379 \
--name redis --privileged=true \
-v /opt/redis/redis.conf:/etc/redis/redis.conf \
-v /opt/redis/data:/data \
redis:6.0.8   
8.  进入容器，并启动redis客户端 


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/7d17faf73d07ecf653ade6827386a165.png)

1. 退出容器，redis容器依旧运行。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/08261447c8e31aa10fd715b858b1257a.png)


1. 首先安装好MySQL镜像 
2. 创建主节点MySQL实例对象，主机端口号为3307，容器内端口号为3306

```java
docker run -p 3307:3306 \
--name mysql-master \
-v /opt/mysql-master/log:/var/log/mysql \
-v /opt/mysql-master/data:/var/lib/mysql \
-v /opt/mysql-master/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=root  \
-d mysql:8.0.19
```

1. 进入/opt/mysql-master/conf目录下新建my.cnf

```java
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=101 
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql  
## 开启二进制日志功能
log-bin=mall-mysql-bin  
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M  
## 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed  
## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7  
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
```

1. 修改完配置后重启master容器实例

```java
docker restart mysql-master
```

1. 进入主节点mysql-master容器实例，进入数据库

```java
docker exec -it 主节点容器实例 /bin/bash
mysql -u root -proot
```

1.  master容器实例内创建数据同步用户并授予权限 CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
ALTER USER 'slave'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%'; 


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/2a672f04bc2879b3793f9c07145c2b3b.png)

1. 创建从节点MySQL容器实例对象，主机端口号为3308，容器内端口号为3306

```java
docker run -p 3308:3306 \
--name mysql-slave \
-v /opt/mysql-slave/log:/var/log/mysql \
-v /opt/mysql-slave/data:/var/lib/mysql \
-v /opt/mysql-slave/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=root  \
-d mysql:8.0.19
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/bd57b2f8805ce784dbdd8ed3cd3a8156.png)

1. 进入/opt/mysql-slave/conf目录下新建my.cnf

```java
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=102
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql  
## 开启二进制日志功能，以备Slave作为其它数据库实例的Master时使用
log-bin=mall-mysql-slave1-bin  
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M  
## 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed  
## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7  
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062  
## relay_log配置中继日志
relay_log=mall-mysql-relay-bin  
## log_slave_updates表示slave将复制事件写进自己的二进制日志
log_slave_updates=1  
## slave设置为只读（具有super权限的用户除外）
read_only=1
```

1. 修改完配置后重启slave实例

```java
docker restart mysql-slave
```

1. 在主节点数据库查看主从同步状态

```java
show master status;
```

获取到File和Position


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/5070453414a4d69a9807550b68099414.png)

1. 进入从节点，配置主从复制

```java
change master to master_host='宿主机ip', master_user='slave', master_password='123456', 
master_port=3307, master_log_file='mall-mysql-bin.000001', master_log_pos=1030, master_connect_retry=30;
```

主从复制参数说明：

>master_host：主数据库的IP地址；

1. 在从节点数据库中查看主从同步状态

```java
show slave status \G;
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/40bc2e7515be68cd7635305edcb1e2fb.png)

1. 在从节点数据库开启主从同步

```java
start slave;
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/b26f0578a2fecbe27c1512bf95839744.png)

1. 在从节点数据库再次查看主从同步状态是否变为Yes

```java
show slave status \G;
```


 ![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/84b8abf65daa361e74c49a8c755453eb.png)

1. 主从复制测试

主节点新建数据库，创建表，插入数据，然后在从节点中进行查看。

从节点数据库查询查看：


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/292681aa00de677c7d8582892f153a8d.png)


## 18.1分布式储存—哈希取余算法

hash(key) % N个机器台数，计算出哈希值，用来决定数据映射到哪一个节点上。

优点：

简单粗暴，直接有效，只需要预估好数据规划好节点，例如3台、8台、10台，就能保证一段时间的数据支撑。使用Hash算法让固定的一部分请求落到同一台服务器上，这样每台服务器固定处理一部分请求（并维护这些请求的信息），起到负载均衡+分而治之的作用。

缺点：

原来规划好的节点，进行扩容或者缩容就比较困难，不管扩缩，每次数据变动导致节点有变动，映射关系需要重新进行计算，在服务器个数固定不变时没有问题，如果需要弹性扩容或故障停机的情况下，原来的取模公式就会发生变化：Hash(key)/3会变成Hash(key) /?。此时地址经过取余运算的结果将发生很大变化，根据公式获取的服务器也会变得不可控。某个redis机器宕机了，由于台数数量变化，会导致hash取余全部数据重新洗牌。

## 18.2分布式储存—一致性哈希算法

一致性哈希算法为了解决分布式缓存数据变动和映射问题。目的是当服务器个数发生变动时， 尽量减少影响客户端到服务器的映射关系

算法构建一致性哈希环

为了在节点数目发生改变时尽可能少的迁移数据，将所有的存储节点排列在收尾相接的Hash环上，每个key在计算Hash后会顺时针找到临近的存储节点存放。而当有节点加入或退出时仅影响该节点在Hash环上顺时针相邻的后续节点。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/0e7d11fb2fae395ce02ce9254d1c0422.png)

优点

加入和删除节点只影响哈希环中顺时针方向的相邻的节点，对其他节点无影响。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/917498e8797f2449ab507b3d89a7cea7.png)

缺点

数据的分布和节点的位置有关，因为这些节点不是均匀的分布在哈希环上的，所以数据在进行存储时达不到均匀分布的效果。


1.  
 <ol> 
  1. ![img](https://img-blog.csdnimg.cn/img_convert/f88c8deab38c0aded621f23ffe3a6d78.png) 
 </ol> 

## 18.3分布式储存—哈希槽算法

哈希槽(散列插槽)实质就是一个数组，数组[0,2^14 -1]形成hash slot空间。解决均匀分配的问题，在数据和节点之间又加入了一层，把这层称为哈希槽（slot），用于管理数据和节点之间的关系，现在就相当于节点上放的是槽，槽里放的是数据。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/2f1ab2aebb3cfe8837132862c437092f.png)

一共有多少个hash槽

一个集群只能有16384个槽，编号0-16383（0-2^14-1）。这些槽会分配给集群中的所有主节点，分配策略没有要求。可以指定哪些编号的槽分配给哪个主节点。集群会记录节点和槽的对应关系。解决了节点和槽的关系后，接下来就需要对key求哈希值，然后对16384取余，余数是几key就落入对应的槽里。slot = CRC16(key) % 16384。以槽为单位移动数据，因为槽的数目是固定的，处理起来比较容易，这样数据移动问题就解决了。

**哈希槽计算**

Redis 集群中内置了 16384 个哈希槽，redis 会根据节点数量大致均等的将哈希槽映射到不同的节点。当需要在 Redis 集群中放置一个 key-value时，redis 先对 key 使用 crc16 算法算出一个结果，然后把结果对 16384 求余数，这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽，也就是映射到某个节点上。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/44e21d8f28df8547391b7c63b3329a50.png)

redis cluster采用数据分片的哈希槽来进行数据存储和数据的读取。redis cluster一共有2^14（16384）个槽，所有的master节点都会有一个槽区比如0～1000，槽数是可以迁移的。master节点的slave节点不分配槽，只拥有读权限。但是注意在代码中redis cluster执行读写操作的都是master节点，并不是读是从节点，写是主节点。

**为什么是16384个槽？** 在握手成功后，两个节点之间会定期发送ping/pong消息，交换数据信息，在redis节点发送心跳包时需要把所有的槽信息放到这个心跳包里，以便让节点知道当前集群信息，在发送心跳包时使用char进行bitmap压缩后是2k（16384÷8÷1024=2kb），也就是说使用2k的空间创建了16k的槽数。 虽然使用CRC16算法最多可以分配65535（2^16-1）个槽位，65535=65k，压缩后就是8k（8 * 8 (8 bit) * 1024(1k) = 8K），也就是说需要需要8k的心跳包，作者认为这样做不太值得；并且一般情况下一个redis集群不会有超过1000个master节点，所以16k的槽位是个比较合适的选择。

- 如果槽位为65536，发送心跳信息的消息头达8k，发送的心跳包过于庞大。 
- redis的集群主节点数量基本不可能超过1000个。集群节点越多，心跳包的消息体内携带的数据越多。如果节点过1000个，也会导致网络拥堵。因此redis作者，不建议redis cluster节点数量超过1000个。 
- 槽位越小，节点少的情况下，压缩率高

## 18.4redis分片集群扩缩容配置案例架构

### 18.4.1分片集群搭建

1.  关闭防火墙，启动docker后台服务  
2.  新建6个redis镜像的容器实例形成6个redis主从节点 

```java
docker run -d \
--name redis-node-1 \
--net host --privileged=true \
-v /data/redis/share/redis-node-1:/data redis:6.0.8 \
--cluster-enabled yes \
--appendonly yes \
--port 6381
```

>命令选项详解：


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/d15debde97fe5b3bf49dd7918181c18e.png)

1. **进入容器redis-node-1并为6台机器构建集群关系**

```java
redis-cli --cluster create --cluster-replicas 1 192.168.150.101:7001 192.168.150.101:7002 192.168.150.101:7003 192.168.150.101:8001 192.168.150.101:8002 192.168.150.101:8003
```

>命令选项详解：

三主三从，由于一个redis集群只有16384个插槽

 所以主节点6381的插槽范围是[0-5460]

所以主节点6381的插槽范围是[5461-10922]

所以主节点6381的插槽范围是[10923-16384]


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/64e71f0299b662d5430844378a884360.png)

1. **查看集群状态**

采用命令查看帮助手册

```java
redis-cli --cluster help
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/31eb9177bb23ba64b5f1498358df851e.png)

查看集群状态：

```java
cluster nodes
```

>6385是从节点，主节点是0e，即6383.<br> 6386是从节点，主节点是aa，即6381.<br> 6384是从节点，主节点是5b，即6382.


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/1af0531be77d5cea6c82e55f39d2b4b9.png)

### 18.4.2主节点哈希槽范围说明

因为当存入k-v键值对时，redis会对key进行crc16算法得出一个值，然后对16384取余，得出的结果位于哪个主节点的哈希槽范围中就存入哪个主节点。

```java
redis-cli -p 主节点端口
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/adaa15ccab45456bf826904074db67db.png)

所以在连接redis集群的时候要添加参数“-c”表示以集群的形式连接redis，这样能实现在主节点之间实现切换。

```java
redis-cli -c -p 主节点端口
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/c3ab9cb615b522462d5ec7626127aa03.png)

查看当前集群状态

```java
redis-cli --cluster check 主机ip:容器中redis端口
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/5c2261c321ea3f7fcdf2bc9dd32dd9f6.png)

### 18.4.3主从容错

 当主节点宕机之后，它的从节点会转换为主节点。


 测试，停止主节点6381，查看其和它的从节点6386的状态

 6381连接失败，其从节点6386变为主节点。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/e24d8c926651882d6251f8bb27a817ce.png)

 恢复之前的主节点6381，查看其和主节点6386的状态

 6381变为从节点，6386还是主节点。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/f508206f22db932983e84686b6566ef8.png)

### 18.4.4主从扩容

在原来的3主3从得基础上扩容到4主4从

1. **根据redis镜像新建两个容器实例，对应的端口号分别是6387、6388**

```java
docker run -d --name redis-node-1 --net host --privileged=true -v /data/redis/share/redis-node-1:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6381
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/11437d143c30c5942f44b835d955bd61.png)

1. **将新增的6387节点作为master节点添加到redis集群中**

 查看帮助

```java
redis-cli --cluster help
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/5f808684a46c805ab12e5ac0f5220d30.png)

将6387节点作为主节点添加到集群当中

```java
redis-cli --cluster add-node new_host:new_port existing_host:existing_port
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/bebf682fff5dbeb8df7ec573a1cc242a.png)

1. **查看集群状态**

 可以看到新加入的主节点 并没有分配哈希槽位


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/ac7c74638232cce07fb12787e6e1755b.png)

1. **重新分配哈希槽位**


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/fa835c6d344035a20a9f9c1621c1e58a.png)

1.  再次查看集群状态 可以发现为新节点分配的哈希槽位并不是连续的，其哈希槽位是由原来的每一个主节点分配的。 


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/d5fc6d8df6c9c8a0115b1b3b8616d5ec.png)

1. **添加新节点6388作为主节点6387的从节点**

```java
redis-cli --cluster add-node 192.168.26.135:6388 192.168.26.135:6387 --cluster-slave --cluster-master-id ee499e33ad50ffaf84d4ed95a39625c49dc1ae4b
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/c893499ceb86f1fb0b955083f37ca338.png)

### 18.4.5主从缩容

使6387和6388主从节点下线

1.  查看6387和6388的节点ID  
2.  将6388从节点从主节点上删除  
3.  将主节点的哈希槽位清空并重新分配槽位给master1 


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/61fb091e651c858b8ee3acb1181ca3ff.png)

1. **删除空槽位的主节点**

```java
redis-cli --cluster del-node 192.168.26.135:6387 ee499e33ad50ffaf84d4ed95a39625c49dc1ae4b
```


## 19.1Dockerfile简介

 [Dockerfile reference | Docker Documentation](https://docs.docker.com/engine/reference/builder/)

Dockerfile是用来构建Docker镜像的文本文件，是由一条条构建镜像所需的指令和参数构成的脚本


**构建Docker镜像的步骤：**

1. 编写Dockerfile文件 
2. docker build 命令构建镜像 
3. docker run 镜像 运行容器实例

## 19.2Dockerfile内容基础知识

1. 每条保留字指令都必须为大写字母且后面要跟随至少一个参数 
2. 指令按照从上到下，顺序执行 
3. #表示注释 
4. 每条指令都会创建一个新的镜像层并对镜像进行提交

## 19.3docker执行Dockerfile的大致流程

1. docker从基础镜像运行一个容器 
2. 执行一条指令并对容器作出修改 
3. 执行类似docker commit的操作提交一个新的镜像层 
4. docker再基于刚提交的镜像运行一个新容器 
5. 执行dockerfile中的下一条指令直到所有指令都执行完成

## 19.4docker镜像、docker容器、Dockerfile

从应用软件的角度来看，Dockerfile、Docker镜像与Docker容器分别代表软件的三个不同阶段，

- **Dockerfile是软件的原材料** 
- **Docker镜像是软件的交付品** 
- **Docker容器则可以认为是软件镜像的运行态，即依照镜像运行的容器实例**

Dockerfile面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/b669895127124fa6a458f31fdaf0a349.png)

1. Dockerfile，需要定义一个Dockerfile，Dockerfile定义了进程需要的一切东西。Dockerfile涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进程和内核进程(当应用进程需要和系统服务和内核进程打交道，这时需要考虑如何设计namespace的权限控制)等等; 
2. Docker镜像，在用Dockerfile定义一个文件之后，docker build时会产生一个Docker镜像，当运行 Docker镜像时会真正开始提供服务; 
3. Docker容器，容器是直接提供服务的。

## 19.5DockerFile常用保留字指令

-  
 <blockquote> 
  <p><strong>FROM</strong></p> 
 </blockquote> 

基础镜像，当前新镜像是基于哪个镜像的，指定一个已经存在的镜像作为模板，第一条必须是from

-  
 <blockquote> 
  <p><strong>MAINTAINER</strong></p> 
 </blockquote> 

 镜像维护者的姓名和邮箱地址

-  
 <blockquote> 
  <p><strong>RUN</strong></p> 
 </blockquote> 

 容器构建时需要运行的命令

-  
 <blockquote> 
  <p><strong>EXPOSE</strong></p> 
 </blockquote> 

 当前容器对外暴露出的端口

-  
 <blockquote> 
  <p><strong>WORKDIR</strong></p> 
 </blockquote> 

 指定在创建容器后，终端默认登陆的进来工作目录，一个落脚点

-  
 <blockquote> 
  <p><strong>USER</strong></p> 
 </blockquote> 

 指定该镜像以什么样的用户去执行，如果都不指定，默认是root

-  
 <blockquote> 
  <p><strong>ENV</strong></p> 
 </blockquote> 

 用来在构建镜像过程中设置环境变量

 这个环境变量可以在后续的任何RUN指令中使用，这就如同在命令前面指定了环境变量前缀一样；

 也可以在其它指令中直接使用这些环境变量。

 比如：WORKDIR $MY_PATH

-  
 <blockquote> 
  <p><strong>ADD</strong></p> 
 </blockquote> 

 将宿主机目录下的文件拷贝进镜像且会自动处理URL和解压tar压缩包

-  
 <blockquote> 
  <p><strong>COPY</strong></p> 
 </blockquote> 

 类似ADD，拷贝文件和目录到镜像中。

 将从构建上下文目录中 &lt;源路径&gt; 的文件/目录复制到新的一层的镜像内的 &lt;目标路径&gt; 位置

-  
 <blockquote> 
  <p><strong>VOLUME</strong></p> 
 </blockquote> 

 容器数据卷，用于数据保存和持久化工作

-  
 <blockquote> 
  <p><strong>CMD</strong></p> 
 </blockquote> 

 指定容器启动后的要干的事情

-  
 <blockquote> 
  <p><strong>ENTRYPOINT</strong></p> 
 </blockquote> 

 也是用来指定一个容器启动时要运行的命令

 类似于 CMD 指令，但是ENTRYPOINT不会被docker run后面的命令覆盖， 而且这些命令行参数会被当作 参数送给 ENTRYPOINT 指令指定的程序。

## 19.6使用Dockerfile实现自定义镜像

自定义镜像需求：具备vim+ifconfig+jdk8环境

1. **创建文件夹myfile，并将jdk压缩包传到当前目录下**


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/ece6443126a271c0328aaebb4cbc0338.png)

1. **创建Dockerfile文件，编辑内容**

```java
FROM centos:7
MAINTAINER xha<2533694604@qq.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

#安装vim编辑器
RUN yum -y install vim
#安装ifconfig命令查看网络IP
RUN yum -y install net-tools
#安装java8及lib库
RUN yum -y install glibc.i686
RUN mkdir /usr/local/java
#ADD 是相对路径jar,把jdk-8u341-linux-x64.tar.gz添加到容器中,安装包必须要和Dockerfile文件在同一位置
ADD jdk-8u341-linux-x64.tar.gz /usr/local/java/
#配置java环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_341
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH

EXPOSE 80

CMD echo $MYPATH
CMD echo "success--------------ok"
CMD /bin/bash
```

注意：centos版本指定为7 centos:7


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/dc6ae2a1c1d8f38ac7b53f667efde252.png)

1. **构建镜像**

```java
docker build -t centosjava8:1.0 .
```

构建完成：


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/efe74375db6353b07efff9d4810d0b56.png)

查看创建的镜像


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/40c415a92a163701c8ca6b6d7f7847a9.png)

1. **运行镜像**

```java
docker run -it 镜像ID /bin/bash
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/28b9849a82a6ba9942d762723ede0509.png)

## 19.7虚悬镜像

仓库名和版本号都为的就是虚悬镜像


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/bde3d912a461918399636e166ab46397.png)

查看所有的虚悬镜像

```java
docker image ls -f dangling=true
```

删除所有虚悬镜像

```java
docker image prune
```


## 20.1将jar包部署到docker容器上

1. **创建新目录，将jar包放在该目录下**


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/99f032b02a3cb5594d1c66334693cf8f.png)

1. **编写Dockerfile文件**

```java
#基础镜像使用java
FROM java:8
# 作者
MAINTAINER xha
# VOLUME 指定临时文件目录为/tmp，在主机/var/lib/docker目录下创建了一个临时文件并链接到容器的/tmp
VOLUME /tmp
# 将jar包添加到容器中并更名
ADD docker_test-0.0.1-SNAPSHOT.jar dockertest.jar
# 运行jar包
RUN bash -c 'touch /dockertest.jar'
ENTRYPOINT ["java","-jar","/dockertest.jar"]
#暴露8090端口作为微服务
EXPOSE 8090
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/d00913dd639e8d5a2b224c01079dba95.png)

1. **构建镜像**

在当前目录下构建镜像

```java
docker build -t dockertest:1.0 .
```

镜像构建成功


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/69ecbb3a4ca9fb982e4a67a9cca4d9ba.png)


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/344cedcc8589f9228ceff6477b743f3e.png)

1. **运行镜像**

```java
docker run -d -p 8090:8090 镜像ID
```

1. **测试项目接口**


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/98e8d94206bd2f2064efaa826c8cc82d.png)


## 21.1docker网络是什么

Docker启动时会在主机上自动创建一个docker0网桥，即一个Linux网桥。容器借助网桥和主机或者其他容器进行通讯。

## 21.2docker启动与不启动时的网络情况

**一：不启动docker时**


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/a7a2d97c8c7b0ee540f07e27ddee507f.png)

- ens33是宿主机ip

 可以发现是在同一网段


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/5685dc1f53beabd7d80edc2e67b082ac.png)

- lo是回环链路网络 
- virbr0

 当选择虚拟化相关的服务后，启动网卡时就会有一个网桥连接的私网地址virbr0网卡（有一个固定的ip地址192.168.122.1），是做虚拟网桥使用的，其作用是连接虚拟机上的虚拟网卡，提供NAT访问外网的功能。

**二：启动docker时**

 会产生一个名为docker0的虚拟网桥


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/5d18be72ceebab1e51484a7607d9933f.png)

## 21.3docker网络的相关命令

查看docker网络的相关命令

```java
docker network --help
```


## 21.4docker网络的作用

-  容器间的互联和通信以及端口映射  
-  容器IP变动时候可以通过服务名直接网络通信而不受到影响 

## 21.5docker网络模式

### 21.5.1docker网络模式分类


### 21.5.2bridge网络模式

Docker 服务默认会创建一个 docker0 网桥（其上有一个 docker0 内部接口），该桥接网络的名称为docker0，它在内核层连通了其他的物理或虚拟网卡，这就将所有容器和本地主机都放到同一个物理网络。Docker 默认指定了 docker0 接口 的 IP 地址和子网掩码，让主机和容器之间。宿主机和容器之间、容器与容器之间可以通过网桥docer0(bridge)相互通信


查看bridge网桥

```java
docker network inspect bridge
```


**使用tomcat镜像创建两个容器，宿主机端口分别是8081和8082，容器端口都是8080**


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/5e55cf57d8d97e0067bfba0148cd1866.png)

**进入容器t1，查看网络模式**

162：eth0@if163 有eth0


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/f67cb98815c2b50007b1fcfc5a4b54e7.png)

**进入容器t2，查看网络模式**

164：eth0@if165 有eth0


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/83eb5960be980e6c57de139dad667784.png)

**在宿主机查看网络模式**

163：veth@if162 有veth

165：veth@if164 有veth


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/1b1b076884cbfad9799a067139003291.png)

综上可以看出，容器内都有一个网络模式和网桥进行相连，实现和宿主机两两匹配验证。

### 21.5.3host网络模式

容器将不会获得一个独立的Network Namespace， 而是和宿主机共用一个Network Namespace。容器将不会虚拟出自己的网卡而是使用宿主机的IP和端口。


在之前搭建redis分片集群的时候，各个redis主从节点使用的网络模式就是host


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/41808345fc707f6e29f441225340cc45.png)

可以发现各个主从节点并没有对应的端口，即指定端口并没有意义，而是和宿主机公用ip和端口号。


使用host网络模式创建tomcat镜像

```java
docker run -d -p 8083:8080 --network=host --name t3 tomcat:8.0.53
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/fbe1096d18dfd063d5a23fab46b011a8.png)

查看容器是否有端口


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/583e40ffbea1aab4c8e3a7bc6ffef570.png)

查看容器的网络模式，可以发现没有ip


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/84cc26c799ca3b20c4e25661629d5e96.png)

查看宿主机和容器的ip信息，可以发现是一模一样的，即**容器使用的是宿主机的ip和端口**。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/0c64c7ea145f01aac99415d9a1559b81.png)


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/b638861285d1f366420c8022c35ce402.png)

### 21.5.4none网络模式

在none模式下，并不为Docker容器进行任何网络配置。 也就是说，这个Docker容器没有网卡、IP、路由等信息，只有一个lo需要我们自己为Docker容器添加网卡、配置IP等。

### 21.5.5container网络模式

新建的容器和已经存在的一个容器共享一个网络ip配置而不是和宿主机共享。新创建的容器不会创建自己的网卡，配置自己的IP，而是和一个指定的容器共享IP、端口范围等。同样，两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。


测试：

创建一个tomcat容器，再创建第二个tomcat容器，创建第二个容器的时候指定使用第一个tomcat容器的网络配置。

```java
docker run -d -p 8084:8080 --name t4 tomcat:8.0.19
docker run -d -p 8085:8080 --network container:t4 --name t5 tomcat:8.0.53
```

但是会提示端口号冲突，因为两台tomcat公用8080端口。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/ca4f77e59adb900537ae21614648299d.png)

### 21.5.6自定义网络模式

**一：使用刚刚创建的两个tomcat镜像容器实例t1和t2，在容器内部首先进行ping对方的ip**

发现可以ping通


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/3527fe6150a17c9793cc5190a6fcbf14.png)

**二：但是当按照服务名ping时发现没有ping通**


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/0b86f8078551c1db21d8bd29022797f2.png)

**三：自定义网络模式**

 docker多个容器之间的集群规划要使用服务名，因为ip是会变动，使用自定义网络模式能够使用服务名进行通信

1. 创建网络，默认模式是bridge（网桥）模式

```java
docker network create 网络名
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/028c5695dfacd968c4ceb4dc88aa913d.png)

1. 创建两个容器实例，并使用自定义的网络模式

```java
docker run -d -p 8081:8080 --network myselfnet --name t1 tomcat:8.0.19
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/886c9539a370a847ef08c4b072d5f105.png)

测试使用服务名ping


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/e318b2f8227f5e5385dd500dd2f8180d.png)

## 21.6docker网络底层ip和容器的映射变化

docker容器内部的ip是有可能是会发生变化的

使用同一镜像创建两个容器实例，并查看网络信息

```java
docker inspect 容器ID
```

第一个容器


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/f0b579f9c900cc00f821270ab4768647.png)

第二个容器


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/b5a1ab52d9618a8ee5f1a136ecd11842.png)

删除第二个容器，再创建一个容器，查看ip


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/2215ff2043bcbcd131bcd972cdd5d47a.png)

可以发现当第二个容器被停止之后，创建的第三个容器的ip与第二个相同。


## 22.1Docker-compose的定义

Docker-Compose就是容器编排，负责实现对Docker容器集群的快速编排。

Docker-Compose可以管理多个 Docker 容器组成一个应用。你需要定义一个 YAML 格式的配置文件docker-compose.yml，写好多个容器之间的调用关系。然后，只要一个命令，就能同时启动/关闭这些容器

## 22.2Docker-Compose的作用

==docker建议我们每一个容器中只运行一个服务,因为docker容器本身占用资源极少,所以最好是将每个服务单独的分割开来。==但是这样我们又面临了一个问题？

如果我需要同时部署好多个服务,难道要每个服务单独写Dockerfile然后在构建镜像,构建容器,这样工作量会和大。所以docker官方给我们提供了docker-compose多服务部署的工具

例如要实现一个Web微服务项目，除了Web服务容器本身，往往还需要再加上后端的数据库mysql服务容器，redis服务器，注册中心eureka，甚至还包括负载均衡容器等等。

Compose允许用户通过一个单独的docker-compose.yml模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。

可以很容易地用一个配置文件定义一个多容器的应用，然后使用一条指令安装这个应用的所有依赖，完成构建。Docker-Compose 解决了容器与容器之间如何管理编排的问题。

## 22.3Docker-Compose的安装

Docker-Compose官网： [Docker Desktop | Docker Documentation](https://docs.docker.com/desktop/)


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/63e0e192e1bde0d7d10c83b0147a5f0d.png)

安装命令：

```java
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker} 

mkdir -p $DOCKER_CONFIG/cli-plugins 

curl -SL https://github.com/docker/compose/releases/download/v2.11.2/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose

chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/c13e147a5c5bd3e552cbb22c8647539a.png)

打印版本号，查看是否成功

```java
docker compose version
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/40bcf1573a4c038be990a38e4e8902d8.png)

## 22.4Docker-Compose的常用命令


## 22.5Docker-Compose核心概念


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/ecf3bb045f487478002d4aa684da517c.png)

## 22.6Docker-Compose的使用步骤

-  编写Dockerfile定义各个微服务应用并构建出对应的镜像文件  
-  使用 docker-compose.yml 定义一个完整业务单元，安排好整体应用中的各个容器服务。  
-  最后，执行docker-compose up命令 来启动并运行整个应用程序，完成一键部署上线 

## 22.7不使用Compose编排存在的问题

1. **不使用Compose部署项目的方式是分别启动mysql服务、redis服务和微服务镜像文件，部署完成**。 
2. **存在的问题**

-  先后顺序要求固定，先mysql+redis才能微服务访问成功  
-  多个run命令…  
-  容器间的启停或宕机，有可能导致IP地址对应的容器实例变化，映射出错， 要么生产IP写死(可以但是不推荐)，要么通过服务调用 

## 22.7使用Compose编排微服务

1.  编写docker-compose.yml文件 在vim模式下 :set paste粘贴的文本数据不会乱 #compose版本
version: "3"  

#微服务项目	
services:
  microService:
#微服务镜像  
    image: zzyy_docker:1.6
    container_name: ms01
    ports:
      - "6001:6001"
#数据卷
    volumes:
      - /app/microService:/data
    networks: 
      - atguigu_net 
    depends_on: 
      - redis
      - mysql
      
#redis服务
  redis:
    image: redis:6.0.8
    ports:
      - "6379:6379"
    volumes:
      - /app/redis/redis.conf:/etc/redis/redis.conf
      - /app/redis/data:/data
    networks: 
      - atguigu_net
    command: redis-server /etc/redis/redis.conf

 #mysql服务
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: '123456'
      MYSQL_ALLOW_EMPTY_PASSWORD: 'no'
      MYSQL_DATABASE: 'db2021'
      MYSQL_USER: 'zzyy'
      MYSQL_PASSWORD: 'zzyy123'
    ports:
       - "3306:3306"
    volumes:
       - /app/mysql/db:/var/lib/mysql
       - /app/mysql/conf/my.cnf:/etc/my.cnf
       - /app/mysql/init:/docker-entrypoint-initdb.d
    networks:
      - atguigu_net
    command: --default-authentication-plugin=mysql_native_password #解决外部无法访问

 #创建自定义网络
networks: 
   atguigu_net:  
2.  修改微服务项目，将映射的ip修改为服务名 

 将SpringBoot项目的配置文件中mysql和redis的ip更改为项目名。之后打包成jar包，在docker中编写Dockerfile文件，构建镜像。

1. **检查当前目录下compose.yml文件是否有语法错误**

```java
docker compose config -q
```

1. **启动所有docker-compose服务并后台运行**

```java
docker compose up -d
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/ecd3210e675c97f2350eb2652258e1fc.png)


## 23.1Protainer是什么

Portainer 是一款轻量级的应用，它提供了图形化界面，用于方便地管理Docker环境，包括单机环境和集群环境。

## 23.2Protainer安装

 [Docker and Kubernetes Management | Portainer](https://www.portainer.io/)

1. **拉取portainer**

```java
docker pull portainer/portainer
```

1. **根据portainer镜像创建容器实例**

restart=always表示即使重启docker服务，Protainer容器依旧存在。

```java
docker run -d -p 9000:9000 --name portainer \
--restart=always \ 
-v /var/run/docker.sock:/var/run/docker.sock \
-v /www/portainer/data:/data \
portainer/portainer
```

查看容器实例


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/697e7c7ba3486e9eeda523c95cd081b8.png)

1. **访问宿主机ip加protainer端口号**

输入密码




![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/7841c2bd8f2688b55bc7add10bd2f7a9.png)

可以查看到镜像、容器、网络等等。


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/6e60613eecc6bd1ddbccb891027e9e9f.png)

## 23.3在Protainer中安装Nginx

1. **添加nginx容器**


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/8d18cbe791ab0d3762c6432fbd9d7da7.png)

1. **nginx镜像拉取成功，nginx容器创建成功**


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/baafc2f1976ac5317678dc018857ac87.png)

1. **访问80端口查看nginx服务是否正常启动**


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/401fb0cc28d0b727bbe6bc9ce070bc3b.png)


## 24.1原生命令监控容器的详细信息

```java
docker status
```


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/cc33da00464826b3820b589f53130a93.png)

问题：

通过docker stats命令可以很方便的看到当前宿主机上所有容器的CPU,内存以及网络流量等数据，一般小公司够用了。

但是，docker stats统计结果只能是当前宿主机的全部容器，数据资料是实时的，没有地方存储、没有健康指标过线预警等功能

## 24.2CIG监控

CIG分别表示：**CAdvisor监控收集+InfluxDB存储数据+Granfana展示图表**


![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/5c04e6d9b632364d92bb8dc36583a99b.png)

-  CAdvisor是一个容器资源监控工具包括容器的内存,CPU,网络IO,磁盘I0等监控。  
-  InfluxDB是用Go语言编写的一个开源分布式时序、 事件和指标数据库,无需外部依赖。  
-  Grafana是一个开源的数据监控分析可视化平台，支持多种数据源配置(支持的数据源包括 

## 24.3使用Compose搭建CIG监控平台

1. **新建cig目录，在目录下创建docker-compose.yml文件，实现容器编排。**

```java
version: '3.1'
 
volumes:
  grafana_data: {}
 
services:
 influxdb:
  image: tutum/influxdb:0.9
  restart: always
  environment:
    - PRE_CREATE_DB=cadvisor
  ports:
    - "8083:8083"
    - "8086:8086"
  volumes:
    - ./data/influxdb:/data
 
 cadvisor:
  image: google/cadvisor
  links:
    - influxdb:influxsrv
  command: -storage_driver=influxdb -storage_driver_db=cadvisor -storage_driver_host=influxsrv:8086
  restart: always
  ports:
    - "8080:8080"
  volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:rw
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
 
 grafana:
  user: "104"
  image: grafana/grafana
  user: "104"
  restart: always
  links:
    - influxdb:influxsrv
  ports:
    - "3000:3000"
  volumes:
    - grafana_data:/var/lib/grafana
  environment:
    - HTTP_USER=admin
    - HTTP_PASS=admin
    - INFLUXDB_HOST=influxsrv
    - INFLUXDB_PORT=8086
    - INFLUXDB_NAME=cadvisor
    - INFLUXDB_USER=root
    - INFLUXDB_PASS=root
```

1. **检查docker-compose.yml文件是否有错误**

```java
docker-compose config -q
```

1. **后台启动docker-compose服务**

```java
docker-compose up -d
```



---

> 作者: wang  
> URL: https://codingroam.github.io/zh-cn/post/docker%E8%AF%A6%E8%A7%A3%E5%B0%9A%E7%A1%85%E8%B0%B7%E9%98%B3%E5%93%A5/  

