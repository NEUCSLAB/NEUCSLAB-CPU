# Git用法

## intulit mortem  
你是否有遇到过这样的问题：好不容易写完的工程，被告知有需要更改的功能，只能复制粘贴一个"xx版"工程继续写；修bug时发现改了这一处bug导致了其他功能出问题，想回滚Ctrl+Z又找不到位置...这时你多么希望你的生活有一个存档点，只要在这里存一个档，之后出了啥错只要读这个挡就能回到啥也没发生的时间点.  

当然，人生没有重来键，还好软件工程有，它就是大名鼎鼎的版本控制系统(Version Control System, VCS). 版本控制系统给你的开发流程提供了比 RPG 更加强大的 SL 功能, 能够让你在过去和未来中随意穿梭, 避免上文中的悲剧降临你的身上.  

没听说过版本控制系统就完成课设, 艰辛地排除万难, 就像游戏通关之后才知道原来游戏可以存档一样, 其实玩游戏的时候进行存档并不是什么丢人的事情.

在实验中, 我们使用 Git 进行版本控制. 

## 了解SL法则
Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件.  
    
    是的你没看错就是那个写 Linux 的 Linus.  

Git 可以跟踪你的文件修改，为你的工程提供多个存档点以供随时读取，还可以对比不同存档点之间的文件差异，管理不同的开发路径等等.  

要实现这些，我们得先安装 Git.  

### Windows
Git 可以在官网获取安装包  

    https://gitforwindows.org/  

如果官网速度慢可以从国内镜像获取  

    https://npm.taobao.org/mirrors/git-for-windows/   

安装后即可. 一般我们使用的是命令行的 Git 工具，可以在资源管理器内右键->Git Bash Here打开.  后续的操作以Windows平台为例，使用其他平台的同学需要自行学习使用方法.

### Linux
根据你所使用的安装包管理工具选择命令即可，通常软件源内已经包含 Git，无需添加仓库

#### Debian/Ubuntu
    apt-get install git

#### CentOS/RedHat
    yum -y install git

### Mac  

本实验不建议使用 Mac ，因为之后使用的 Vivado 软件是不支持 MacOS 的. 如果想使用类 unix 环境，请使用 x86 平台的 Linux 系统.   

## 构建SL法则  
在安装完成后，我们需要配置一下 Git. 打开 Git Bash 或是命令提示符/PowerShell/Windows Terminal等等, 输入以下命令
```bash
git config --global user.name "Zhang San"        # your name
git config --global user.email "zhangsan@foo.com"    # your email
git config --global core.editor vim            # your favourite editor
git config --global color.ui true
```
这里配置的用户名和邮箱会记录在每次存档的信息里，注意修改为自己的信息. 我们不强制用户名和邮箱修改为学校内的信息，你可以按照你的上网习惯进行配置. 

之后，我们就可以开始使用 Git 了.  

首先我们需要让 Git 知道它需要跟踪的目录. 将终端中的路径切换至你的工程目录下，输入  

    git init

Git 会在这个目录下建立它的跟踪文件，之后输入  

    git add .

Git 会开始跟踪目录下的所有文件.  

如果你不想让Git跟踪某个文件，可以输入以下命令  

    git rm <filename>

## 在世界的中心呼唤Save
即使 Git 已经开始跟踪你的工作，但你可以像以前一样编写代码. 等到你的开发取得了一些阶段性成果, 你应该马上进行"存档".  

首先你需要使用 git status 查看是否有新的文件或已修改的文件未被跟踪, 若有, 则使用 git add 将文件加入跟踪列表  

    git add <filename>

会将`<filename>`加入跟踪列表. 如果需要一次添加所有未被跟踪的文件, 你可以使用  

    git add -A

大多数时候，这个命令会将大量编译中间文件添加进跟踪列表，你需要编辑`.gitignore`文件进行筛选，或放弃这个命令手动添加. 对于本实验，Vivado会自己管理中间文件，因此不存在这个问题.   

把新文件加入跟踪列表后, 使用`git status`再次确认. 确认无误后就可以存档了, 使用
```
git commit
```
提交工程当前的状态. 执行这条命令后, 将会弹出文本编辑器, 你需要在第一行中添加本次存档的注释, 例如"fix bug for xxx". 你应该尽可能添加详细的注释, 将来你需要根据这些注释来区别不同的存档. 编写好注释之后, 保存并退出文本编辑器, 存档成功.   

需要注意，默认的文本编辑器是VIM，你需要按<kbd>i</kbd>进入输入模式，输入完内容后按<kbd>esc</kbd>回到一般模式，再按<kbd>:wq</kbd>进行保存. 以上命令的具体内容可以参考VIM的入门教程.  

如果你不喜欢每次存档都要这么输入一次注释，可以使用`-m`参数直接完成注释
```
git commit -m "fix bug for xxx"
```

你可以使用`git log`查看存档记录, 你应该能看到刚才编辑的注释.  

## 检查存档点
使用  

    git log

查看目前为止所有的存档.

使用  

    git status

可以得知, 与当前存档相比, 哪些文件发生了变化.

## I need LOAD!
好了，现在你遇到问题了，你想回档了，你想起之前你有commit过，你应该怎么做呢？  

首先使用`git log`查看目前为止所有的存档, 确认你需要读取哪个存档，每一个存档都有一个hash code用于辨识，如`9c7a0e9f6e977d17d624bc4ebc986c67f98c4e0f`. 你需要通过这个hash code来告诉 Git 你要读哪个档
```
git reset --hard 9c7a
```
只需要能标识出存档的前缀，Git 就能帮你回到过去，但是需要注意，一但回到过去，那么比这个档新的所有存档都将被删除，因此每次回档都需要反复确认，因为你无法再次回到未来. 

## Another View
OK，假设你现在需要写新功能了，但你不知道你的代码会不会破坏原有的系统. 你想存个档，但又想保留开发过程中的信息，因此你忍不住弄了一个副本开始写...  

Stop, Git 提供了一个功能让你管理不同类型的存档点，这就是分支功能. 通过  
```
git branch
```
查看目前所有的分支信息.   

当 Git 初始化的时候，默认会创建一个`master`分支作为主分支，我们可以通过
```
git branch <branch-name>
```
来创建新的分支，之后通过
```
git checkout <branch-name>
```
来切换至不同的分支. 如果使用上一节中的hash code作为分支名，Git 会创建一个虚拟分支展示该存档的内容，你需要通过`git checkout -b <branch-name>`来将这个分支保存.  

当然，我们也可以通过
```
git checkout -b <branch-name>
```
来立刻创建分支. 

不同的分支是相互独立的，你所做的任何修改都不会相互影响，你可以对不同的分支进行commit，区分不同的开发方向. 现在，你可以在代码的平行世界里穿梭了.   

当然，我们最后提交的程序不可能是一个分支实现一个功能的状态，因此我们需要将分支合并起来. 首先切换到想要合并的主分支  
```
git checkout <destination-branch>
```
然后将目标分支合并
```
git merge <source-branch>
```
如果你需要删除某个分支
```
git branch -d <branch-name>
```

## Internet Overdose
Git 是一个分布式的版本管理系统，这意味着它可以通过互联网进行文件同步. 网上有很多基于 Git 的代码托管平台，最大最出名的可能就是 Github 了. 我们非常推荐你在开发时将代码上传至 Github，这样我们可以更方便地追踪你的开发进度，也更有利与小组内同步代码.  

首先你需要注册一个Github账号，千万不要使用学校的邮箱进行注册，学校邮箱会在毕业后收回，除非你确认这个Github账号是一次性账号，不然不要使用学校邮箱注册.  

由于你的本地 Git 仓库和 GitHub 仓库之间的传输是通过SSH加密的，所以我们需要配置验证信息. 使用以下命令生成 SSH Key
```
ssh-keygen -t rsa -C "youremail@example.com"
```
这个邮箱和你Git配置的邮箱保持一致. 之后会提示你输入保存路径和密码，直接使用默认的即可
```bash
$ ssh-keygen -t rsa -C "youremail@example.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/<username>/.ssh/id_rsa):  #直接回车
Enter passphrase (empty for no passphrase): #直接回车
Enter same passphrase again:    #直接回车
Your identification has been saved in /c/Users/<username>/.ssh/id_rsa
Your public key has been saved in /c/Users/<username>/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:prbMDNk<............>FiR1Lns "youremail@example.com"
The key's randomart image is:
+---[RSA 3072]----+
| ... .           |
|o.. +            |
|=..+ .           |
|.+.++            |
|o +..E  S        |
|.++.+o o         |
|o..+.o +          |
|=.B..O .         |
|B...o=          |
+----[SHA256]-----+
```
以上内容已隐去部分信息，根据自己终端输出内容为准. 之后打开`C://Users/<username>/.ssh/id_rsa`，复制里面的内容，这就是本机的ssh公钥.  

打开Github，进入Settings-> SSH and GPG keys,点击New SSH key按钮, title设置标题，可以随便填，key粘贴刚刚复制的ssh公钥，保存即可.  

为了验证是否成功，输入以下命令：
```bash
$ ssh -T git@github.com
The authenticity of host 'github.com (20.205.243.166)' can't be established.
ED25519 key fingerprint is SHA256:<ssh-key>.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? y
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
Hi <username>! You've successfully authenticated, but GitHub does not provide shell access.
```
以下命令说明我们已成功连上 Github.   

在Github上创建一个新仓库后，根据提示的指令即可将本地仓库上传至Github上.
```bash
git remote add [shortname] [url]        #添加远程仓库
git push -u [shortname] [branch-name]   #推送更改
```
如果你需要下载别人的仓库，可以在对应仓库的Github页面中的Clone选单里找到对应的git链接，然后输入
```
git clone <git-url>
```
即可将该仓库下载至本地，包括原开发者的所有commit记录与branch记录.  

## 更多功能
以上仅仅介绍了Git的基础用法，但足够你对付本实验了. 随着你在计算机的路上走的更远，你会逐渐感觉上面的这些功能满足不了你的需求，这时就需要你自己去学习Git的更多高级用法了.  

记住，如果你要学习一个工具，最值得你信赖的只有它的手册，其他所有的教程都要抱着怀疑的态度去看.  

## 参考资料
1. Git help(命令行输入git help)
2. Git manual(命令行输入man git, linux限定)
3. [Git菜鸟教程](https://www.runoob.com/git/git-tutorial.html)