# Git设置SSH方式


Linux环境Git上传出现：The requested URL returned error: 403解决办法小记

<!--more-->



## Git设置SSH方式

### 一、Gitee场景

​		阿里云环境push代码失败，代码托管为Gitee。切换https方式到ssh后提交成功，方式如下：

#### 1、在linux控制台(powershell也是如此)生成sshkey

1. $ ssh-keygen -t rsa -C "xxxxx@xxxxx.com"
1. 按照提示完成三次回车，即可生成 ssh key
1. cat ~/.ssh/id_rsa.pub <!--查看生成的public key并复制-->

#### 2、将ssh key添加到gitee仓库

1. 通过仓库主页 **「管理」->「部署公钥管理」**
1. 点击黄色方框中**添加个人公钥**超链接 <!--如果不把ssh key添加到个人公钥中，git只有查看权限，没有修改权限-->

#### 3、在linux终端输入ssh -T git@gitee.com

> 首次使用需要确认并添加主机到本机SSH可信列表。若返回 `Hi XXX! You've successfully authenticated, but Gitee.com does not provide shell access.` 内容，则证明添加成功。

#### 4、修改原先https的下载方式

1. git config --list 查看配置得到 remote.origin.url=https://gitee.com/wk_acme/git_study.git
1. 修改 git config  remote.origin.url git@gitee.com:wk_acme/git_study.git

#### 5、push代码成功

### 二、Github场景

#### 1、首先需要检查你电脑是否已经有 SSH key 

运行 git Bash 客户端，输入如下代码：

```html
$ cd ~/.ssh
$ ls
```

这两个命令就是检查是否已经存在 id_rsa.pub 或 id_dsa.pub 文件，如果文件已经存在，那么你可以跳过步骤2，直接进入步骤3。

#### 2、创建一个 SSH key 

```html
$ ssh-keygen -t rsa -C "your_email@example.com"
```

代码参数含义：

-t 指定密钥类型，默认是 rsa ，可以省略。
-C 设置注释文字，比如邮箱。
-f 指定密钥文件存储文件名。

以上代码省略了 -f 参数，因此，运行上面那条命令后会让你输入一个文件名，用于保存刚才生成的 SSH key 代码，如：

```html
Generating public/private rsa key pair.
# Enter file in which to save the key (/c/Users/you/.ssh/id_rsa): [Press enter]
```

当然，你也可以不输入文件名，使用默认文件名（推荐），那么就会生成 id_rsa 和 id_rsa.pub 两个秘钥文件。

接着又会提示你输入两次密码（该密码是你push文件的时候要输入的密码，而不是github管理者的密码），

当然，你也可以不输入密码，直接按回车。那么push的时候就不需要输入密码，直接提交到github上了，如：

```html
Enter passphrase (empty for no passphrase): 
# Enter same passphrase again:
```

接下来，就会显示如下代码提示，如：

```html
Your identification has been saved in /c/Users/you/.ssh/id_rsa.
# Your public key has been saved in /c/Users/you/.ssh/id_rsa.pub.
# The key fingerprint is:
# 01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db your_email@example.com
```

当你看到上面这段代码的收，那就说明，你的 SSH key 已经创建成功，你只需要添加到github的SSH key上就可以了。

#### 3、添加你的 SSH key 到 github上面去

1. 首先你需要拷贝 id_rsa.pub 文件的内容，你可以用编辑器打开文件复制，也可以用git命令复制该文件的内容，如：

   ```
   $ clip < ~/.ssh/id_rsa.pub
   ```

　　Window 使用 clip 命令复制，Mac 则使用 pbcopy 命令

2. 登录你的github账号，从又上角的设置（ [Account Settings](https://github.com/settings) ）进入，然后点击菜单栏的 SSH key 进入页面添加 SSH key。

3. 点击 Add SSH key 按钮添加一个 SSH key 。把你复制的 SSH key 代码粘贴到 key 所对应的输入框中，记得 SSH key 代码的前后不要留有空格或者回车。当然，上面的 Title 所对应的输入框你也可以输入一个该 SSH key 显示在 github 上的一个别名。默认的会使用你的邮件名称。

#### 4、测试一下该SSH key

在git Bash 中输入以下代码

```html
$ ssh -T git@github.com
```

当你输入以上代码时，会有一段警告代码，如：

```html
The authenticity of host 'github.com (207.97.227.239)' can't be established.
# RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
# Are you sure you want to continue connecting (yes/no)?
```

这是正常的，你输入 yes 回车既可。如果你创建 SSH key 的时候设置了密码，接下来就会提示你输入密码，如：

```html
Enter passphrase for key '/c/Users/Administrator/.ssh/id_rsa':
```

当然如果你密码输错了，会再要求你输入，知道对了为止。

注意：输入密码时如果输错一个字就会不正确，使用删除键是无法更正的。

密码正确后你会看到下面这段话，如：

```html
Hi username! You've successfully authenticated, but GitHub does not
# provide shell access.
```

成功！

---

> 作者: wang  
> URL: https://codingroam.github.io/zh-cn/post/git%E8%AE%BE%E7%BD%AEssh%E6%96%B9%E5%BC%8F/  

