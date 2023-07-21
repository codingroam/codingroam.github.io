# centos7网卡报错,ip地址丢失不见等问题解决方法

centos7网卡报错,ip地址丢失不见等问题解决方法
<!--more-->

问题出现: 
ip地址丢失不见,网卡报错,当重启网卡时 
Job for network.service failed because the control process exited with error code. See "systemctl status network.service" and "journalctl -xe" for details. 
按照提示输入systemctl status network.service查看报错如下 
Failed to start LSB: Bring up/down networking. 

解决办法: 
系统自带的NetworkManager这个管理套件出错，关掉. 
关掉方法: 
systemctl stop NetworkManager 

systemctl disable NetworkManager 

重新启动网络： systemctl start network.service 





---

> 作者: wang  
> URL: https://codingroam.github.io/zh-cn/post/centos7%E7%BD%91%E5%8D%A1%E6%8A%A5%E9%94%99ip%E5%9C%B0%E5%9D%80%E4%B8%A2%E5%A4%B1%E4%B8%8D%E8%A7%81%E7%AD%89%E9%97%AE%E9%A2%98%E8%A7%A3%E5%86%B3%E6%96%B9%E6%B3%95/  

