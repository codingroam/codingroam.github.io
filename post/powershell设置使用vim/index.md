# PowerShell设置使用vim


PowerShell设置使用vim

<!--more-->

### PowerShell设置使用vim

1. 如果系统安装了Git，Git本身有自带的vim，位置为%GITHOME%\usr\bin\vim.exe，如果没有安装Git,可以去github下载vim，[Github vim下载](https://github.com/vim/vim-win32-installer/releases)。

1. 打开C:\Windows\System32\WindowsPowerShell\v1.0文件夹新建profile.ps1文件，内容如下，重点是$VIMPATH变量，值为第一步中vim.exe路径

   ```
   # There's usually much more than this in my profile!
   $SCRIPTPATH = "C:\Program Files\Git\usr\share\vim"    # 此行根据$VIMPATH寻找相应vim路径即可,此行不用更改
   $VIMPATH    = "D:\DevelopmentSoftware\Git\usr\bin\vim.exe"  # 此行为1中vim.exe路径
     
   Set-Alias vi   $VIMPATH
   Set-Alias vim  $VIMPATH
     
   # for editing your PowerShell profile
   Function Edit-Profile
   {
       vim $profile
   }
     
   # for editing your Vim settings
   Function Edit-Vimrc
   {
       vim $home\_vimrc
   }
   ```

   

3. 以管理员身份运行PowerShell，运行Set-ExecutionPolicy RemoteSigned，然后输入`Y`即可
3. 重新打开PowerShell即可使用vim命令

---

> 作者: wang  
> URL: https://codingroam.github.io/post/powershell%E8%AE%BE%E7%BD%AE%E4%BD%BF%E7%94%A8vim/  

