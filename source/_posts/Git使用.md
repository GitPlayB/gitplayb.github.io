---
title: Git使用
date: 2019-12-22 22:54:48
tags:
  - Git
  - 黄金屋
categories:
  - BUG制造者
---

# Git使用

## 1. 基础

### 创建工作目录

一般有两种方式：`git init`或者`git clone url` 

**前者**会在当前选定目录下创建一个**.git**隐藏目录（Windows上需要把隐藏文件查看勾选上才可以看到），即本目录将会作为一个本地仓库可以与远程仓库连接。打开.git目录可以看到里面含有一些此仓库的基本信息，主要是记录分支信息还有文件的更改情况等。

**后者**可以直接从远程仓库中复制一份完全一样的代码，到你的本地目录，作为本地仓库（前提是你配置好了Git客户端如GitHub的用户名/密码或者SSH秘钥）。



### 记录仓库中的改变

**版本控制系统**最大的作用就是**记录、跟踪文件的改变，使用户可以方便地在不同版本间切换**。对于Git，基本状态有四个：**Untracked、Unmodified、Modified、Staged**。

- **Untracked**——未被git加入跟踪，即文件怎么改变git都不管
- **Unmodified**——已跟踪、未修改，git记录其改变情况，但现在还没有改变
- **Modified**——已修改、未暂存，已经发生了改变，并且git记录了此改变
- **Staged**——已暂存，已将改变情况告诉git，加入暂存区，作为下次commit（提交）的材料



#### 跟踪/记录文件

当我们想让git为我们记录一个文件的改变时，使用`git add`命令，后面跟着的是想要跟踪的文件名。这时我们查看git状态（`git status`）会发现，状态已经由**untracked**变成了**tracked**，终端会显示*Changes to be committed*。即，我们已经到达提交前的一个状态：所有改变已被记录并暂存。如果提交之前又对加入跟踪的文件作了改变，还需要再次运行`git add`命令，将改变暂存，然后提交。

由此可以看出，`git add`命令有两个作用：

1. 开始追踪/记录文件
2. 将文件（改变）提交到暂存区

#### 

#### 查看状态（简版）

查看状态有两种方式：`git status`或者`git status -s`或者`git status --short`。

前者查看较为详细的状态；后者查看简要状态。

其中后者的状态表示有两列：**第一列表示暂存区的状态；第二列表示远程仓库的状态。**`??`表示为追踪；`A`代表已暂存；`M`代表已修改。

在git bash中操作一番发现，**已修改但是没有暂存时`M`出现在右边且是红色的；修改且暂存后`M`出现在左边且是绿色的**。如下：

![git-status-s](Git使用\git-status-s.png)

显示还是挺人性化的，红色表示警告，暂存之后，就变成了绿色，表示正常状态。

如果做了改变，又没有commit，就会出现下图的情况：

![1573698024559](Git使用\1573698024559.png)

使用`git diff`查看具体区别：

![1573697956849](Git使用\1573697956849.png)

使用`git add -A`将所有改动文件暂存：（详细版本会说“changes to be committed”，并且会以绿色字体列出改动的文件）

![1573698564785](Git使用\1573698564785.png)

使用commit命令提交后，再看status

![1573698789058](Git使用\1573698789058.png)

提醒你要push代码，将改动同步到远程仓库

使用`git status -s`则会什么都不显示



#### 忽略文件

通常有一些文件，你是不想让git追踪的，即你怎么改变都是在本地改变的，不需要git干预（因为默认情况下git会把**.git**所在目录下所有文件都当成潜在客户，即你没有add（追踪），它会缠着你让你追踪）。这时，我们就需要一个新建一个名为**.gitignore**的隐藏文件，告诉git，哪些文件不需要追踪。

文件的书写格式如下：

- 空行或以**#**开头（注释）的行会被忽略
- 支持部分**正则表达式**
- 开头加**/**避免递归
- 末尾加**/**指定一个目录
- 开头加**!**表示否定

示例的.gitignore文件：

```
# ignore all .a files
*.a
# but do track lib.a, even though you're ignoring .a files above
!lib.a
# only ignore the TODO file in the current directory, not subdir/TODO
/TODO
# ignore all files in any directory named build
build/
# ignore doc/notes.txt, but not doc/server/arch.txt
doc/*.txt
# ignore all .pdf files in the doc/ directory and any of its subdirectories
doc/**/*.pdf
```

以上的描述已经很清楚了，主要就是通配符*****的使用。



#### 查看暂存/未暂存的文件

使用`git diff`。

如果改动了某个文件，使用diff可以查看**已暂存/未暂存**的文件，已暂存的文件下次commit将会被提交。

使用diff时可以加上一个参数：`git diff --cached`或者`git status --staged`，没什么区别。

还可以在其它外部工具（不是terminal）中使用git diff，使用`git difftool --tool-help`查看本机可以用的比较工具。



#### 提交

使用`git commit`。

输入此命令会自动打开git bash配置的默认编辑器（未设置是vim），你可以写上本次提交的描述；

或者，直接`git commit -m "message"`加上描述；

如果不想每次commit前都要暂存（使用`git add`），可以在提交时加上**-a**参数，即`git commit -a`。



#### 删除文件

在git上删除一个文件，你得首先让git不要再追踪（track）它，然后提交。使用`git rm`就可以做到，并且也会将它从电脑磁盘上删除。如果直接使用**rm**，会提示：changes not staged for commit

使用`git rm`，按下回车会反馈：删除了哪个文件。再查看status，会记录之前的操作--删除文件，并等待提交：

```bash
$ git rm PROJECTS.md
rm 'PROJECTS.md'
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
  
    deleted: PROJECTS.md
```

删除以后，下次提交git就不会管它了。如果已经修改了这个文件或者已经把它add到了暂存区，删除时要使用**- f**强制删除，这是为了防止意外删除，因为删除了无法恢复。

还有一种情况，你只想把文件从暂存区删除，但是保留在你的电脑磁盘上。这就好比你忘了在.gitignore中忽略这个文件。删除时添加**--cached**参数，即：

```bash
$ git rm --cached README
```

在命令中也可以使用通配符*，如：

```bash
$ git rm log/\*.log
```

上面命令将删除所有log/目录下的.log文件（好像是遇到*需要处理成转义字符）

```bash
$ git rm \*~
```

上面命令会删除所有以~结尾的文件



#### 移动文件/重命名

不像其他的VCS（版本控制）系统，Git不会显式地追踪移动操作，即你在磁盘目录上对文件移动不会被记录到git的状态中。【但是！】Git提供了mv命令，如果使用mv命令进行移动，则会被记录在status中。如：

```bash
$ git mv README.md README
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
  
    renamed: README.md -> README
```

重命名操作被Git记录了下来，此操作相当于以下的命令：

```bash
$ mv README.md README
$ git rm README.md
$ git add README
```

即，如果仅仅是在磁盘上对文件进行移动，完了还要告诉Git，将暂存区的旧文件移除，新文件暂存。

这样看来，使用Git提供的移动/重命名操作，还是很方便的（**或者直接在文件夹里修改名字就好了，Git不会记录**）。



### 查看提交历史

使用`git log`命令查看提交历史记录。







# ###############廖雪峰

**git撤销回退版本和切换分支的命令都可以是`git checkout`**，但切换分支还可以用**`git switch`**，所以最好用switch切换分支以区分。

**git合并时默认使用fast forward模式**