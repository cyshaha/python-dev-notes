Git 是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。

## Git 与 SVN 区别

参考：https://www.runoob.com/git/git-tutorial.html

Git 不仅仅是个版本控制系统，它也是个内容管理系统(CMS)，工作管理系统等。

Git 与 SVN 区别点：

- **1、Git 是分布式的，SVN 不是**：这是 Git 和其它非分布式的版本控制系统，例如 SVN，CVS 等，最核心的区别。

- **2、Git 把内容按元数据方式存储，而 SVN 是按文件：**所有的资源控制系统都是把文件的元信息隐藏在一个类似 .svn、.cvs 等的文件夹里。

- **3、Git 分支和 SVN 的分支不同：**分支在 SVN 中一点都不特别，其实它就是版本库中的另外一个目录。

- **4、Git 没有一个全局的版本号，而 SVN 有：**目前为止这是跟 SVN 相比 Git 缺少的最大的一个特征。

- **5、Git 的内容完整性要优于 SVN：**Git 的内容存储使用的是 SHA-1 哈希算法。这能确保代码内容的完整性，确保在遇到磁盘故障和网络问题时降低对版本库的破坏。

  



## 新项目一般流程

1.进入文件夹

进入文件夹，右键git bash here



2.配置名字邮箱

配置一次即可

```shell
git config --global user.name "cys"
git config --global user.email "cys.com"
```



3.初始化

```shell
git init
```



4.查看状态

```shell
git status
#  红色：本地修改增加删除的文件
#  绿色：git已经管理起来
#  生成版本：无色
```



5.添加

```shell
git add 文件名
# 添加之后文件状态变为Changes to be committed
```

```shell
git add .
# 当前status为Untracked files的都提交
```



6.提交

```shell
git commit -m 'v1'
```



7.查看记录

```shell
# 显示所有提交过的版本信息
git log
```

```shell
# 可以查看所有分支的所有操作记录（包括已经被删除的 commit 记录和 reset 的操作）
git reflog
```



## git三大区域

参考：https://www.runoob.com/git/git-workspace-index-repo.html

**工作区**

就是你在电脑里能看到的目录。

**暂存区**

英文叫 stage 或 index。一般存放在 **.git** 目录下的 index 文件（.git/index）中，所以我们把暂存区有时也叫作索引（index）

**版本库**

工作区有一个隐藏目录 **.git**，这个不算工作区，而是 Git 的版本库。

![img](https://www.runoob.com/wp-content/uploads/2015/02/1352126739_7909.jpg)

![image-20210726124006358](C:\Users\cheny\AppData\Roaming\Typora\typora-user-images\image-20210726124006358.png)







## 创建分支

```shell
# 创建dev分支
git branch dev
# 查看分支
git branch
# 切换到dev分支
git checkout dev

# 合并分支（切换到对应主分支操作）
git merge dev
# 删除分支（切换到对应主分支操作）
git branch -d dev
```





## github

配置remote：给远程仓库取别名

```shell
git remote add origin git@github.com:cyshaha/gitproject01.git
```



设置ssh密钥

```shell
ssh-keygen -t rsa -C "cys.com"
```

github上配置公钥id_rsa.pub



向远程推送代码

```shell
git push -u origin master
git push -u origin dev
```



从远程克隆代码

```shell
git clone git@github.com:cyshaha/gitproject01.git
```



从远程拉取代码

```shell
git pull origin dev
# 从origin拉到dev，等同于下面两个命令
git fetch origin dev
git merge origin/dev
```



## git rebase

### 场景一：合并提交记录

第一次修改提交

```shell
git add .
git commit -m 'v1'
```

第二次修改提交

```shell
git add .
git commit -m 'v2'
```

第三次修改提交

```shell
git add .
git commit -m 'v3'
```

合并：

从当前最新提交合并到前三个

```shell
git rebase -i HEAD~2
```

执行后打开一个vim，把v3前面的pick改为s，表示把v3合并到v2

编辑完，:wq，又打开一次vim，写提交信息，:wq保存退出。

然后查看提交信息就少了一条

```shell
git log
```



### 场景二：分支合并记录不分叉

场景如下：

```shell
# 分支创建了文件
git checkout dev
makdir dev1.py
git add .
git commit -m 'dev1.py'
# master也创建了文件
git checkout master
makdir master1.py
git add .
git commit -m 'master1.py'
```

如果直接merge会有分支

```shell
git merge dev
# 查看分支
git log --graph
```

我们使用rebase

```shell
#先切换到dev
git checkout dev
git rebase master
git checkout master
git merge dev
```

这样git log就只有一个分支记录了

```
git log --graph
```



### 场景三：从远程更新代码

以前使用git pull

```shell
git pull 
# 就是
git fetch
git merge
```

现在使用rebase

```shell
git fetch origin/dev
git rebase origin/dev
```

或

```shell
git pull --rebase origin:master
```





# 基本命令

参考：https://www.jianshu.com/p/93318220cdce



## git init

初始化项目所在目录，初始化后会在当前目录下出现一个名为 .git 的目录。



## git config

配置 Git 的相关参数。

```shell
# 查看配置信息
# --local：仓库级，--global：全局级，--system：系统级
$ git config <--local | --global | --system> -l

# 查看当前生效的配置信息
$ git config -l

# 编辑配置文件
# --local：仓库级，--global：全局级，--system：系统级
$ git config <--local | --global | --system> -e

# 添加配置项
# --local：仓库级，--global：全局级，--system：系统级
$ git config <--local | --global | --system> --add <name> <value>

# 获取配置项
$ git config <--local | --global | --system> --get <name>

# 删除配置项
$ git config <--local | --global | --system> --unset <name>

# 配置提交记录中的用户信息
$ git config --global user.name <用户名>
$ git config --global user.email <邮箱地址>
```



## git clone

从远程仓库克隆一个版本库到本地。

```shell
# 默认在当前目录下创建和版本库名相同的文件夹并下载版本到该文件夹下
$ git clone <远程仓库的网址>

# 指定本地仓库的目录
$ git clone <远程仓库的网址> <本地目录>

# -b 指定要克隆的分支，默认是master分支
$ git clone <远程仓库的网址> -b <分支名称> <本地目录>
```



## git status

查看本地仓库的状态。

```shell
# 查看本地仓库的状态
$ git status

# 以简短模式查看本地仓库的状态
# 会显示两列，第一列是文件的状态，第二列是对应的文件
# 文件状态：A 新增，M 修改，D 删除，?? 未添加到Git中
$ git status -s
```



## git add

把要提交的文件的信息添加到暂存区中。当使用 git commit 时，将依据暂存区中的内容来进行文件的提交。

```shell
# 把指定的文件添加到暂存区中
$ git add <文件路径>

# 添加所有修改、已删除的文件到暂存区中
$ git add -u [<文件路径>]
$ git add --update [<文件路径>]

# 添加所有修改、已删除、新增的文件到暂存区中，省略 <文件路径> 即为当前目录
$ git add -A [<文件路径>]
$ git add --all [<文件路径>]

# 查看所有修改、已删除但没有提交的文件，进入一个子命令系统
$ git add -i [<文件路径>]
$ git add --interactive [<文件路径>]
```



## git commit

将暂存区中的文件提交到本地仓库中。

```shell
# 把暂存区中的文件提交到本地仓库，调用文本编辑器输入该次提交的描述信息
$ git commit

# 把暂存区中的文件提交到本地仓库中并添加描述信息
$ git commit -m "<提交的描述信息>"

# 把所有修改、已删除的文件提交到本地仓库中
# 不包括未被版本库跟踪的文件，等同于先调用了 "git add -u"
$ git commit -a -m "<提交的描述信息>"

# 修改上次提交的描述信息
$ git commit --amend
```



## git log

显示提交的记录。

```shell
# 打印所有的提交记录
$ git log

# 打印从第一次提交到指定的提交的记录
$ git log <commit ID>

# 打印指定数量的最新提交的记录
$ git log -<指定的数量>
```



## git remote

操作远程库。

```shell
# 列出已经存在的远程仓库
$ git remote

# 列出远程仓库的详细信息，在别名后面列出URL地址
$ git remote -v
$ git remote --verbose

# 添加远程仓库
$ git remote add <远程仓库的别名> <远程仓库的URL地址>

# 修改远程仓库的别名
$ git remote rename <原远程仓库的别名> <新的别名>

# 删除指定名称的远程仓库
$ git remote remove <远程仓库的别名>

# 修改远程仓库的 URL 地址
$ git remote set-url <远程仓库的别名> <新的远程仓库URL地址>
```



## git branch

操作 Git 的分支命令。

```shell
# 列出本地的所有分支，当前所在分支以 "*" 标出
$ git branch

# 列出本地的所有分支并显示最后一次提交，当前所在分支以 "*" 标出
$ git branch -v

# 创建新分支，新的分支基于上一次提交建立
$ git branch <分支名>

# 修改分支名称
# 如果不指定原分支名称则为当前所在分支
$ git branch -m [<原分支名称>] <新的分支名称>
# 强制修改分支名称
$ git branch -M [<原分支名称>] <新的分支名称>

# 删除指定的本地分支
$ git branch -d <分支名称>

# 强制删除指定的本地分支
$ git branch -D <分支名称>
```



## git checkout

检出命令，用于创建、切换分支等。

```shell
# 切换到已存在的指定分支
$ git checkout <分支名称>

# 创建并切换到指定的分支，保留所有的提交记录
# 等同于 "git branch" 和 "git checkout" 两个命令合并
$ git checkout -b <分支名称>

# 创建并切换到指定的分支，删除所有的提交记录
$ git checkout --orphan <分支名称>

# 从暂存区或版本库拉取代码覆盖本地工作区的修改，如果暂存区有，就从暂存区拉，否则从版本库拉
# 相对上次提交,放弃工作区的修改
$ git checkout -- <文件路径>
```



## git fetch

从远程仓库获取最新的版本到本地的 tmp 分支上。

```shell
# 将远程仓库所有分支的最新版本全部取回到本地
$ git fetch <远程仓库的别名>

# 将远程仓库指定分支的最新版本取回到本地
$ git fetch <远程主机名> <分支名>
```



## git merge

合并分支。

```shell
# 把指定的分支合并到当前所在的分支下
$ git merge <分支名称>
```



## git diff

比较版本之间的差异。

```shell
# 比较当前文件和暂存区中文件的差异，显示没有暂存起来的更改
$ git diff

# 比较暂存区中的文件和上次提交时的差异
$ git diff --cached
$ git diff --staged

# 比较当前文件和上次提交时的差异
$ git diff HEAD

# 查看从指定的版本之后改动的内容
$ git diff <commit ID>

# 比较两个分支之间的差异
$ git diff <分支名称> <分支名称>

# 查看两个分支分开后各自的改动内容
$ git diff <分支名称>...<分支名称>
```



## git pull

从远程仓库获取最新版本并合并到本地。
首先会执行 `git fetch`，然后执行 `git merge`，把获取的分支的 HEAD 合并到当前分支。



## git push

把本地仓库的提交推送到远程仓库。

```shell
# 把本地仓库的分支推送到远程仓库的指定分支
$ git push <远程仓库的别名> <本地分支名>:<远程分支名>

# 删除指定的远程仓库的分支
$ git push <远程仓库的别名> :<远程分支名>
$ git push <远程仓库的别名> --delete <远程分支名>
```



## git reset

还原提交记录。

```shell
# 重置暂存区，但文件不受影响
# 相当于将用 "git add" 命令更新到暂存区的内容撤出暂存区，可以指定文件
# 没有指定 commit ID 则默认为当前 HEAD
$ git reset [<文件路径>]
$ git reset --mixed [<文件路径>]

# 将 HEAD 的指向改变，撤销到指定的提交记录，文件未修改
$ git reset <commit ID>
$ git reset --mixed <commit ID>

# 将 HEAD 的指向改变，撤销到指定的提交记录，文件未修改
# 相当于调用 "git reset --mixed" 命令后又做了一次 "git add"
$ git reset --soft <commit ID>

# 将 HEAD 的指向改变，撤销到指定的提交记录，文件也修改了
$ git reset --hard <commit ID>
```



## git revert

生成一个新的提交来撤销某次提交，此次提交之前的所有提交都会被保留。

```shell
# 生成一个新的提交来撤销某次提交
$ git revert <commit ID>
```



## git tag

操作标签的命令。

```shell
# 打印所有的标签
$ git tag

# 添加轻量标签，指向提交对象的引用，可以指定之前的提交记录
$ git tag <标签名称> [<commit ID>]

# 添加带有描述信息的附注标签，可以指定之前的提交记录
$ git tag -a <标签名称> -m <标签描述信息> [<commit ID>]

# 切换到指定的标签
$ git checkout <标签名称>

# 查看标签的信息
$ git show <标签名称>

# 删除指定的标签
$ git tag -d <标签名称>

# 将指定的标签提交到远程仓库
$ git push <远程仓库的别名> <标签名称>

# 将本地所有的标签全部提交到远程仓库
$ git push <远程仓库的别名> –tags
```





# 一些场景解决

## 1.开发过程需要从远程更新代码

在dev分支，工作区开发了一点东西，但是远程代码也修改了同一文件，工作区的东西还不想丢

1.只在工作区有修改，还没add和commit

```shell
# 从远程拉取代码，会报冲突，解决冲突
git pull origin dev

# 会以下报错
#  Your local changes to the following files would be overwritten by merge:
#        dev.py
#Please commit your changes or stash them before you merge.
```

运行`git-merge`时含有大量的未commit文件很容易让你陷入困境，这将使你在冲突中难以回退。因此非常不鼓励在使用`git-merge`时存在未commit的文件，建议使用`git-stash`命令将这些未commit文件暂存起来，并在解决冲突以后使用`git stash pop`把这些未commit文件还原出来。

解决：

```shell
# 把当前开发的内容加到临时区，保存当前进度，此时工作区的修改内容都没了
git stash
# 查看
git stash list
# 从远程拉取最新代码
git pull origin dev
# 把临时区的内容拉到工作区，有冲突要解决冲突
git stash pop
# 写完之后
git add .
git commit -m 'msg'
# 推送到远程仓库
git push origin dev
# 产看历史
git log --graph
# 不产生分支
```



2.已经add 和 commit了

先pull下来解决冲突，再push上去

```shell
# 先编辑并提交，远程同文件有个dev3-github
vi dev.py
git add .
git commit -m 'dev3-work'

# 拉取最新代码解决冲突
git pull origin dev
vi dev.py
git add .
git commit -m '解决dev3冲突'

# push上去
git push origin dev
```

这样会多一条解决冲突的提交记录

![image-20210805004802959](C:\Users\cheny\AppData\Roaming\Typora\typora-user-images\image-20210805004802959.png)



使用以下方法

```shell
# 先编辑并提交，远程同文件有个dev4-github
vi dev.py
git add .
git commit -m 'dev4-work'

# 拉取的代码到本地仓库
git fetch origin dev
# 与本地工作区代码合并
git rebase origin/dev
# 有冲突解决冲突
vi dev.py
# 继续合并
git rebase --continue
# 提交推送代码
git push origin dev
```

 这样的话提交记录没有合并代码的记录，比较干净

![image-20210805004737355](C:\Users\cheny\AppData\Roaming\Typora\typora-user-images\image-20210805004737355.png)



## 2.把本地代码还原到远程某一版本

本地最新commit的较远程落后一个版本，要舍弃本地的修改，使用远程内容覆盖本地

```shell
git fetch --all
git reset --hard origin/dev
```



## 3.本次新提交覆盖上次提交

先写了一部分代码，已经commit了，但是上次commit的版本想再加点内容，不新commit

```shell
# 上次提交
git add .
git commit -m 'test first'

# 再次写代码提交
git add .
git commit --amend # 会打开编辑提交信息

# 查看历史
git log
```

查看结果‘test first’的提交记录已经没了，被新提交的信息覆盖



## 4.取消暂存区的文件

```shell
git reset HEAD <file>
```



## 5.拉取全新远程分支

本地不存在的分支

```shell
git fetch origin develop（develop为远程仓库的分支名）
git checkout -b dev(本地分支名称) origin/develop(远程分支名称)
git pull origin develop(远程分支名称)
```



