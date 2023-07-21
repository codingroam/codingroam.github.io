# navicat连接docker环境mysql8.0报1251


navicat连接docker环境mysql8.0报1251

<!--more-->

```
docker exec -it mysql bash
mysql -uroot -p
#输入密码

#进入到mysql
use mysql;
select host, user, authentication_string, plugin from user;
 
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
(设置远程登录密码123456,密码别设置的太简单，否则知道你服务器ip就随便登录你的mysql了，不然就给服务器3306端口加上来源限制，真的血的教训)

// 设置远程连接权限
grant all on *.* to 'root'@'%';

//刷新权限
flush privileges;

```



---

> 作者: wang  
> URL: https://codingroam.github.io/post/navicat%E8%BF%9E%E6%8E%A5docker%E7%8E%AF%E5%A2%83mysql8.0%E6%8A%A51251/  

