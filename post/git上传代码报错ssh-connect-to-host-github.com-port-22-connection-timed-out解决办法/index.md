# git上传代码报错ssh: connect to host github.com port 22: Connection timed out解决办法


git上传代码报错ssh: connect to host github.com port 22: Connection timed out解决办法

<!--more-->

### git上传代码报错ssh: connect to host github.com port 22: Connection timed out解决办法

当在远程库上设置了SSH 之后还是报错连接超时，问题如下

```
$ git push 
```

报错：

```
ssh: connect to host github.com port 22: Connection timed out
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

这个时候需要检查一下SSH是否能够连接成功，输入以下命令

```
ssh -T git@github.com
```

稍等片刻如果继续报错，如下：

**ssh: connect to host github.com port 22: Connection timed out***则，可以使用一下解决办法*

打开存放ssh的目录

```
cd ~/.ssh 

ls
```

查看是否存在 id_rsa  id_rsa.pun known_hosts 三个文件，如果没有请查看[Git设置SSH方式](https://codingroam.github.io/post/git%e8%ae%be%e7%bd%aessh%e6%96%b9%e5%bc%8f/)，如果有请按照以下方式设置

**1. vim config (进入到vim编辑界面，如果是windows系统powershell，可以设置使用vim,参考文章[powershell设置vim](https://codingroam.github.io/post/powershell%E8%AE%BE%E7%BD%AE%E4%BD%BF%E7%94%A8vim/)）**

**2. insert 编辑模式**

```
Host github.com
User YourEmail@163.com
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443
```

其中， YourEmail 为绑定的邮箱。 

保存之后再次执行"ssh -T git@github.com"时，会出现如下提示，回车"yes"即可



---

> 作者: wang  
> URL: https://codingroam.github.io/post/git%E4%B8%8A%E4%BC%A0%E4%BB%A3%E7%A0%81%E6%8A%A5%E9%94%99ssh-connect-to-host-github.com-port-22-connection-timed-out%E8%A7%A3%E5%86%B3%E5%8A%9E%E6%B3%95/  

