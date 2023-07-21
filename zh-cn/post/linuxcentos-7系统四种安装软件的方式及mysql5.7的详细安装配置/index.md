# Linux(CentOS 7)系统四种安装软件的方式及mysql5.7的详细安装配置


本文以安装mysql为例介绍了Linux(CentOS 7)环境下安装软件的四种方式，比较四种方式的难易程度，以及详细介绍了mysql5.7的安装和配置方法

<!--more-->

# Linux安装软件的四种方式介绍

四种方式：rpm命令、yum命令、带bin目录的tar安装包和source源码编译安装

## 四种方式比较

1. rpm与yum为命令安装，不同的是npm在安装时需要手动下载依赖包，而yum会自动下载依赖

2. 带bin的tar包和source源码安装区别是前者是编译好的，bin文件夹中就是编译之后的二进制可执行文件，后者需要自己手动编译

## 四种方式操作说明

### 一、npm和yum安装

rpm与yum命令对比（yum简单，rpm复杂）

> + RPM 全名 RedHat Package Managerment，是由Red Hat公司提出，被众多Linux发行版本所采用，是一种数据库记录的方式来将所需要的软件安装到到Linux系统的一套软件管理机制
> + rpm在安装时有严格的顺序限制，包与包有依赖关系，且安装过程中可能依赖别的包需要手动安装，而yum安装某个功能（例如mysql的server端）会自动下载安装依赖
> +    安装软件：rpm -ivh [软件包名称]
>      卸载软件：rpm -e [软件包名称]
>      更新软件：rpm -Uvh [软件包名称]
>
> +   yum check-update：列出所有可更新的软件清单命令;
>
>     yum update：更新所有软件或指定软件命令;
>
>     yum install ：仅安装指定的软件命令；
>
>     yum list：列出所有可安装的软件清单命令；
>
>     yum remove ：删除软件包命令；
>
>     yum search ：查找软件包命令：

   以安装mysql为例，对比yum和rpm的安装过程

①查看是否安装了mysql/mariadb的服务

```
[root@localhost ~]# rpm -qa |grep -i mysql
MySQL-client-5.6.23-1.sles11.x86_64
MySQL-server-5.6.23-1.sles11.x86_64
MySQL-shared-5.6.23-1.sles11.x86_64
MySQL-devel-5.6.23-1.sles11.x86_64
...
[root@localhost ~]# root@# rpm -qa |grep -i mariadb
...
```

②如果安装需要卸载所有服务

``` 
[root@localhost ~]#  rpm -e --nodeps MySQL-client-5.6.23-1.sles11.x86_64
[root@localhost ~]#  rpm -e --nodeps MySQL-server-5.6.23-1.sles11.x86_64
...
```

③使用命令或者去mysql官网下载rpm包

+ ``` 
  yum安装mysql需要先下载一个基础包安装
  [root@localhost ~]#  wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
  
  ```

+ rpm安装需要下载bundle包（rpm文件打包合集）

  ![image-20220527150223317](/images/image-20220527150223317.png)

   rpm Bundle表示rpm的打包合集，包含基础包，lib包，client,server 等包如下图

  ![image-20220527162211385](/images/image-20220527162211385.png)

④安装：

+ yum方式

  ``` 
  先安装基础包：
  [root@localhost ~]# yum install y mysql57-community-release-el7-10.noarch.rpm 
  [root@localhost ~]# yum install mysql-community-server --nogpgcheck
  --安装完毕
  ```

+ rpm

  ``` 
  rpm需要依次安装bundle包中的rpm包，包括基础包，库，client和server等
  安装mysql-server服务，只需要安装如下4个软件包即可，使⽤rpm -ivh进⾏安装（按顺序安装，后⾯的
  服务依赖前⾯的服务
  [root@localhost ~]# rpm -ivh mysql-community-common-5.7.23-1.el7.x86_64.rpm
  [root@localhost ~]# rpm -ivh mysql-community-libs-5.7.23-1.el7.x86_64.rpm
  [root@localhost ~]# rpm -ivh mysql-community-client-5.7.23-1.el7.x86_64.rpm
  [root@localhost ~]# rpm -ivh mysql-community-server-5.7.23-1.el7.x86_64.rpm
  安装时如果出现缺少依赖，如少libaio、net-tools还需要yum install [名称]来安装依赖
  ```

  

⑤启动mysql，设置密码

``` 
1、[root@localhost ~]# systemctl start  mysqld.service -->启动mysql
2、[root@localhost ~]# grep "password" /var/log/mysqld.log -->查看密码 CSLQ:F=Um5i1
    A temporary password is generated for root@localhost: CSLQ:F=Um5i1
3、[root@localhost ~]# mysql -uroot -p -->登录root用户
4、[root@localhost ~]# CSLQ:F=Um5i1 -->输入密码
5、登录进mysql之后设置密码规则(不设置有可能无法修改成简单密码)
  set global validate_password_policy=0;
  set global validate_password_length=1;
6、ALTER USER 'root'@'localhost' IDENTIFIED BY 'root'; -->修改密码为root
```

⑥查找并修改mysql配置文件

``` 
1、[root@localhost ~]# which mysql -->查找mysql命令在什么位置
   usr/bin/mysql
2.[root@localhost ~]# /usr/bin/mysql --verbose --help | grep -A 1 'Default options'
   Default options are read from the following files in the given order:
   /etc/mysql/my.cnf /etc/my.cnf ~/.my.cnf 
返回信息表示首先读取的是/etc/mysql/my.cnf文件，如果前一个文件不存在则继续读/etc/my.cnf文件，如若还不存在便会去读~/.my.cnf文件,这三处即是mysql配置文件存放处，找到修改即可
默认配置文件如下：
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid


```

### 二、tar包安装和source源码安装

> 已编译tar包(带bin目录的tar)和source源码安装区别是前者是编译好的，bin文件夹中就是编译之后的二进制可执行文件，后者需要自己手动编译

①带bin的tar包安装

![image-20220527172826841](/images/image-20220527172826841.png)

``` 
1、[root@localhost ~]#  tar -xvf mysql-5.7.26-linux-glibc2.12-x86_64.tar 
2、[root@localhost ~]#  cp -r mysql-5.7.26-linux-glibc2.12-x86_64 /usr/local/mysql -->拷贝文件夹到/usr/local目录下并重命名为mysql
3、[root@localhost ~]# mkdir /usr/local/mysql/data /usr/local/mysql/logs -->创建data和logs文件夹
4、[root@localhost ~]#  groupadd mysql  -->添加mysql用户组
5、[root@localhost ~]#  useradd -r -g mysql mysql  -->向mysql用户组添加mysql用户，-r <参数表示mysql用户是系统用户，不可用于登录系统，-g 参数表示把mysql用户添加到mysql用户组中
6、chown -R mysql:mysql /usr/local/mysql/ -->将mysql目录权限分配给mysql用户组下的mysql用户
已编译tar包安装需要新建mysql用户和用户组，并将mysql目录权限分配给用户，若不进行此操作，在mysql服务启动时会报Starting MySQL. ERROR! The server quit without updating PID file错误
7、[root@localhost ~]#  vim /etc/my.cnf -->在/etc下新增配置文件
[mysqld]
port = 3306
user = mysql
basedir = /usr/local/mysql
datadir = /usr/local/mysql/data
socket =  /usr/local/mysql/data/mysql.sock
bind-address = 0.0.0.0
pid-file =  /usr/local/mysql/data/mysqld.pid
character-set-server = utf8
collation-server = utf8_general_ci
max_connections = 200
log-error =  /usr/local/mysql/logs/mysqld.log
8、[root@localhost ~]# cd /usr/local/mysql/bin/ -->进入mysql的bin目录
9、[root@localhost ~]#./mysqld --defaults-file=/etc/my.cnf --user=mysql --basedir=/usr/local/mysql/ --datadir=/data/mysql/  --initialize -->初始化
10、[root@localhost ~]# cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql -->将mysql.server放置到/etc/init.d/mysql中
11、[root@localhost ~]# service mysql start -->启动服务
12、[root@localhost ~]# cat /usr/local/mysql/logs/mysqld.log -->日志最后一行有随机生成的初始密码,可登录mysql
13、[root@localhost ~]# ln -s  /usr/local/mysql/bin/mysql    /usr/bin -->创建软连接到/usr/bin可以全局使用mysql命令(方便登录mysql)
14、[root@localhost ~]# mysql -uroot -p -->登录mysql
15、[root@localhost ~]# 输入12步得到的密码，登录mysql
16、若登录出现Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)，需要将mysql.sock软连接到tmp目录即可 [root@localhost ~]# ln -s /usr/local/mysql/data/mysql.sock /tmp
```



②source源码安装

![img](/images/Snipaste_2022-05-27_18-22-03.png)

``` 
(1)创建安装目录
 [root@localhost ~]# mkdir /usr/local/mysql/{data,logs,tmp,run} -p
(2）首先安装源码编译所需要的包
 [root@localhost ~]# yum -y install make gcc-c++ cmake bison-devel  ncurses-devel
(3)解压
 [root@localhost ~]# tar -zxvf mysql-5.7.27.tar.gz 
 [root@localhost ~]# tar -zxvf mysql-boost-5.7.27.tar.gz  #（不需要安装，在安装mysql时自动安装，）
两个文件解压以后都会在同一个目录上 mysql-5.7.27 

(4)编译安装（编译参数按实际情况制定）
 cd mysql-5.7.27
 cmake . \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql/ \
-DMYSQL_DATADIR=/usr/local/mysql/data \
-DDOWNLOAD_BOOST=1 \
-DWITH_BOOST=/opt/software/mysql-5.7.27/boost \
-DSYSCONFDIR=/etc \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITH_FEDERATED_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DENABLED_LOCAL_INFILE=1 \
-DMYSQL_UNIX_ADDR=/usr/local/mysql/run/mysql.sock \
-DENABLE_DTRACE=0 \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DWITH_EMBEDDED_SERVER=1
 
make & make install
(5)/etc/my.cnf配置mysql
(6) ./mysqld --defaults-file=/etc/my.cnf --initialize #初始化
(7) cp support-files/mysql.server /etc/init.d/mysql #启动项设置
(8) service mysqld start
(9)登录不在赘述

```

## mysql常见报错及处理方式：

1. Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)

   https://blog.csdn.net/weixin_43464964/article/details/121807241

2. 防火墙问题导致mysql连不上：https://www.68idc.com/news/content/582.html

3. 连接mysql报错误码1130：https://blog.csdn.net/web_15534274656/article/details/126493936

4. CentOS7查看开放端口命令、查看端口占用情况和开启端口命令、杀掉进程

   https://blog.csdn.net/weixin_48053866/article/details/127715462

5. centos 7 tcp6端口地址不通，导致无法访问：https://www.pianshen.com/article/54202723812/





​    



---

> 作者: wang  
> URL: https://codingroam.github.io/zh-cn/post/linuxcentos-7%E7%B3%BB%E7%BB%9F%E5%9B%9B%E7%A7%8D%E5%AE%89%E8%A3%85%E8%BD%AF%E4%BB%B6%E7%9A%84%E6%96%B9%E5%BC%8F%E5%8F%8Amysql5.7%E7%9A%84%E8%AF%A6%E7%BB%86%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE/  

