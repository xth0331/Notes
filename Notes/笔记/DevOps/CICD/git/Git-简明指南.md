# Git - 简明指南

## 创建新仓库

创建新文件夹，然后打开，执行:

```bash
# 创建新的git仓库
git init    
```

## 检出仓库

执行如下命令创建一个本地仓库的克隆版本：

```bash
git clone /path/to/repository
```

如果是远端服务器上的仓库，命令会是这个样子：

```bash
git clone username@host:/path/to/repository
```

##工作流

你的本地仓库由git维护的三棵“树”组成。第一个是你的`工作目录`，它持有实际文件；第二个是`暂存区(Index)`，它像是个缓存区域，临时保存你的改动；最后是`HEAD`，它指向你最后一个提交的结果。

## 添加和提交

你可以提出更改(把它们添加到暂存区)，使用如下命令：

```bash
git add <filename>
git add *
```

这是git基本工作流程的第一步；使用如下命令以实际提交改动：

```bash
git commit -m "代码提交信息"
```

现在，你的改动已经提交到了HEAD，但是还没有你的远程仓库。

## 推送改动

你的改动现在已经在本地仓库的HEAD中了。执行如下命令以将这些改动提交到远端仓库：

```bash
# 可以把master换成你想要推送的任何分支。
git push origin master
```

如果你还没有克隆现有仓库，并欲将你的仓库连接到某个远程服务器，你可以使用如下命令添加：

```bash
git remote add origin <server>
```

如此你就能够将你的改动推送到所添加的服务器上去了。

## 分支

分支是用来将特性开发绝缘开来的。在你创建仓库的时候，master是“默认的”分支。在其他分支上进行开发，完成后再将它们合并到主分支上。

创建一个叫做`feature_x`的分支，并切换过去：

```bash
git checkout -b feature_x
```

切换回主分支：

```bash
git checkout master
```

再把新建的分支删掉：

```bash
git branch -d feature_x
```

除非你将分支推送到远端仓库，不然该分支就死不为他人所见的：

```bash
git push origin <branch>
```

## 更新与合并

要更新你的本地仓库至最新改动，执行：

```bash
# 在你的工作目录中获取（fetch）并合并（merge）远端的改动。
git pull
```

要合并其他分支到你的当前分支（例如master），执行：

```bash
git merge <branch>
```

在这两种情况下，git都会尝试去自动合并改动。遗憾的是，这可能并非每次都成功，并可能出现冲突（conflicts）改完之后，只需要执行如下命令以将他们标记为合并成功：

```bash
git add <filename>
```

在合并改动之前，你可以使用如下命令预览差异：

```bash
git diff <source_cranch> <target_branch>
```

## 标签

为软件发布创建标签是推荐的。这个概念早已存在，在SVN中也有。你可以执行如下命令创建一个叫做1.0.0的标签：

```bash
git tag 1.0.0 1b2e1d63ff
```

1b2e1d63ff 是你想要标记的提交ID的前十位字符。可以使用下列命令获取提交ID：

```bash
git log
```

你可以使用少一点的提交ID前几位，只要它的指向具有唯一性。

## log

如果你想了解本地仓库的历史记录，最简单的命令就是使用：

```bash
git log
```

你可以添加一些参数来修改他的输出，从而得到自己想要的结果。

只要某一个人的提交记录：

```bash
git log --author=bob
```

压缩后的每一条提交记录只占一行输出：

```bash
git log --pretty=oneline
```

或者你想通过ASCII艺术的树型结构来展示所有的分支，每个分支都标示了他的名字和标签:

```bash
git log --graph --oneline --decorate --all
```

看哪些文件改变了：

```bash
git log --name-status 
```

这些只是你可以使用的参数中很小的一部分。更多信息，参考：

```bash
git log --help
```

## 替换本地改动

加入你操作失误，可以使用如下命令替换掉本地改动：

```bash
git checkout -- <filename>
```

此命令会使用HEAD中的最新内容替换掉你的工作目录的文件。已添加到暂存区的改动以及新文件都会不收到影响。

假如你想丢弃在你本地的所有改动与提交，可以到服务器上获取最新的版本历史，并将你的本地主分支指向它：

```bash
git fetch origin
git reset --hard origin/master
```

## 实用小贴士

内建的图形化 git：
`gitk`
彩色的 git 输出：
`git config color.ui true`
显示历史记录时，每个提交的信息只显示一行：
`git config format.pretty oneline`
交互式添加文件到暂存区：
`git add -i`

