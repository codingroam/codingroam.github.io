# Linux常用设置、命令杂记


记录Linux常用的便捷设置和常用命令

<!--more-->

1. ### 快速卸载已安装的rpm软件

   rpm -qa | grep -i java | xargs -n1 rpm -e --nodeps 

   Ø rpm -qa：查询所安装的所有rpm软件包

   Ø grep -i：忽略大小写

   Ø xargs -n1：表示每次只传递一个参数

   Ø rpm -e –nodeps：强制卸载软件，忽略依赖

2. ### 安装java

   ***\*1\*******\*）\*******\*卸载现有\*******\*JDK\****

   注意：安装JDK前，一定确保提前删除了虚拟机自带的JDK。详细步骤见问文档3.1节中卸载JDK步骤。

   ***\*2\*******\*）\*******\*用\*******\*XShell传输\*******\*工具将JDK导入到opt目录下面的software文件夹下面\****

   ![img](/images/javapackage.jpg) 

   ***\*3\*******\*）\*******\*在Linux系统下的opt目录中查看软件包是否导入成功\****

   [atguigu@hadoop102 ~]$ ls /opt/software/

   看到如下结果：

   jdk-8u212-linux-x64.tar.gz

   ***\*4\*******\*）\*******\*解压JDK到/opt/module目录下\****

   [atguigu@hadoop102 software]$ tar -zxvf jdk-8u212-linux-x64.tar.gz -C /opt/module/

   ***\*5\*******\*）\*******\*配置JDK环境变量\****

   ​	（1）新建/etc/profile.d/my_env.sh文件

   [atguigu@hadoop102 ~]$ sudo vim /etc/profile.d/my_env.sh

   添加如下内容

   \#JAVA_HOME

   export JAVA_HOME=/opt/module/jdk1.8.0_212

   export PATH=$PATH:$JAVA_HOME/bin

   ​	（2）保存后退出

   :wq

   ​	（3）source一下/etc/profile文件，让新的环境变量PATH生效

   [atguigu@hadoop102 ~]$ source /etc/profile

   ***\*6\*******\*）\*******\*测试JDK\*******\*是否\*******\*安装成功\****

   [atguigu@hadoop102 ~]$ java -version

   如果能看到以下结果，则代表Java安装成功。

   java version "1.8.0_212"

3. ### 编写集群分发脚本xsync

   ***\*1）scp（\*******\*secure copy\*******\*）\*******\*安全\*******\*拷贝\****

   （1）scp定义

   scp可以实现服务器与服务器之间的数据拷贝。（from server1 to server2）

   （2）基本语法

   scp  -r     $pdir/$fname       $user@$host:$pdir/$fname

   命令  递归   要拷贝的文件路径/名称  目的地用户@主机:目的地路径/名称

   （3）案例实操

   Ø 前提：在hadoop102、hadoop103、hadoop104都已经创建好的/opt/module、      /opt/software两个目录，并且已经把这两个目录修改为atguigu:atguigu

   [atguigu@hadoop102 ~]$ sudo chown atguigu:atguigu -R /opt/module

   （a）在hadoop102上，将hadoop102中/opt/module/jdk1.8.0_212目录拷贝到hadoop103上。

   [atguigu@hadoop102 ~]$ scp -r /opt/module/jdk1.8.0_212 atguigu@hadoop103:/opt/module

   （b）在hadoop103上，将hadoop102中/opt/module/hadoop-3.1.3目录拷贝到hadoop103上。

   [atguigu@hadoop103 ~]$ scp -r atguigu@hadoop102:/opt/module/hadoop-3.1.3 /opt/module/

   （c）在hadoop103上操作，将hadoop102中/opt/module目录下所有目录拷贝到hadoop104上。

   [atguigu@hadoop103 opt]$ scp -r atguigu@hadoop102:/opt/module/* atguigu@hadoop104:/opt/module

   ***\*2）rsync远程\*******\*同步\*******\*工具\****

   rsync主要用于备份和镜像。具有速度快、避免复制相同内容和支持符号链接的优点。

   rsync和scp区别：用rsync做文件的复制要比scp的速度快，rsync只对差异文件做更新。scp是把所有文件都复制过去。

   ​	（1）基本语法

   rsync   -av    $pdir/$fname       $user@$host:$pdir/$fname

   命令  选项参数  要拷贝的文件路径/名称  目的地用户@主机:目的地路径/名称

   ​	 选项参数说明

   | 选项 | 功能         |
   | ---- | ------------ |
   | -a   | 归档拷贝     |
   | -v   | 显示复制过程 |

   （2）案例实操

   ​	（a）删除hadoop103中/opt/module/hadoop-3.1.3/wcinput

   [atguigu@hadoop103 hadoop-3.1.3]$ rm -rf wcinput/

   ​	（b）同步hadoop102中的/opt/module/hadoop-3.1.3到hadoop103

   [atguigu@hadoop102 module]$ rsync -av hadoop-3.1.3/ atguigu@hadoop103:/opt/module/hadoop-3.1.3/

   ***\*3）\*******\*xsync集群分发\*******\*脚本\****

   （1）需求：循环复制文件到所有节点的相同目录下

   ​	（2）需求分析：

   （a）rsync命令原始拷贝：

   rsync  -av   /opt/module  	 atguigu@hadoop103:/opt/

   （b）期望脚本：

   xsync要同步的文件名称

   （c）期望脚本在任何路径都能使用（脚本放在声明了全局环境变量的路径）

   [atguigu@hadoop102 ~]$ echo $PATH

   /usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/atguigu/.local/bin:/home/atguigu/bin:/opt/module/jdk1.8.0_212/bin

   （3）脚本实现

   （a）在/home/atguigu/bin目录下创建xsync文件

   [atguigu@hadoop102 opt]$ cd /home/atguigu

   [atguigu@hadoop102 ~]$ mkdir bin

   [atguigu@hadoop102 ~]$ cd bin

   [atguigu@hadoop102 bin]$ vim xsync

   在该文件中编写如下代码

   ```
   #
   #!/bin/bash
   
   #1. 判断参数个数
   if [ $# -lt 1 ]
   then
       echo Not Enough Arguement!
       exit;
   fi
   #2. 遍历集群所有机器
   for host in centos1 centos2 centos4
   do
       echo ====================  $host  ====================
       #3. 遍历所有目录，挨个发送
   
       for file in $@
       do
           #4. 判断文件是否存在
           if [ -e $file ]
               then
                   #5. 获取父目录
                   pdir=$(cd -P $(dirname $file); pwd)
   
                   #6. 获取当前文件的名称
                   fname=$(basename $file)
                   ssh $host "mkdir -p $pdir"
                   rsync -av $pdir/$fname $host:$pdir
               else
                   echo $file does not exists!
           fi
       done
   done
   
   
   ```
   
   
   
   （b）修改脚本 xsync 具有执行权限
   
   [atguigu@hadoop102 bin]$ chmod +x xsync
   
   （c）测试脚本
   
   [atguigu@hadoop102 ~]$ xsync /home/atguigu/bin
   
   （d）将脚本复制到/bin中，以便全局调用
   
   [atguigu@hadoop102 bin]$ sudo cp xsync /bin/
   
   （e）同步环境变量配置（root所有者）
   
   [atguigu@hadoop102 ~]$ sudo ./bin/xsync /etc/profile.d/my_env.sh
   
   注意：如果用了sudo，那么xsync一定要给它的路径补全。
   
   让环境变量生效
   
   [atguigu@hadoop103 bin]$ source /etc/profile
   
   [atguigu@hadoop104 opt]$ source /etc/profile
   
4. ### SSH无密登录配置

   ***\*1\*******\*）\*******\*配置ssh\****

   （1）基本语法

   ssh另一台电脑的IP地址

   （2）ssh连接时出现Host key verification failed的解决方法

   [atguigu@hadoop102 ~]$ ssh hadoop103

   Ø 如果出现如下内容

   Are you sure you want to continue connecting (yes/no)? 

   Ø 输入yes，并回车

   （3）退回到hadoop102

   [atguigu@hadoop103 ~]$ exit

   ***\*2\*******\*）\*******\*无密钥配置\****

   （1）免密登录原理 

   ![img](/images/sshprinciple.png)

   （2）生成公钥和私钥

   [atguigu@hadoop102 .ssh]$ pwd

   /home/atguigu/.ssh

    

   [atguigu@hadoop102 .ssh]$ ssh-keygen -t rsa

   然后敲（三个回车），就会生成两个文件id_rsa（私钥）、id_rsa.pub（公钥）

   （3）将公钥拷贝到要免密登录的目标机器上

   [atguigu@hadoop102 .ssh]$ ssh-copy-id hadoop102

   [atguigu@hadoop102 .ssh]$ ssh-copy-id hadoop103

   [atguigu@hadoop102 .ssh]$ ssh-copy-id hadoop104

   注意：

   还需要在hadoop103上采用atguigu账号配置一下无密登录到hadoop102、hadoop103、hadoop104服务器上。

   还需要在hadoop104上采用atguigu账号配置一下无密登录到hadoop102、hadoop103、hadoop104服务器上。

   还需要在hadoop102上采用root账号，配置一下无密登录到hadoop102、hadoop103、hadoop104；

   ***\*3\*******\*）\*******\*.ssh文件夹下\*******\*（~/.ssh）\*******\*的文件功能解释\****

   | known_hosts     | 记录ssh访问过计算机的公钥（public key） |
   | --------------- | --------------------------------------- |
   | id_rsa          | 生成的私钥                              |
   | id_rsa.pub      | 生成的公钥                              |
   | authorized_keys | 存放授权过的无密登录服务器公钥          |

---

> 作者: wang  
> URL: https://codingroam.github.io/post/linux%E5%B8%B8%E7%94%A8%E8%AE%BE%E7%BD%AE%E5%91%BD%E4%BB%A4%E6%9D%82%E8%AE%B0/  

