# 精通Git


精通Git

<!--more-->

## 精通Git

### 1、Git基础

#### （1）三种状态

Git 有三种状态，你的文件可能处于其中之一：已提交（committed）、已修改（modified）和已暂存（staged）。已提交表示数据已经安全的保存在本地数据库中。已修改表示修改了文件，但还没保存到数据库中。已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中

**由此引入 Git 项目的三个工作区域的概念：Git 仓库、工作目录以及暂存区域。**

![image-20221206002243949](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221206002243949.png)

+ Git 仓库目录是 Git 用来保存项目的元数据和对象数据库的地方。这是 Git 中最重要的部分，从其它计算机克隆仓库时，拷贝的就是这里的数据。

+ 工作目录是对项目的某个版本独立提取出来的内容。这些从 Git 仓库的压缩数据库中提取出来的文件，放在磁盘上供你使用或修改。

+ 暂存区域是一个文件，保存了下次将提交的文件列表信息，一般在 Git 仓库目录中。有时候也被称作`‘索引’'，不过一般说法还是叫暂存区域。

  **总结：如果 Git 目录中保存着的特定版本文件，就属于已提交状态。如果作了修改并已放入暂存区域，就属于已暂存状态。如果自上次取出后，作了修改但还没有放到暂存区域，就是已修改状态。**

#### （2）记录数据方式

**Git直接记录快照，而非差异比较**

Git 和其它版本控制系统（包括 Subversion 和近似工具）的主要差别在于 Git 对待数据的方法。概念上来区分，其它大部分系统以文件变更列表的方式存储信息。这类系统（CVS、Subversion、Perforce、Bazaar 等等）将它们保存的信息看作是一组基本文件和每个文件随时间逐步累积的差异。

![image-20221206004811919](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221206004811919.png)

Git 不按照以上方式对待或保存数据。反之，Git 更像是把数据看作是对小型文件系统的一组快照。每次你提交

更新，或在 Git 中保存项目状态时，它主要对当时的全部文件制作一个快照并保存这个快照的索引。为了高效，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。Git 对待数据更像是一个 **快照流**。

![image-20221206005030224](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221206005030224.png)

**Git 更像是一个小型的文件系统，提供了许多以此为基础构建的超强工具，而不只是一个简单的 VCS**

#### （3）**近乎所有操作都是本地执行**和保证文件完整性**

### 2、配置信息

​		--global 选项，那么该命令只需要运行一次，因为之后无论你在该系统上做任何事情， Git 都会使用那些信息。当你想针对特定项目使用不同的用户名称与邮件地址时，可以在那个项目目录下运行没有 --global 选项的命令来配置

1. **git config --global user.name "kw"**

1. **git config --global user.email 1121034507@qq.com**
1. **git config --list** `<!--命令来列出所有 Git 当时能找到的配置-->`
1. **git config --globalcredential.helper cache/store**  `缓存用户名/密码，cache-15分钟 store-永久`
1. **配置Git可以用git config --list查看配置项，再使用命令git config 配置项 设置值**

### **3、git基础操作**

1. **git init** `<!--在当前目录初始化git-->`

1. **$ git add *.c ** `<!--暂存命令，添加一个或多个文件到暂存区（开始跟踪一个新文件，或者把已跟踪的文件放到暂存区，概括起来就是添加内容到下一次提交中）-->`

   **$ git add LICENSE**

   **$ git commit -m 'initial project version'**  `<!--提交命令,将暂存区文件提交到本地仓库-->`

1. **git clone https://github.com/libgit2/libgit2  [别名]**`<!--克隆远程仓库-->`

1. **git status **`<!--检查当前文件状态-->`

   **$ git status -s**

      **M README**   `<!--右边的 M 表示该文件被修改了但是还没放入暂存区-->`

   **M    lib/simplegit.rb** `<!--左边的 M文件被修改了并将修改后的文件放入了暂存区-->`

   **MM Rakefile**  `<!--在工作区被修改并提交到暂存区后又在工作区中被修改了，所以在暂存区和工作区都有该文件被修改了的记录-->`

   **A    lib/git.rb ** `<!--新添加到暂存区中的文件前面有 A 标记，修改过的文件前面有 M 标记-->`

   **??   LICENSE.txt**  `<!--新添加的未跟踪文件前面有 ?? 标记-->`

1. **git diff** `<!--**查看已暂存和未暂存的修改**-->`

   1. **要查看尚未暂存的文件更新了哪些部分，不加参数直接输入 git diff：**
   1. **要查看已暂存的将要添加到下次提交里的内容，可以用 git diff --cached/--staged**

1. **git commit  -m "message"**  `<!--将暂存区文件提交到本地仓库-->`

   **git commit -a -m "message"** `<!--将已追踪文件提交到本地仓库，省略git add .操作-->`

1. **git rm 文件名 **`<!--从已跟踪文件清单中移除（确切地说，是从暂存区域移除），然后提交。该命令连带从工作目录中删除文件-->`

   **git rm --cached README**`<!--从本地仓库删除，当然也包括暂存区和工作目录-->`

   **git rm log/\\*.log  **`<!--删除 log/ 目录下扩展名为 .log 的所有文件。星号 * 之前的反斜杠 \，因为 Git 有它自己的文件模式扩展匹配方式-->`

   **git rm \\*~** `<!--删除以 ~ 结尾的所有文件-->`

1. **git mv file_from file_to**  `<!--改名操作-->`

1. **git log **`<!--查看提交历史-->`

   **git log -p -2**  `<!--（-p）查看提交差异，2表示最近两次提交-->**`

   **git log --stat **`<!--简略信息-->**`

   **git log --pretty=format:"%h - %an, %ar : %s" ** `<!-- --pretty。这个选项可以指定使用不同于默认格式的方式展示提交历史-->`

   ```
   ca82a6d - Scott Chacon, 6 years ago : changed the version number
   085bb3b - Scott Chacon, 6 years ago : removed unnecessary test
   a11bef0 - Scott Chacon, 6 years ago : first commit
   ```

   ​		**用 --author 选项显示指定作者的提交，用 --grep 选项搜索提交说明中的关键字。（请注意，如果要得到同时满足这两个选项搜索条件的提交，就必须用 --all-match 选项。否则，满足任意一个条件的提交都会被匹配出来）。另一个非常有用的筛选选项是 -S，可以列出那些添加或移除了某些字符串的提交。比如说，你想找出添加或移除了某一个特定函数的引用的提交，你可以这样使用： $ git log -Sfunction_name**

​		

| 选项              | 说明                               |
| ----------------- | ---------------------------------- |
| -(n)              | 仅显示最近的 n 条提交              |
| --since, --after  | 仅显示指定时间之后的提交。         |
| --until, --before | 仅显示指定时间之前的提交           |
| --author          | 仅显示指定作者相关的提交。         |
| --committer       | 仅显示指定提交者相关的提交。       |
| --grep            | 仅显示含指定关键字的提交           |
| -S                | 仅显示添加或移除了某个关键字的提交 |

10. **git tag `查看标签`**

    1. git tag -a v1.4 -m 'my version 1.4' `增加标签V1.4`

    1. git tag tagname `轻量标签`

       ```
       $ git log --pretty=oneline
       15027957951b64cf874c3557a0f3547bd83b3ff6 Merge branch 'experiment'
       a6b4c97498bd301d84096da251c98a07c7723e65 beginning write support
       0d52aaab4479697da7686c15f77a3d64d9165190 one more thing
       $ git tag -a v1.2 0d52aaa //对历史提交打标签，如果不加提交校验和会对在最近一次的提交打tag
       ```

    1. git push origin [tagname] `推送tag到远程仓库（git push不会推送tag）`

    1. git push origin --tags `推送所有标签`

    1. git checkout -b [branchname] [tagname]  `在特定的标签上创建一个新分支`

       Switched to a new branch 'version2'

10. **git config `设置别名`**

    ```
    $ git config --global alias.co checkout
    $ git config --global alias.br branch
    $ git config --global alias.ci commit
    $ git config --global alias.st status
    ```

    

### 4、Git分支

​		Git 保存的不是文件的变化或者差异，而是一系列不同时刻的文件快照。Git 的分支，其实本质上仅仅是指向提交对象的可变指针。Git 的默认分支名字是 master。在多次提交操作之后，其实已经有一个指向最后那个提交对象的 master 分支。它会在每次的提交操作中自动向前移动。Git 又是怎么知道当前在哪一个分支上呢？也很简单，它有一个名为 HEAD 的特殊指针

![Snipaste_2022-11-30_14-39-01](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/Snipaste_2022-11-30_14-39-01.png)

` git branch testing 创建分支testing`

![image-20221130144713445](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221130144713445.png)

`git checkout testing 切换分支`

![image-20221130144813890](C:/Users/21036/AppData/Roaming/Typora/typora-user-images/image-20221130144813890.png)

```
$ vim test.txt
$ git commit -a -m 'made a change'
```

![image-20221130144945977](C:/Users/21036/AppData/Roaming/Typora/typora-user-images/image-20221130144945977.png)

 		如图所示，testing 分支向前移动了，但是 master 分支却没有，它仍然指向运行 git checkout 时所指的对象。分支切换会改变你工作目录中的文件在切换分支时，一定要注意你工作目录里的文件会被改变。如果是切换到一个较旧的分支，你的工作目录会恢复到该分支最后一次提交时的样子。如果 Git 不能干净利落地完成这个任务，它将禁止切换分支，要留意工作目录和暂存区里那些还没有被提交的修改，它可能会和你即将检出的分支产生冲突从而阻止 Git 切换到该分支（Git 会自动添加、删除、修改文件以确保此时你的工作目录和这个分支最后一次提交时的样子一模一样）

1. **git checkout -b iss53 `新建并检出分支iss53`**

   ```
   //等于以下两条命令
   $ git branch iss53
   $ git checkout iss53
   ```

   ​		从一个远程跟踪分支检出一个本地分支会自动创建一个叫做 “跟踪分支”（有时候也叫做 “上游分支”）。跟踪分支是与远程分支有直接关系的本地分支。如果在一个跟踪分支上输入 git pull，Git 能自动地识别去哪个服务器上抓取、合并到哪个分支。

   ​		当克隆一个仓库时，它通常会自动地创建一个跟踪 origin/master 的 master 分支。然而，如果你愿意的话可以设置其他的跟踪分支 - 其他远程仓库上的跟踪分支，或者不跟踪 master 分支。最简单的就是之前看到的例子，运行 git checkout -b [branch]   [remotename]/[branch]。这是一个十分常用的操作所以 Git 提供了 --track 快捷方式：

   ```
   $ git checkout --track origin/serverfix
   Branch serverfix set up to track remote branch serverfix from origin.
   Switched to a new branch 'serverfix'
   ```

   如果想要将本地分支与远程分支设置为不同名字，你可以轻松地增加一个不同名字的本地分支的上一个命令：

   ```
   $ git checkout -b sf origin/serverfix
   Branch sf set up to track remote branch serverfix from origin.
   Switched to a new branch 'sf'
   ```

   现在，本地分支 sf 会自动从 origin/serverfix 拉取。

   

   设置已有的本地分支跟踪一个刚刚拉取下来的远程分支，或者想要修改正在跟踪的上游分支，你可以在任意时间使用 -u 或 --set-upstream-to 选项运行 git branch 来显式地设置。

   ```
   $ git branch -u origin/serverfix
   Branch serverfix set up to track remote branch serverfix from origin.
   ```

   

1. **git merge `合并分支`**

   ```
   $ git checkout master //检出master分支
   $ git merge hotfix   //合并testing分支到master
   ```

   > 遇到冲突时的分支合并：**如果在两个不同的分支中，对同一个文件的同一个部分进行了不同的修改，Git 就没法干净的合并它们，这就是合并冲突**

​		

```
    $ git merge iss53
    Auto-merging index.html
    CONFLICT (content): Merge conflict in index.html
    Automatic merge failed; fix conflicts and then commit the result.

    //查看具体冲突信息
    $ git status
    On branch master
    You have unmerged paths.
      (fix conflicts and run "git commit")
    Unmerged paths:
      (use "git add <file>..." to mark resolution)
        both modified: index.html
    no changes added to commit (use "git add" and/or "git commit -a")
    
    //解决冲突
    手动修改冲突文件后，对每个文件使用 git add 命令来将其标记为冲突已解决
    $ git add index.html
    
    //查看状态，已无冲突
    $ git status
    On branch master
    All conflicts fixed but you are still merging.
      (use "git commit" to conclude merge)
    Changes to be committed:
        modified: index.html
     
     //合并提交
     $ git commit 使用git commit提交合并
     
```

3. **git branch  `分支管理，不加任务参数情况下显示所有分支列表`**

   ```
   $ git branch
     iss53
   * master //当前检出分支
     testing
   ```

   1. git branch -v  `要查看每一个分支的最后一次提交`

      ```
      $ git branch -v
        iss53 93b412c fix javascript issue
      * master 7a98805 Merge branch 'iss53'
        testing 782fd34 add scott to the author list in the readmes
      ```

   1. git branch -vv `查看设置的所有跟踪分支`

      ```
      $ git branch -vv
        iss53 7e424c3 [origin/iss53: ahead 2] forgot the brackets
        master 1ae2a45 [origin/master] deploying index fix
      * serverfix f8674d9 [teamone/server-fix-good: ahead 3, behind 1] this
      should do it
        testing 5ea463a trying something new
      ```

      ​        这里可以看到 iss53 分支正在跟踪 origin/iss53 并且 “ahead” 是 2，意味着本地有两个提交还没有推送到服务器上。也能看到 master 分支正在跟踪 origin/master 分支并且是最新的。接下来可以看到serverfix 分支正在跟踪 teamone 服务器上的 server-fix-good 分支并且领先 2 落后 1，意味着服务器上有一次提交还没有合并入同时本地有三次提交还没有推送。最后看到 testing 分支并没有跟踪任何远程分支。

      ​		需要重点注意的一点是这些数字的值来自于你从每个服务器上最后一次抓取的数据。这个命令并没有连接服务器，它只会告诉你关于本地缓存的服务器数据。如果想要统计最新的领先与落后数字，需要在运行此命令前抓取所有的远程仓库。可以像这样做：$ git fetch --all; git branch -vv

   1. git branch [--no-merged/--merged] `查看已经合并或尚未合并到当前分支的分支`

   1. git branch -d branchname  `删除分支 -d无法删除未合并的分支，-D可以强制删除`

3. **远程分支 ` git ls-remote (remote)或者git remote show (remote)来显示远程分支列表或者详细信息`**

   ​		Git 的 clone 命令会为你自动将其命名为 origin，拉取它的所有数据，创建一个指向它的 master 分支的指针，并且在本地将命名为 origin/master。Git 也会给你一个与 origin 的 master 分支在指向同一个地方的本地 master 分支，这样你就有工作的基础

   ​	远程仓库名字 “origin” 与分支名字 “master” 一样，在 Git 中并没有任何特别的含义一样。同时 “master” 是当你运行 git init 时默认的起始分支名字，原因仅仅是它的广泛使用，“origin” 是当你运行 git clone 时默认的远程仓库名字。如果你运行 git clone -o booyah，那么你默认的远程分支名字将会是 booyah/master

​		![image-20221130153527248](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221130153527248.png)	

​		如果你在本地的 master 分支做了一些工作，然而在同一时间，其他人推送提交到 git.ourcompany.com 并更新了它的 master 分支，那么你的提交历史将向不同的方向前进。也许，只要你不与 origin 服务器连接，你的 origin/master 指针就不会移动。

![image-20221130180608920](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221130180608920.png)

​		` git fetch origin  在进行同步工作工作时，从远程仓库抓取本地没有的数据，更新本地数据库并将origin/master向后移动 `

​			

![image-20221130181126694](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221130181126694.png)

5. **git remote add `添加远程仓库分支`**

   1. git remote add 别名 远程仓库

      ![image-20221202111147742](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221202111147742.png)

      

5. **git fetch `拉取远程仓库`**

   ​		当 git fetch 命令从服务器上抓取本地没有的数据时，它并不会修改工作目录中的内容。它只会获取数据然后让你自己合并。然而，有一个命令叫作 git pull 在大多数情况下它的含义是一个 git fetch 紧接着一个git merge 命令。如果有一个像之前章节中演示的设置好的跟踪分支，不管它是显式地设置还是通过 clone 或checkout 命令为你创建的，git pull 都会查找当前分支所跟踪的服务器与分支，从服务器上抓取数据然后尝试合并入那个远程分支。

   ​		运行 git fetch teamone 来抓取远程仓库 teamone 有而本地没有的数据。因为那台服务器上现

   有的数据是 origin 服务器上的一个子集，所以 Git 并不会抓取数据而是会设置远程跟踪分支

   teamone/master 指向 teamone 的 master 分支

   ![image-20221202111436324](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221202111436324.png)

5. **git push `推送`**

   1. git push (remote) (branch)

   1. git push --all `推送到所有远程仓库`

   1. git push origin --delete serverfix `删除远程分支serverfix `

      ```
      $ git push origin --delete serverfix
      To https://github.com/schacon/simplegit
       - [deleted] serverfix
      ```

      

5. **变基**

   > 在 Git 中整合来自不同分支的修改主要有两种方法：merge 以及 rebase

​		![image-20221203165953885](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221203165953885.png)

​	    **之前介绍过，整合分支最容易的方法是 merge 命令。它会把两个分支的最新快照（C3 和 C4）以及二者最近的共同祖先（C2）进行三方合并，合并的结果是生成一个新的快照（并提交）。**

![image-20221203170205241](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221203170205241.png)

**使用rebase也可以达到目的，提取在 C4 中引入的补丁和修改，然后在 C3 的基础上再应用一次。在 Git 中，这种操作就叫做 变基。你可以使用 rebase 命令将提交到某一分支上的所有修改都移至另一分支上，就好像“重新播放”一样**

```
$ git checkout experiment
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: added staged command
```

**它的原理是首先找到这两个分支（即当前分支 experiment、变基操作的目标基底分支 master）的最近共同祖先 C2，然后对比当前分支相对于该祖先的历次提交，提取相应的修改并存为临时文件，然后将当前分支指向目标基底 C3, 最后以此将之前另存为临时文件的修改依序应用。（译注：写明了 commit id，以便理解，下同）**

![image-20221203170813182](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221203170813182.png)

**现在回到 master 分支，进行一次快进合并。**

```
$ git checkout master
$ git merge experiment
```

![image-20221203170957245](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221203170957245.png)

​       此时，C4' 指向的快照就和上面使用 merge 命令的例子中 C5 指向的快照一模一样了。这两种整合方法的最终结果没有任何区别，但是变基使得提交历史更加整洁。你在查看一个经过变基的分支的历史记录时会发现，尽管实际的开发工作是并行的，但它们看上去就像是先后串行的一样，提交历史是一条直线没有分叉。

​       一般我们这样做的目的是为了确保在向远程分支推送时能保持提交历史的整洁——例如向某个别人维护的项目贡献代码时。在这种情况下，你首先在自己的分支里进行开发，当开发完成时你需要先将你的代码变基到origin/master 上，然后再向主项目提交修改。这样的话，该项目的维护者就不再需要进行整合工作，只需要快进合并便可。

​       请注意，无论是通过变基，还是通过三方合并，整合的最终结果所指向的快照始终是一样的，只不过提交历史不同罢了。变基是将一系列提交按照原有次序依次应用到另一分支上，而合并是把最终结果合在一起。

**变基的风险**

呃，奇妙的变基也并非完美无缺，要用它得遵守一条准则：**不要对在你的仓库外有副本的分支执行变基。**


---

> 作者: wang  
> URL: https://codingroam.github.io/zh-cn/post/%E7%B2%BE%E9%80%9Agit/  

