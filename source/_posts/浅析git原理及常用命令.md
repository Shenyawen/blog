---
title: 浅析git原理及常用命令
date: 2018-08-31 16:40:00
tags: git
---

git 码农必备，那下面简单了解下git的原理吧
### git层级关系：工作目录(workspace)、暂存区(index/stage)、本地仓库(local repository)和远程仓库(remote repository)
只有被提交到本地仓库的文件才会被git跟踪变化，但是在提交到本地仓库之前需要先添加到暂存区。这里说的提交和添加都是文件快照而不是实际文件
{% asset_img git__liucheng.png 目标图片 %}
{% asset_img git__banbenku.png 目标图片 %}
工作目录：工程文件
缓存区：提交代码跟解决冲突的中转站
本地仓库：接本地代码跟远程代码的枢纽，不能联网时本地代码可先提交至该处
远程仓库: 即保存我们代码的远端服务器
其中暂存区跟本地仓库都保存在.git文件中

### .git文件详解
当有一个新目录或者已有目录执行git init时，git会新建一个.git目录文件。目录结构如下：
1.description文件：仅供GitWeb程序使用
2.config文件：包含项目特有的配置选项
3.info目录：包含一个全局性排除(global exclude)文件，用以放置那些不希望被记录在 .gitignore文件中的忽略模式(ignored patterns)
4.hooks目录：包含客户端或服务端的钩子脚本(hook scripts)
5.***HEAD***文件：指出目前被检出的分支即指向你当前所在分支的引用标识符
6.index文件：保存暂存区信息
7.objects目录：存储所有数据内容
8.refs 目录：存储指向数据（分支）的提交对象的指针

### 常用命令
```bash
// 在本地生成git管理即生成.git文件，git初始化
$ git init
```

```bash
git add files // 把当前文件放入暂存区域
```

```bash
git commit // 给暂存区生成文件快照并提交到本地仓库(当前分支)，此过程在本地进行不涉及网络请求
```

### 分支内部管理
1.如下图所示，版本的每一次提交(commit)，git都将他们根据提交的时间点串联成一条线。刚开始的时候只有一条线，即master分支，HEAD指向当前分支的当前版本
{% asset_img git_fenzhi01.png 目标图片 %}

2.当创建了新分支，比如dev分支（通过命令git branch dev完成）, git 新建一个指针dev, dev=master, dev指向master指向的版本，然后切换到dev分支(通过命令git checkout dev完成)，把HEAD指向dev，如下图：
{% asset_img git_fenzhi02.png 目标图片 %}

3.在dev分支上编码开发时，都时在dev上进行指针移动，比如在dev分支上commit一次，dev指针往前移动一次，但是master指针没有变，如下图
{% asset_img git_fenzhi03.png 目标图片 %}

4.当我们完成了dev分支上的工作，要进行分支合并，把dev分支的内容合并到master分支上（通过首先切换到master分支，git branch master，然后合并git merge dev命令完成）。其内部的原理，其实就是先把HEAD指针指向master，再把master指针指向现在的dev指针指向的内容。如下图。
{% asset_img git_fenzhi04.png 目标图片 %}

5.当合并分支的时候出现冲突（confict），比如在dev分支上commit了一个文件file1，同时在master分支上也提交了该文件file1，修改的地方不同（比如都修改了同一个语句），那么合并的时候就有可能出现冲突，如下图所示。
{% asset_img git_fenzhi05.png 目标图片 %}
这时候执行git merge dev命令，git会默认执行合并，但是要手动解决下冲突，然后在master上git add并且git commit，现在git分支的结构如下图。
{% asset_img git_fenzhi06.png 目标图片 %}
可以使用如下命令查看分支合并情况。
```bash
git log --graph --pretty=oneline --abbrev-commit 
```

### 分支策略
如何合适地使用分支？

在实际开发中，我们应该按照几个基本原则进行分支管理：
1、master分支应该是非常稳定的，也就是仅用来发布新版本，平时不要在master分支上编码开发。master分支应该与远程仓库保持同步；
2、平常编码开发都在dev分支上，也就是说，dev分支是不稳定的，到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本；*** dev分支也应该与远程保持同步 ***；（git push/git pull也要解决冲突）
3、你和团队成员每个人都在本地的dev分支上干活，每个人都有自己的分支，时不时地往远程dev分支上push/pull就可以了。（*** push/pull的时候是要解决冲突的. ***）
{% asset_img git_fenzhi.png 目标图片 %}

*** PS ***：git本没有公共服务器的概念，git的每个节点都是一个完整的git库，但是公共服务器是方便了git节点之间的代码互相push/pull（要不然每个git节点都需要互相连接，每增加一个git节点就要连接其他的git节点）这其实也是区块链的中心思想之一：最安全的中心即没有中心

[Git常用命令](https://www.cnblogs.com/libin-1/p/7712794.html)
[Git版本控制与工作流](https://www.jianshu.com/p/67afe711c731)
[Git远程操作详解](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)
[Git 少用 Pull 多用 Fetch 和 Merge](https://www.oschina.net/translate/git-fetch-and-merge?cmp)
