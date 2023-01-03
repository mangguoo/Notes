## Git 配置

git的所有配置文件都保存在本地，可以通过git config -l来查看他们，查看不同级别的配置文件：

``` gas
# 查看系统配置
git config --system --list
　　
# 查看当前用户(global)配置
git config --global  --list

# 查看仓库(Private)配置
git config --local --list

# 修改配置
git config --local|global|system alias.co "checkout"
```

Git相关的配置文件：

1. %Git%\etc\gitconfig：   Git 安装目录下的 gitconfig   --system 系统级

   系统配置文件中一般只存储一些公用配置：

   ``` gas
   [core]
   	symlinks = false
   	autocrlf = true
   	fscache = true
   [color]
   	diff = auto
   	status = auto
   	branch = auto
   	interactive = true
   [help]
   	format = html
   [diff "astextplain"]
   	textconv = astextplain
   [rebase]
   	autosquash = true
   [filter "lfs"]
   	clean = git-lfs clean -- %f
   	smudge = git-lfs smudge -- %f
   	process = git-lfs filter-process
   	required = true
   [credential]
   	helper = helper-selector
   ```

2. %HOME%\ .gitconfig：   只适用于当前登录用户的配置  --global 全局

   用户配置文件通常会存放用户私有配置，如name和email等：

   ``` gas
   [user]
   	name = ilmango
   	email = mangguoli42@gmail.com
   [alias]
       co = checkout
       ci = commit
       br = branch
       st = status
   ```

3. %repository%/.git/config：   只适用于当前仓库的配置

   这个文件中一般用于保存仓库的配置，如origin地址、本地与远程分支的关系等：

   ``` gas
   [core]
   	repositoryformatversion = 0
   	filemode = false
   	bare = false
   	logallrefupdates = true
   	symlinks = false
   	ignorecase = true
   [remote "origin"]
   	url = git@github.com:ilmangoi/HY.git
   	fetch = +refs/heads/*:refs/remotes/origin/*
   [branch "master"]
   	remote = origin
   	merge = refs/heads/master
   [branch "dev"]
   	remote = origin
   	merge = refs/heads/dev
   ```

> 设置用户名与邮箱（用户标识，必要）

安装Git后首先要做的事情是设置用户名称和e-mail地址。这是非常重要的，因为每次Git提交都会使用该信息。它被永远的嵌入到了你的提交中：

``` gas
git config --global user.name "kuangshen"  #名称
git config --global user.email 24736743@qq.com   #邮箱
```

> 配置文件的优先级

local > global > system

每一个级别会覆盖上一级别的配置，所以local的配置会覆盖global中的配置。而global的配置会覆盖system中的配置。

## Git的基本理论

> git仓库的四个区域

- Workspace：工作区，就是你平时存放项目代码的地方
- Index / Stage：暂存区，用于临时存放你的改动，事实上它只是一个文件，保存即将提交到文件列表信息
- Repository：仓库区（或本地仓库），就是安全存放数据的位置，这里面有你提交到所有版本的数据。其中HEAD指向最新放入仓库的版本
- Remote：远程仓库，托管代码的服务器，可以简单的认为是你项目组中的一台电脑用于远程数据交换

![640](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/640.png)

> 仓库目录结构

- Directory：使用Git管理的一个目录，也就是一个仓库，包含我们的工作空间和Git的管理空间。
- WorkSpace：需要通过Git进行版本控制的目录和文件，这些目录和文件组成了工作空间。
- .git：存放Git管理信息的目录，初始化仓库的时候自动创建。
- Index/Stage：暂存区，或者叫待提交更新区，在提交进入repo之前，我们可以把所有的更新放在暂存区。
- Local Repo：本地仓库，一个存放在本地的版本库；HEAD会只是当前的开发分支（branch）。
- Stash：隐藏，是一个工作状态保存栈，用于保存/恢复WorkSpace中的临时状态。

![640 (1)](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/640%20(1).png)

> Git的工作流程

1. 在工作目录中添加、修改文件；

2. 将需要进行版本管理的文件放入暂存区域；

3. 将暂存区域的文件提交到git仓库。

   因此，git管理的文件有三种状态：已修改（modified）,已暂存（staged）,已提交(committed)

![640](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/640.jpg)

## Git项目搭建

> 本地创建搭建

``` gas
# 在当前目录新建一个Git代码库
git init
```

> 克隆远程仓库

``` gas
# 克隆一个项目和它的整个代码历史(版本信息)
git clone [url]  # https://gitee.com/kuangstudy/openclass.git
```

## Git忽略文件

> Git忽略规则匹配语法

1. 空格不匹配任意文件，可作为分隔符，可用反斜杠转义。
2. 如果一个模式不包含斜杠，则它匹配相对于当前 .gitignore 文件路径的内容。

3. 文件中的空行或以井号（#）开始的行将会被忽略。

4. 可以使用Linux通配符。例如：星号（*）代表任意多个字符，问号（？）代表一个字符，方括号（[abc]）代表可选字符范围，大括号（{string1,string2,...}）代表可选的字符串等。

5. ! 开头的模式表示否定，该文件可能会再次被包含，如果排除了该文件的父级目录，则使用 ! 也不会再次被包含。

6. / 开始的模式匹配项目跟目录。

7. / 结束的模式只匹配文件夹以及在该文件夹路径下的内容，但是不匹配该文件。

8. ** 匹配多级目录，可在开始，中间，结束。

> 常见匹配规则

- bin/: 忽略当前路径下的bin文件夹，该文件夹下的所有内容都会被忽略，不忽略 bin 文件 
- /bin: 忽略根目录下的bin文件 
- /*.c: 忽略 cat.c，不忽略 build/cat.c
- debug/*.obj: 忽略 debug/io.obj，不忽略 debug/common/io.obj 和 tools/debug/io.obj 
- **/foo: 忽略/foo, a/foo, a/b/foo等
- a/**/b: 忽略a/b, a/x/b, a/x/y/b等 
- !/bin/run.sh: 不忽略 bin 目录下的 run.sh 文件 
- *.log: 忽略所有 .log 文件 
- config.php: 忽略当前路径的 config.php 文件

> 常见问题：

如果想要添加一个文件到Git，但发现添加不了，原因是这个文件被`.gitignore`忽略了，如果确实想添加该文件，可以用`-f`强制添加到Git：

``` gas
$ git add App.class
The following paths are ignored by one of your .gitignore files:
App.class
Use -f if you really want to add them.

# 强制添加
$ git add -f App.class
```

或者也有可能是`.gitignore`写得有问题，需要找出来到底哪个规则写错了，可以用`git check-ignore`命令检查：

``` gas
$ git check-ignore -v App.class
.gitignore:3:*.class	App.class
# Git提示.gitignore的第3行规则忽略了该文件
```

- 修订规则：

  ``` gas
  # 排除所有.开头的隐藏文件:
  .*
  # 排除所有.class文件:
  *.class
  
  # 不排除.gitignore和App.class:
  !.gitignore
  !App.class
  ```

## Hello Wrold!

进行第一次版本控制：

一、使用git add把文件添加到仓库：

``` gas
git add helloWorld.md
```

二、使用git commit把文件提交到仓库：

``` gas
 git commit -m "wrote a readme file"
```

为什么Git添加文件需要`add`，`commit`一共两步呢？因为`commit`可以一次提交很多文件，所以可以多次`add`不同的文件，比如：

``` gas
git add file1.txt
git add file2.txt file3.txt
git commit -m "add 3 files."
```

## 版本控制

修改helloworld文件后，可以通过`git status`命令来查看仓库状态：

``` gas
$ git status

On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)     
        modified:   helloWrold.md

no changes added to commit (use "git add" and/or "git commit -a")
```

`git status`命令可以让我们时刻掌握仓库当前的状态，上面的命令输出告诉我们，`readme.txt`被修改过了，但还没有准备提交的修改。

#### git diff命令

虽然Git告诉我们`readme.txt`被修改了，但是不能看到具体修改了什么内容，如果想要查看具体哪些内容被修改了，需要使用`git diff`命令：

``` gas
$ git diff .\helloWrold.md

diff --git a/helloWrold.md b/helloWrold.md
index bc7774a..a3395c8 100644
--- a/helloWrold.md
+++ b/helloWrold.md
@@ -1 +1,5 @@
-hello world!
\ No newline at end of file
+hello world!
+changed!
+a
+a
+a
\ No newline at end of file
```

git diff 有两个主要的应用场景：

- 尚未缓存的改动：**git diff**
- 查看已缓存的改动： **git diff --staged**
- 查看已缓存的与未缓存的所有改动：**git diff HEAD**

``` gas
git diff [file]
# 比较文件在暂存区和工作区的差异

git diff --staged [file]
# 比较暂存区和上一次提交(commit)的差异

git diff HEAD [file]
# 比较工作区和上一次提交(commit)的差异

git diff [first-commit] [second-commit]
# 比较两次提交之间的差异
```

#### 版本回退

不断对文件进行修改，然后不断提交修改到版本库里，就好比玩RPG游戏时，每通过一关就会自动把游戏状态存盘，如果某一关没过去，你还可以选择读取前一关的状态。Git也是一样，每当你觉得文件修改到一定程度的时候，就可以“保存一个快照”，这个快照在Git中被称为`commit`。一旦你把文件改乱了，或者误删了文件，还可以从最近的一个`commit`恢复，然后继续工作，而不是把几个月的工作成果全部丢失。

我们可以通过`git log`命令来查看历史版本，如果嫌输出信息太多，可以加上`--pretty-oneline`参数来简化输出：

``` gas
$ git log --pretty=oneline

ad38df9d9ff3d926280ce431b0df7a39f26566a9 (HEAD -> master) worte a md fild    
a9b8e9e609922660b81e868c351e0bbb489fdd23 first commit
```

每个版本中一大串类似`1094adb...`的是`commit id`（版本号），和SVN不一样，Git的`commit id`不是1，2，3……递增的数字，而是一个SHA1计算出来的一个非常大的数字，用十六进制表示。

为什么`commit id`需要用这么一大串数字表示呢？因为Git是分布式的版本控制系统，它可以让多人在同一个版本库里工作，如果大家都用1，2，3……作为版本号，那肯定就冲突了。

知道了`commit id`，就可以通过它回退到之前的版本。在Git中，用`HEAD`表示当前版本，上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个`^`比较容易数不过来，所以写成`HEAD~100`。

``` gas
$ git reset --hard HEAD^
HEAD is now at ad38df9 worte a md fild
```

现在项目就已经回退到了上一个版本中，可以使用`git log`再看看现在版本库的状态：

``` gas
$ git log
commit ad38df9d9ff3d926280ce431b0df7a39f26566a9 (HEAD -> master)
Author: ilmango <mangguoli42@gmail.com>
Date:   Tue May 24 14:00:03 2022 +0800

    worte a md fild

commit a9b8e9e609922660b81e868c351e0bbb489fdd23
Author: ilmango <mangguoli42@gmail.com>
Date:   Tue May 24 13:50:21 2022 +0800

    first commit
```

最新的那个版本已经看不到了！好比从21世纪坐时光穿梭机来到了19世纪，如果想要回去，就必须找到版本的`commit id`：

``` gas
$ git reset --hard a8387aa
HEAD is now at a8387aa change md file
```

Git的版本回退速度非常快，因为Git在内部有个指向当前版本的`HEAD`指针，当你回退版本的时候，Git仅仅是改变了HEAD的指向，然后顺便把工作区的文件更新了，所以你让`HEAD`指向哪个版本号，你就把当前版本定位在哪。

```ascii
┌────┐
│HEAD│
└────┘
   │
   └──> ○ append GPL
        │
        ○ add distributed
        │
        ○ wrote a readme file
┌────┐
│HEAD│
└────┘
   │
   │    ○ append GPL
   │    │
   └──> ○ add distributed
        │
        ○ wrote a readme file
```

现在，你回退到了某个版本，关掉了电脑，第二天早上就后悔了，想恢复到新版本怎么办？找不到新版本的`commit id`怎么办？

在Git中，总是有后悔药可以吃的。当你用`$ git reset --hard HEAD^`回退到`add distributed`版本时，再想恢复回来，就必须找到它的commit id。Git提供了一个命令`git reflog`用来记录你的每一次命令：

``` gas
$ git reflog

a8387aa (HEAD -> master) HEAD@{0}: reset: moving to a8387aa
ad38df9 HEAD@{1}: reset: moving to HEAD^
a8387aa (HEAD -> master) HEAD@{2}: reset: moving to a8387aa
ad38df9 HEAD@{3}: reset: moving to HEAD^
a8387aa (HEAD -> master) HEAD@{4}: reset: moving to a8387aa
ad38df9 HEAD@{5}: reset: moving to HEAD^
a8387aa (HEAD -> master) HEAD@{6}: commit: change md file
ad38df9 HEAD@{7}: commit: worte a md fild
a9b8e9e HEAD@{8}: commit (initial): first commit
```

##### git reset的三种模式：

- reset --hard:重置stage区和工作目录

  reset --hard 会在重置HEAD和branch的同时，重置stage区和工作目录里的内容。stage区和工作目录里的内容会被完全重置为指定提交中的内容。

  > 换句话说就是没有commit 的修改会被全部擦掉。也可以类比为先git restore暂存区中的内容，然后git reset重置HEAD，最后再把工作区中的内容更改为HEAD中的内容。

- reset --soft:保留工作目录,把重置HEAD所带来的新的差异放进暂存区

  reset --soft 会在重置HEAD和branch时，保留工作目录和暂存区中的内容，并把[重置HEAD所带来的新的差异]放进暂存区。
  
  ![](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/131f7bb20e3b4382bf5868d7a78e43ab.png)
  
  把HEAD从4移动到3，而且在reset过程中工作目录和暂存区的内容没有被清理掉，所以4中的改动在reset后就也成了工作目录新增的 [工作目录和HEAD的差异]。这就是[重置HEAD所带来的差异]。
  
  当我们想合并当前节点和reset目标节点之间不具有太大意义的commit记录 (可能是阶段性地频繁提交)时，可以考虑使用Soft Reset来让commit 演进线图较为清晰点。
  
  > 这个操作可以类比为先使用git reset重置HEAD，这时工作区和最新提交一定是有差异的，这时再使用git add *把所有差异提交到暂存区。
  
- reset 不加参数（mixed）：保留工作目录，并清空暂存区

  保留工作目录并且清空暂存区。也就是说工作目录的修改、暂存区的内容以及由reset所导致的新的文件的差异，都会被放进工作目录。简而言之 ，就是把所有差异都混合（mixed）放在工作目录中。

  > 这个操作可以类比为先使用git retore把暂存区的修改撤销，然后再使用git reset重置HEAD指向。

#### 工作区和暂存区

> 工作区

工作区就是电脑中能看到的项目文件。

> 版本库

工作区有一个隐藏目录`.git`，这个不算工作区，而是Git的版本库。

Git的版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支`master`，以及指向`master`的一个指针叫`HEAD`。

![0](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/0.jpg)

Git版本库里添加的时候，是分两步执行的：

- 第一步是用`git add`把文件添加进去，实际上就是把文件修改添加到暂存区；

- 第二步是用`git commit`提交更改，实际上就是把暂存区的所有内容提交到当前分支。

因为我们创建Git版本库时，Git自动为我们创建了唯一一个`master`分支，所以，现在，`git commit`就是往`master`分支上提交更改。

可以简单理解为，需要提交的文件修改通通放到暂存区，然后，一次性提交暂存区的所有修改。一旦提交后，如果你又没有对工作区做任何修改，那么工作区就是“干净”的：`nothing to commit, working tree clean`

#### 管理修改

为什么Git比其他版本控制系统设计得优秀，因为Git跟踪并管理的是修改，而非文件。什么是修改？比如你新增了一行，这就是一个修改，删除了一行，也是一个修改，更改了某些字符，也是一个修改，删了一些又加了一些，也是一个修改，甚至创建一个新文件，也算一个修改。

为什么说Git管理的是修改，而不是文件呢？我们对helloWrold.md做一个修改，比如加一行内容，然后添加它：

``` gas
$ git add readme.txt
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   helloWrold.md
```

然后再修改readme.txt，最后提交它：

``` gas
$ git commit -m "change"
[master 1850a27] change
 1 file changed, 1 insertion(+)
```

提交完成之后再查看仓库状态：

``` gas
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   helloWrold.md
```

可以看到第二次的修改没有被提交，回顾一个操作过程：

第一次修改 -> `git add` -> 第二次修改 -> `git commit`

这就体现了Git管理的是修改，当用`git add`命令后，在工作区的第一次修改被放入暂存区，准备提交，但是，在工作区的第二次修改并没有放入暂存区，所以，`git commit`只负责把暂存区的修改提交了，也就是第一次的修改被提交了，第二次的修改不会被提交。

#### 撤销修改

撤销修改可以使用`git checkout`指令：

``` gas
git checkout helloWorld.md
```

命令`git checkout helloWorld.md`意思就是，把这个文件在工作区的修改全部撤销，这里有两种情况：

- `readme.txt`修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；

- 一种是`readme.txt`已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

总之，就是让这个文件回到最近一次`git commit`或`git add`时的状态。

------

如果在helloWorld中写了一些错误的内容，并且已经将其add到暂存区了，所幸在commit之前发现了这个问题。那么这时，可以通过`git restore`来解决这个问题：

``` gas
git restore --staged .\helloWrold.md
```

这个命令可以把暂存区的修改撤销掉(unstage)，重新放回到工作区，再用`git status`查看一下，现在暂存区是干净的，工作区有修改：

``` gas
$ git status

On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   helloWrold.md
```

然后再结合`git checkout`来撤销工作区的修改：

``` gas
$ git checkout .\helloWrold.md
Updated 1 path from the index

$ git status
On branch master
nothing to commit, working tree clean
```

#### 删除文件

在Git中删除也是一个修改操作，比如我现在删除了一个文件，Git检测到删除了文件，因此，工作区和版本库就不一致了，`git status`命令会输出哪些文件被删除了：

``` gas
$ del helloWorld.md
$ git status

On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        deleted:    helloWrold.md
```

现在这种情况两个选择：

1. 确实要从版本库中删除该文件，那就用命令`git rm`删掉，并且`git commit`：

   ``` gas
   $ git rm helloWrold.md
   rm 'helloWrold.md'
   
   $ git commit -m "remove md file"
   [master d82db0c] remove md file
    1 file changed, 3 deletions(-)
    delete mode 100644 helloWrold.md
   ```

2. 删错了，因为版本库里还有呢，所以可以很轻松地把误删的文件恢复到最新版本：

   ``` gas
   $ git checkout helloWrold.md
   Updated 1 path from the index
   ```

`git checkout`其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。从来没有被添加到版本库就被删除的文件，是无法恢复的！

当我们删除一个文件，并使用`git rm`命令对其进行删除后，将无法使用`git checkout`命令将其删除操作撤销，只能通过`git restore`撤销缓存区的修改，然后再将仓库回退至上一个版本。

``` gas
$ git rm helloWrold.md
rm 'helloWrold.md'

$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    helloWrold.md

$ git restore --staged helloWrold.md
$ git checkout helloWrold.md
Updated 1 path from the index
```

## 远程仓库

GitHub提供Git仓库托管服务，只要注册一个GitHub账号，就可以免费获得Git远程仓库。由于本地Git仓库和GitHub仓库之间的传输是通过SSH加密的，所以需要一点设置：

1. 创建SSH Key。

   ``` gas
   ssh-keygen -t rsa -C "youremail@example.com"
   ```

   指令运行完毕后，就可以在用户主目录里找到`.ssh`目录，里面有`id_rsa`和`id_rsa.pub`两个文件，这两个就是SSH Key的秘钥对，`id_rsa`是私钥，不能泄露出去，`id_rsa.pub`是公钥，可以放心地告诉任何人。

2. 登陆GitHub，设置publick key。

   ![Snipaste_2022-05-24_18-06-00](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/Snipaste_2022-05-24_18-06-00.png)

为什么GitHub需要SSH Key呢？

因为GitHub需要识别出你推送的提交确实是你推送的，而不是别人冒充的，而Git支持SSH协议，所以，GitHub只要知道了你的公钥，就可以确认只有你自己才能推送。

#### 添加远程库

1. 首先要在GitHub上创建一个仓库：

   ![Snipaste_2022-05-24_18-16-42](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/Snipaste_2022-05-24_18-16-42.png)

2. 创建的仓库是一个空仓库，GitHub告诉我们，可以从这个仓库克隆出新的仓库，也可以把一个已有的本地仓库与之关联，然后，把本地仓库的内容推送到GitHub仓库。我们可以根据GitHub的提示，在本地HY仓库中运行如下命令：

   ``` gas
   git remote add origin git@github.com:ilmangoi/HY.git
   ```

   每个本地仓库都可以单独绑定远和仓库，如果添加的时候地址写错了，或者就是想删除远程库，可以用`git remote rm <name>`命令。使用前，建议先用`git remote -v`查看远程库信息：

   ``` gas
   $ git remote -v
   origin  git@github.com:ilmangoi/HY.git (fetch)
   origin  git@github.com:ilmangoi/HY.git (push)
   $ git remote rm origin
   ```
3. 下一步就可以把本地库的所有内容推送到远程库上：

   ``` gas
   $ git push -u origin master
   The authenticity of host 'github.com (20.205.243.166)' can't be established.
   ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
   This key is not known by any other names
   Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
   Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
   Enumerating objects: 24, done.
   Counting objects: 100% (24/24), done.
   Delta compression using up to 8 threads
   Compressing objects: 100% (20/20), done.
   Writing objects: 100% (24/24), 8.48 KiB | 1.06 MiB/s, done.
   Total 24 (delta 5), reused 0 (delta 0), pack-reused 0
   remote: Resolving deltas: 100% (5/5), done.
   To github.com:ilmangoi/HY.git
    * [new branch]      master -> master
   Branch 'master' set up to track remote branch 'master' from 'origin'.
   ```

   由于远程库是空的，我们第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。

   推送成功后，可以立刻在GitHub页面中看到远程库的内容已经和本地一模一样，从现在起，只要本地作了提交，就可以通过命令把本地`master`分支的最新修改推送至GitHub：

   ``` gas
   git push
   ```


> SSH警告

第一次使用Git的`clone`或者`push`命令连接GitHub时，会得到一个警告：

``` gas
The authenticity of host 'github.com (20.205.243.166)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? 
```

这是因为Git使用SSH连接，而SSH连接在第一次验证GitHub服务器的Key时，需要你确认GitHub的Key的指纹信息是否真的来自GitHub的服务器，输入`yes`回车即可。然后Git会输出另一个警告，告诉你已经把GitHub的Key添加到本机的一个信任列表里了：

``` gas
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
```

这个警告只会出现一次，后面的操作就不会有任何警告了。如果担心有人冒充GitHub服务器，输入`yes`前可以对照GitHub的RSA Key的指纹信息是否与SSH连接给出的一致。

#### 从远程库克隆

上节中是先有本地库，后有远程库，然后如何关联远程库。但是如果从零开发，那么最好的方式是先创建远程库，然后，从远程库克隆。

1. 首先登陆GitHub创建一个新的仓库。

2. 远程库准备好了以后，就可以用`git clone`克隆一个本地库。

   ``` gas
   $ git clone git@github.com:ilmangoi/HY.git
   
   Cloning into 'HY'...
   remote: Counting objects: 3, done.
   remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 3
   Receiving objects: 100% (3/3), done.
   ```

如果有多个人协作开发，那么每个人各自从远程克隆一份就可以了。

并且GitHub给出的地址不止一个，还可以用`https://github.com/ilmangoi/HY.git`这样的地址。实际上，Git支持多种协议，默认的`git://`使用ssh，但也可以使用`https`等其他协议。使用`https`除了速度慢以外，还有个最大的麻烦是每次推送都必须输入口令，但是在某些只开放http端口的公司内部就无法使用`ssh`协议而只能用`https`。

## 分支管理

> 分支在实际中有什么用呢？

假设准备开发一个新功能，但是需要两周才能完成，第一周写了50%的代码，如果立刻提交，由于代码还没写完，会导致其他人无法工作。如果等代码全部写完再一次性提交，又存在丢失每天进度的巨大风险。

现在有了分支就可以创建一个属于自己的分支，别人看不到，还继续在原来的分支上正常工作，而自己在自己的分支上干活，想提交就提交，直到开发完毕后，再一次性合并到原来的分支上，这样既安全又不影响别人工作。

#### 创建与合并分支

Git会把所有提交都串成一条时间线，这条时间线就是一个分支。仓库初始化后默认只有一个分支，即`master`分支。

`master`指针是指向最新的`commit`，`HEAD`指针是指向当前分支，这样就能确定当前分支，以及当前分支的提交点：

![0](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/0.png)

当我们创建新的分支，例如`dev`时，Git新建了一个指针叫`dev`，指向`master`相同的提交，再把`HEAD`指向`dev`，就表示当前分支在`dev`上：

![l](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/l.png)

从现在开始，对工作区的修改和提交就是针对`dev`分支了，比如新提交一次后，`dev`指针往前移动一步，而`master`指针不变：

![l (1)](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/l%20(1).png)

如果在`dev`上的工作完成了，就可以把`dev`合并到`master`上。合并的方法就是直接把`master`指向`dev`的当前提交：

![0 (1)](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/0%20(1).png)

创建`dev`分支，然后切换到`dev`分支：

``` gas
$ git branch dev

$ git switch dev
Switched to branch 'dev'

$ git branch
* dev
  master
```

进行一次提交后，`dev`分支的工作完成，切换回`master`分支：

``` gas
$ git add *
$ git commit -m "branch test"
[dev 579cb2d] branch test
 1 file changed, 1 insertion(+)
 
$ git switch master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
```

现在就把`dev`分支的工作成果合并到`master`分支上：

``` gas
git merge dev
Updating 6420dcf..579cb2d
Fast-forward
 helloWrold.md | 1 +
 1 file changed, 1 insertion(+)
```

`git merge`命令用于合并指定分支到当前分支。合并后，再查看在dev分支中修改的内容，就可以看到和`dev`分支的最新提交是完全一样的。

注意到上面的`Fast-forward`信息，Git告诉我们，这次合并是“快进模式”，也就是直接把`master`指向`dev`的当前提交，所以合并速度非常快。当然，也不是每次合并都能`Fast-forward`，因为dev分支提交内容的同时，master分支也有可能提交内容，这就有可能导致合并冲突。合并完成后，就可以放心地删除`dev`分支了：

``` gas
$ git branch -d dev
Deleted branch dev (was 579cb2d).

$ git branch
* master
```

因为创建、合并和删除分支非常快，所以Git鼓励使用分支完成某个任务，合并后再删掉分支，这和直接在`master`分支上工作效果是一样的，但过程更安全。

> git branch常见选项

``` gas
git branch
# 列举仓库中的所有分支。此命令与`git branch --list`是同义词。

git branch <branch>
# 创建一个名为 <branch>的分支。但此命令并不会自动检出新创建的分支。

git branch -d <branch>
# 删除指定分支。这是一个安全的操作，因为当分支中含有未合并的变更时，Git会阻止这一次删除操作。

git branch -D <branch>
# 强制删除指定分支，即便其中含有未合并的变更。该命令常见于当开发者希望永久删除某一开发过程中的所有commit。

git branch -m <branch>
# 把当前分支重命名为<branch>。

git branch -a
# 列举所有分支，包括已绑定的远程分支。

git branch --set-upstream-to=origin/<branch> <branch>
# 把远程的<branch>分支与本地的<branch>分支创建连接，创建连接后就可以简化git push
# 与git pull指令，原本要推送master分支的话，要执行git push origin master指令
# 建立连接后再想摄像头master分支只需要切换到mater分支后执行git push即可

# git push -u origin master指令可以在推送master分支的同时为远程master分支与本地
# master分支建立连接，其效果与git branch --set-upstream-to相同
```

#### 解决冲突

如果`master`分支和`feature1`分支各自都分别有新的提交：

![0](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/0-16534055659711.png)

这种情况下，Git无法执行“快速合并”，只能试图把各自的修改合并起来，但这种合并就可能会有冲突，可以通过`git status`查看冲突的文件：

``` gas
$ git merge feature1
Auto-merging helloWrold.md
CONFLICT (content): Merge conflict in helloWrold.md
Automatic merge failed; fix conflicts and then commit the result.

$ git status
On branch master
Your branch is ahead of 'origin/master' by 2 commits.
  (use "git push" to publish your local commits)

You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   helloWrold.md
```

Git告诉我们helloWorld.md文件在合并时发生了冲突，我们看一下这个文件的内容：

``` gas
hello world!
changed!
changed!
changed!
<<<<<<< HEAD
master
=======
feature1
>>>>>>> feature1
```

手动解决冲突，可以删除其中一个分支的内容，也可以合并两个分支冲突的内容，或者进行其它修改，修改完成后再对其进行提交：

``` gas
$ git add .\helloWrold.md
$ git commit -m "merge branch"
[master cbb3bfd] merge branch
```

现在，`master`分支和`feature1`分支变成了下图所示：

![0 (1)](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/0%20(1)-16534065410972.png)

#### 分支管理策略

通常，合并分支时，如果可能，Git会用`Fast forward`模式，但这种模式下，删除分支后，会丢掉分支信息。

如果要强制禁用`Fast forward`模式，Git就会在merge时生成一个新的`commit`，这样，从分支历史上就可以看出分支信息。

首先在`dev`分支中提交一个新的`commit`，切换回`master`后合并`dev`分支，请注意`--no-ff`参数，表示禁用`Fast forward`：

``` gas
# 创建并切换dev分支：
$ git switch -c dev
Switched to a new branch 'dev'
# 修改readme.txt文件，并提交一个新的commit：
$ git add readme.txt 
$ git commit -m "add merge"
[dev f52c633] add merge
 1 file changed, 1 insertion(+)
# 我们切换回master
$ git switch master
Switched to branch 'master'
# 合并dev分支
$ git merge --no-ff -m "merge with no-ff" dev
Merge made by the 'recursive' strategy.
 readme.txt | 1 +
 1 file changed, 1 insertion(+)
```

因为本次合并要创建一个新的commit，所以加上`-m`参数，把commit描述写进去。不使用`Fast forward`模式，merge后就是这样：

![0 (2)](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/0%20(2).png)

> 分支策略

在实际开发中，我们应该按照几个基本原则进行分支管理：

首先，`master`分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活。干活都在`dev`分支上，也就是说，`dev`分支是不稳定的，到某个时候，比如1.0版本发布时，再把`dev`分支合并到`master`上，在`master`分支发布1.0版本。每个人都在`dev`分支上干活，每个人都有自己的分支，时不时地往`dev`分支上合并就可以了。所以，团队合作的分支看起来就像这样：

![0 (3)](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/0%20(3).png)

#### Bug分支

在Git中分支非常的强大，每个bug都可以通过一个新的临时分支来修复，修复后合并分支，然后将临时分支删除。

假如现在接到一个代号101的bug的任务，这时候可以创建一个分支`issue-101`来修复它，但是正在`dev`上进行的工作还没有提交，并不是不想提交，而是工作只进行到一半，还没法提交，预计完成还需1天时间。但是，必须在两个小时内修复该bug。

这个时候就可以用到Git提供的`stash`功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作：

``` gas
$ git status
On branch dev
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   helloWrold.md

$ git stash
Saved working directory and index state WIP on dev: 5d53bf1 merge stash

$ git status
On branch dev
nothing to commit, working tree clean
```

保存工作现场后，就可以切换到需要修复bug的分支，master分支是发布版本，因此修复bug的任务一般在master分支中。

在当前的master分支中创建一个issue-101分支修复Bug，然后切换回`master`分支，完成合并，最后删除`issue-101`分支：

``` gas
$ git switch master
Switched to branch 'master'

$ git branch issue-101
$ git switch issue-101
Switched to branch 'issue-101'

$ git add *
$ git commit -m "fix issue-101"
[issue-101 bd2523a] fix issue-101
 1 file changed, 1 insertion(+), 1 deletion(-)
 
$ git switch master
Switched to branch 'master'

$ git merge --no-ff -m "merge issue-101" issue-101
Merge made by the 'ort' strategy.
 helloWrold.md | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

Bug修复完毕后，就可以回到`dev`分支上干活了：

``` gas
git switch dev
Switched to branch 'dev'
PS G:\web\HY> git stash pop
On branch dev
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   helloWrold.md
```

可以看到刚才保存的工作状态就恢复回来了。但是dev分支是早期从master分支分出来的，所以，这个bug其实在当前dev分支上也存在。

要在dev分支上修复同样的bug，可以重复操作一次再提交就可以了。但是git给我们提供了更简便的方法。

同样的bug，要在dev上修复，我们只需要把`fix issue-101`这个提交所做的修改“复制”到dev分支。注意：我们只想复制`fix issue-101`这个提交所做的修改，并不是把整个master分支merge过来。 为了方便操作，Git专门提供了一个`cherry-pick`命令，让我们能复制一个特定的提交到当前分支：

``` gas
$ git branch
* dev
  master
  
$ git cherry-pick 4c805e2
[master 1d4b803] fix bug 101
 1 file changed, 1 insertion(+), 1 deletion(-)
```

Git自动给dev分支做了一次提交，注意这次提交的commit是`1d4b803`，它并不同于master的`4c805e2`，因为这两个commit只是改动相同，但确实是两个不同的commit。用`git cherry-pick`，我们就不需要在dev分支上手动再把修bug的过程重复一遍。

> git stash指令

``` gas
$ git stash [save "XXX"]                          
# 暂存工作区，加上save选项表示给暂存加注释

$ git stash list                      
# 查看暂存列表

$ git stash pop [stash_id]                       
# 将堆栈中最新的内容pop出来应用到当前分支上,例如git stash pop 0

$ git stash apply [stash_id]
# 与pop相似，但他不会在堆栈中删除这条缓存，适合在多个分支中进行缓存应用

$ git stash drop [stash_id]
# 删除单个暂存

$ git stash clear
# 清空所有暂存

$ git stash show [stash_id] [-p]
# 查看指定暂存与当前分支的差异，加上-p可以看详细差异
```

#### Feature分支

添加一个新功能时，为了不让一些实验性质的代码把主分支搞乱，所以每添加一个新功能，最好新建一个feature分支，在feature分支上面开发，开发完成后合并分支，最后删除该feature分支。

``` gas
# 创建feature分支
$ git branch feature
$ git switch feature
Switched to branch 'feature'

# 开发完成后提交
$ git add *
$ git commit -m "add feature vulcan"
[feature 2da8299] add feature vulcan
 1 file changed, 1 insertion(+)
```

开发完成后就可以切回`dev`，准备合并。正常情况下就是合并feature分支，然后删除。但如果此时领导要求这个新功能要废弃掉，虽然白干了，但是这个包含机密资料的分支还是必须就地销毁：

``` gas
$ git switch dev
Switched to branch 'dev'
Your branch is up to date with 'origin/dev'.

$ git branch -d feature
error: The branch 'feature' is not fully merged.
If you are sure you want to delete it, run 'git branch -D feature'.
```

销毁失败。Git提醒`feature`分支还没有被合并，如果删除，将丢失掉修改，如果要强行删除，需要使用大写的`-D`参数：

``` gas
git branch -D feature
Deleted branch feature (was 2da8299).
```

#### 多人协作

> 多人协作的工作模式：

1. 首先，可以试图用`git push origin <branch-name>`推送自己的修改；
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；
3. 如果合并有冲突，则解决冲突，并在本地提交；
4. 没有冲突或者解决掉冲突后，再用`git push origin <branch-name>`推送就能成功！

如果`git pull`提示`no tracking information`，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream-to <branch-name> origin/<branch-name>`。

> 克隆仓库

从远程仓库克隆时，实际上Git自动把本地的`master`分支和远程的`master`分支对应起来了，并且，远程仓库的默认名称是`origin`，可以通过git remote进行查看：

``` gas
$ git remote -v
origin  git@github.com:ilmangoi/HY.git (fetch)
origin  git@github.com:ilmangoi/HY.git (push)
```

上面显示了可以抓取和推送的`origin`的地址。如果没有推送权限，就看不到push的地址。

> 推送分支

推送分支，就是把该分支上的所有本地提交推送到远程库。推送时要指定本地分支，这样Git就会把该分支推送到远程库对应的远程分支上：

``` gas
$ git push -u origin master
# master分支是主分支，因此要时刻与远程同步；

$ git push -u origin dev
# dev分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；
```

> 抓取分支

多人协作时，大家都会往`master`和`dev`分支上推送各自的修改。

现在模拟一个小伙伴与我协同合作。可以在另一台电脑（注意要把SSH Key添加到GitHub）或者同一台电脑的另一个目录下克隆：

``` gas
$ git clone git@github.com:ilmangoi/HY.git
Cloning into 'HY'...
remote: Enumerating objects: 57, done.
remote: Counting objects: 100% (57/57), done.
remote: Compressing objects: 100% (27/27), done.
remote: Total 57 (delta 23), reused 56 (delta 22), pack-reused 0
Receiving objects: 100% (57/57), 10.93 KiB | 2.19 MiB/s, done.
Resolving deltas: 100% (23/23), done.

$ git remote -v
origin  git@github.com:ilmangoi/HY.git (fetch)
origin  git@github.com:ilmangoi/HY.git (push) 

$ git branch
* master
```

clone完成后，默认情况下只能看到本地的`master`分支。如果要在`dev`分支上开发，就必须创建远程`origin`的`dev`分支到本地：

``` gas
$ git branch dev origin/dev
Branch 'dev' set up to track remote branch 'dev' from 'origin'.

$ git switch dev
Switched to branch 'dev'
Your branch is up to date with 'origin/dev'.
```

现在，小伙伴就可以在`dev`上继续修改，然后，时不时地把`dev`分支`push`到远程：

``` gas
$ git add *
$ git commit -m "firend change!"
[dev bd8a39f] firend change!
 1 file changed, 1 insertion(+)
 
$ git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 299 bytes | 299.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
To github.com:ilmangoi/HY.git
   cd7f3cb..bd8a39f  dev -> dev
```

小伙伴已经向`origin/dev`分支推送了他的提交，而碰巧自己也对同样的文件作了修改并试图推送：

``` gas
$ git add *
$ git commit -m "master change"
[dev 227f6dc] master change
 1 file changed, 1 insertion(+)
 
$ git push
To github.com:ilmangoi/HY.git
 ! [rejected]        dev -> dev (fetch first)
error: failed to push some refs to 'github.com:ilmangoi/HY.git'
hint: Updates were rejected because the remote contains work that you do
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

推送失败，因为小伙伴的最新提交和自己推送的提交有冲突，解决办法也很简单，Git已经提示我们，先用`git pull`把最新的提交从`origin/dev`抓下来，然后在本地合并，解决冲突后再推送：

``` gas
$ git pull
Auto-merging helloWrold.md
CONFLICT (content): Merge conflict in helloWrold.md
Automatic merge failed; fix conflicts and then commit the result.

##### 手动解决冲突 #####

$ git add *
$ git commit -m "fix conflict"
[dev 4d919d9] fix conflict

$ git push
Enumerating objects: 10, done.
Counting objects: 100% (10/10), done.
Delta compression using up to 8 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (6/6), 561 bytes | 561.00 KiB/s, done.
Total 6 (delta 4), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (4/4), completed with 2 local objects.
To github.com:ilmangoi/HY.git
   bd8a39f..4d919d9  dev -> dev
```

#### Rebase

多人在同一个分支上协作时，很容易出现冲突。即使没有冲突，后push的童鞋不得不先pull，在本地合并，然后才能push成功。每次合并再push后，分支就会变得很乱，为什么Git的提交历史不能是一条干净的直线？

这里就可以用到`git rebase`指令了，rebase操作可以把本地未push的分叉提交历史整理成直线，rebase的目的是使得我们在查看历史提交的变化时更容易，因为分叉的提交需要三方对比。

> Rebase应用实例

仓库在和远程分支同步后，又对`hello.py`这个文件做了两次提交。用`git log`命令看看：

``` gas
$ git log --graph --pretty=oneline --abbrev-commit
* 582d922 (HEAD -> master) add author
* 8875536 add comment
* d1be385 (origin/master) init hello
```

可以注意到Git用`(HEAD -> master)`和`(origin/master)`标识出当前分支的HEAD和远程origin的位置分别是`582d922 add author`和`d1be385 init hello`，本地分支比远程分支快两个提交。现在我们尝试推送本地分支：

``` gas
$ git push
To github.com:ilmangoi/HY.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'git@github.com:ilmangoi/HY.git'
... ...
```

提交失败了，这说明有人先于我们推送了远程分支。这时候要pull一下：

``` gas
$ git pull
  ... ...
```

远程分支与本地分支没有冲突，因此Git自己帮我们对分支进行了合并，用`git log`看看：

``` gas
$ git log --graph --pretty=oneline --abbrev-commit
*   e0ea545 (HEAD -> master) Merge branch 'master' of github.com:ilmangoi/HY.git
|\  
| * f005ed4 (origin/master) set exit=1
* | 582d922 add author
* | 8875536 add comment
|/  
* d1be385 init hello
```

可以看到提交历史分叉了，如果现在把本地分支push到远程当然也没有问题，但是很显然，这样并不美观。想要解决这个问题就可以使用`git rebase`指令： 

``` gas
$ git rebase
  ... ...
  
$ git log --graph --pretty=oneline --abbrev-commit
* 7e61ed4 (HEAD -> master) add author
* 3611cfe add comment
* f005ed4 (origin/master) set exit=1
* d1be385 init hello
```

原本分叉的提交现在变成一条直线了！其实原理非常简单。注意观察可以发现Git把我们本地的提交“挪动”了位置，放到了`f005ed4 (origin/master) set exit=1`之后，这样，整个提交历史就成了一条直线。rebase操作前后，最终的提交内容是一致的，但是我们本地的commit修改内容已经变化了，它们的修改不再基于`d1be385 init hello`，而是基于`f005ed4 (origin/master) set exit=1`，但最后的提交`7e61ed4`内容是一致的。

最后，通过push操作把本地分支推送到远程后用`git log`看看效果，远程分支的提交历史也是一条直线了：

``` gas
$ git push

$ git log --graph --pretty=oneline --abbrev-commit
* 7e61ed4 (HEAD -> master, origin/master) add author
* 3611cfe add comment
* f005ed4 set exit=1
* d1be385 init hello
```

## 标签管理

Git的标签虽然是版本库的快照，但其实它就是指向某个commit的指针（跟分支很像，但是分支可以移动，标签不能移动），所以创建和删除标签都是瞬间完成的。Git已经有了commit，为什么还要引入tag？

这是因为commit号是一串乱七八糟的数字与字母的组合，没有标识性，而tag是一个让人容易记住的有意义的名字，它跟某个commit绑在一起，这样就使方便我们去查找某一个版本。

#### 创建标签

``` gas
$ git tag v1.0         
# 创建一个标签，默认标签是打在最新提交的commit上的

$ git tag
v1.0
# 查看所有标签

$ git tag v0.9 [commit-id]
# 给指定commit打标签

$ git show v0.9
commit 5d53bf1f4052f4c161f0a952758476a12e19676b (tag: v0.9)
Author: ilmango <mangguoli42@gmail.com>
Date:   Wed May 25 11:29:40 2022 +0800

    merge stash
# 查看标签详情

$ git tag -a v0.1 -m "version 0.1 released" 1850a  
# 创建带有说明的标签

$ git show v0.1
tag v0.1
Tagger: ilmango <mangguoli42@gmail.com>
Date:   Wed May 25 20:31:06 2022 +0800

version 0.1 released

commit 1850a27dcd8e9ce89a2c3f1553a74223a8353dd2 (tag: v0.1)
Author: ilmango <mangguoli42@gmail.com>
Date:   Tue May 24 15:53:30 2022 +0800

    change
# 查看带有说明的标签详情
```

**注意：标签总是和某个commit挂钩。如果这个commit既出现在master分支，又出现在dev分支，那么在这两个分支上都可以看到这个标签。**

#### 操作标签

``` gas
$ git tag -d v0.1
Deleted tag 'v0.1' (was 288f40d)
# 删除标签
```

因为创建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除。如果要推送某个标签到远程，使用如下命令：

``` gas
$ git push origin v1.0
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:ilmangoi/HY.git
 * [new tag]         v1.0 -> v1.0
# 推送某个标签到远程

$ git push origin --tags
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:ilmangoi/HY.git
 * [new tag]         v0.9 -> v0.9
# 一次性推送全部尚未推送到远程的本地标签
```

如果标签已经推送到远程，要删除远程标签要先从本地删除，然后再从远程删除：

``` gas
$ git tag -d v0.9
Deleted tag 'v0.9' (was 5d53bf1)

$ git push origin -d tag v1.0
To github.com:ilmangoi/HY.git
 - [deleted]         v0.9
```

要看看是否真的从远程库删除了标签，可以登陆GitHub查看：

![Snipaste_2022-05-25_20-50-07](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/Snipaste_2022-05-25_20-50-07.png)

## Pull Request

开源项目开源容易，但是让大家都参与进来比较困难，因为要参与就要提交代码，如果给每个想提交代码的群众都开一个账号那是不现实的，因此群众也仅限于报个bug，即使能改掉bug，也只能把diff文件用邮件发过去，很不方便。

但是通过GitHub的Pull Request功能，可以让群众真正可以自由参与各种开源项目。参与开源项目的步骤：

1. Fork自己想要参与的开源项目仓库：

   ![Snipaste_2022-05-25_21-07-52](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/Snipaste_2022-05-25_21-07-52.png)

2. 克隆Fork回来的仓库，一定要从自己的账号下clone仓库，这样你才能推送修改：

   ``` gas
   git clone git@github.com:ilmangoi/learngit.git
   ```

   learngit的官方仓库`michaelliao/learngit`、自己在GitHub上克隆的仓库`ilmangoi/learngit`，以及自己克隆到本地电脑的仓库，他们的关系如下图所示：

   ```ascii
   ┌─ GitHub ────────────────────────────────────┐
   │                                             │
   │ ┌─────────────────┐     ┌─────────────────┐ │
   │ │ twbs/bootstrap  │────>│  my/bootstrap   │ │
   │ └─────────────────┘     └─────────────────┘ │
   │                                  ▲          │
   └──────────────────────────────────┼──────────┘
                                      ▼
                             ┌─────────────────┐
                             │ local/bootstrap │
                             └─────────────────┘
   ```

3. 给learngit添加一点内容，然后往自己Fork的仓库中推送：

   ``` gas
   $ git add *
   $ git commit -m "add a git notes"
   [master 7296579] add a git notes
    17 files changed, 1086 insertions(+)
    ... ...
    
   $ git push origin master
   Enumerating objects: 23, done.
   Counting objects: 100% (23/23), done.
   Delta compression using up to 8 threads
   Compressing objects: 100% (21/21), done.
   Writing objects: 100% (21/21), 393.32 KiB | 887.00 KiB/s, done.
   Total 21 (delta 2), reused 0 (delta 0), pack-reused 0
   remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
   To github.com:ilmangoi/learngit.git
      f38a6b3..7296579  master -> master
   ```

4. pull request：

   > 提交pull request

   ![Snipaste_2022-05-25_21-28-19](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/Snipaste_2022-05-25_21-28-19.png)

   > 查看已提交的pull request

   ![Snipaste_2022-05-25_21-30-02](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/Snipaste_2022-05-25_21-30-02.png)

## wroktree

> 一个git仓库可以支持多个工作树，分别对应不同的分支。我们在git中通过"git init"或"git clone"创建一个工作区（main working tree）。同理，我们使用git worktree创建一个不同目录的工作区，我们称之为为"链接工作区（linked working tree）"。git仓库有一个主工作树和零个或多个链接工作树。与重建的孤立的目录不同，链接工作树和主仓库是有机关联的，任何一个链接工作树的变更提交都在仓库内部。链接工作树用完后，可以直接通过git worktree remove删除。

git worktree方案可以概括为：通过创建共享版本仓库的多个工作区，实现多分支并行开发，从而实现多个工程环境的缓存，达到提升开发效率的目的。具体地：

- 可为一个分支创建一个工作区

- 每个工作区的工程环境独立运行

- 每个工作区共享同一个版本仓库信息

相比通过git clone方式创建多个独立工程环境的工作区，git worktree的优点在于：

- 更节省硬盘空间

  git clone方式下，每个工作区都有一个版本仓库

  git worktree方式下，每个工作区共享同一个版本仓库，节省了n-1/n（n为工作区数量）的硬盘空间

- 各个工作区之间的更新同步更快

  git clone方式下，A工作区和B工作区同步更新的路径：A工作区commit - A工作区push - B工作区pull

  git worktree方式下，A工作区只要本地提交更新后，其他工作区就能立即收到（因为它们共享同一个版本仓库）

![](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/v2-1688c37a2c59e782b3d7e6b4f75ad7f4_r.jpg)

1. 工作树的创建和创建新分支一样简单而高效。运行下面的格式创建工作树：

   ``` base
   git worktree add -b ../工作树目录 分支
   ```

   该命令会在../工作树目录下，创建一套完整分支工作区。该目录可以任意指定，但是最好在主仓库目录之外，免得污染仓库。然后就可以在该目录下检出分支，向上游推送，等等。如果分支不存在，则可以用-b操作，可以新建分支并使这个新分支关联到工作树。

2. list功能会列出每个工作树的详细信息：

   ``` base
   git worktree list --porcelain]
   ```

   --porcelain 选项，可以列出更完整的哈希值和分支信息。

3. *移动worktree到其他目录* ：

   ``` base
   git worktree move <worktree> <new-path> 
   ```

4. *清除那些检出目录已经被删除的worktree* ：

   ``` base
   git worktree prune
   ```

5. *删除worktree, 同时删除检出目录* ：

   ``` base
   git worktree remove -f <worktree>
   ```

   注意：该命令只能删除干净的工作树（没有未跟踪的文件，也无法对跟踪的文件进行任何修改）。不干净的工作树或带有子模块的树需要使用--force删除。主工作树无法删除。

## 其它

### 修改提交信息

> 如果提交commit之后，发现提交信息有误，比如说提交人、提交人邮箱或者提交信息错误，则不需要回退版本，则可以直接修改

```bash
git commit --amend -m "这里填写提交的注释" --author="username <email>" --no-edit
```

如果直接输入

```bash
git commit --amend
```

则会进入编辑模式，按i键然后直接编辑注释信息，编辑完成后则可以ZZ保存并退出

### rtab与btag

> #### btag:
>
> **用途：**用来记录一次测试环境打包的产物名称，可以用来作为查找某一次功能打包的时间点，大多数情况下作为发布线上的tag_name
>
> #### rtag：
>
> **用途：**用来记录线上环境打包的产物名称，可以用来作为查找某一次功能打包的时间点，大多数情况下作为<u>回滚线上发布</u>的tag_name

**rtag产生的过程：**

分支部署 → 生成btag  → 发布线上，生成rtag

首先确认需要上线分支的已经是最新代码，btag是由已经diff过的最新代码打包生成，并且在测试环境验证无其他问题，再确认发布上线的btag是正确的，来确保rtag是正确的

备注：若为回滚的rtag，一定要与开发确认该rtag的正确性（一般使用上一次线上发布的rtag）
