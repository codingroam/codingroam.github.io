# VMware虚拟机CentOS 7.5设置静态ip


CentOS 7.5设置静态ip,配置网关，DNS等

<!--more-->

1. 打开vmware虚拟机界面的编辑->虚拟网络编辑器如下图：

![ipconfig](/images/vmnet8.png)

2. 打开网配置文件 

   #vim /etc/sysconfig/network-scripts/ifcfg-ens33

   ``` 
   DEFROUTE="yes"
   IPV4_FAILURE_FATAL="no"
   IPV6INIT="yes"
   IPV6_AUTOCONF="yes"
   IPV6_DEFROUTE="yes"
   IPV6_FAILURE_FATAL="no"
   IPV6_ADDR_GEN_MODE="stable-privacy"
   NAME="ens33"
   UUID="48aeeb81-96b5-4d08-880a-53e3f527469c"
   DEVICE="ens33"
   ONBOOT="yes"
   BOOTPROTO=static #静态ip获取方式
   IPADDR="192.168.52.104" #静态ip地址
   NETMASK="255.255.255.0" #子网掩码
   GATEWAY="192.168.52.2" #网关
   DNS1="114.114.114.114" #国内移动联通电信网络DNS
   ```

---

> 作者: wang  
> URL: https://codingroam.github.io/post/vmware%E8%99%9A%E6%8B%9F%E6%9C%BAcentos-7.5%E8%AE%BE%E7%BD%AE%E9%9D%99%E6%80%81ip/  

