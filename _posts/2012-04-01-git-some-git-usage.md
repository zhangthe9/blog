---
layout: post
title: 一些git常用方法
categories:
- git
tags:
- git
description:
- 一些git常用方法
---
###ssh 连github
ssh-keygen -t rsa -C "admin@ddatsh.com"
生成id_rsa和id_rsa.pub
然后到github ，SSH Keys加入公钥

cmd下
ssh -v -T git@github.com
查找密钥路径要设置
set HOME=%USERPROFILE%

###分支管理一例
Git 的分支不需要文件目录。从一个分支转到另外一个分支的时候，自动重建现场
例如，在一个分支里先有3个文件，然后开了一个新的分支，并提交了几个新文件以及修改了2个文件。当切换到第一个分支的时候，工作目录里恢复到原先的3地个文件，其内容也是先前的内容，没有改动过

分支不仅仅用来管理程序的版本，而且可以用来辅助尝试性程序开发，称为主题分支/专题分支(Topic Branch)

因此用熟了 Git 的人常常会在做任何事情以前开个分支，如果成功就合并到主分支，如果不成功则简单删了这个试验分支

有时候也会忘记先开分支，直到改了一批文件后才发现应该先开个分支
当没有提交时，只要开个新分支。等提交修改的内容的时候，它们会在新开的分支里出现，而不是在老的分支里

即使已提交过时，也很容易修正过来
例如master 当前最新的提交准备放在新分支
`git branch xx` #开一个新的
`git reset --hard head~1`

###空分支(没有parent)
三种方法
1.
`git symbolic-ref head refs/heads/empty`
`rm .git/index`
`git clean -fdx`
2.
echo ref: refs/heads/newbranch > .git/HEAD
git rm -rf .
3.Git 1.7.2后 –orphan
git checkout --orphan newbranch
git rm -rf . 

###git cherry-pick 和 rebase 混合
假设master分支commit两次(A,B)之后建了topic1分支
在topic1分支提交了C-D-E-F-G

然后只想把 D,F 加到 master ，其他的留在 topic ，而且又不想把 history 搞得很乱
最好是这样:
<xmp>
A-B-D-F  master
      \
        C-E-G topic
</xmp>
`git checkout master`
`git cherry-pick D`
`git cherry-pick F`
`git checkout topic`
`git rebase master`

###使用 git rebase 避免无谓的 merge
git pull 默认行为是将远端的 repo与本地repo合并，这也是DVCS的初衷
但是，很多时候会发生以下这种情形
![merge]({{ site.url }}/imgs/dev/git/git-usage/merge.jpg)

原因:
本地branch与远端branch同步非常频繁（通常是同名的branch,如 master）
这两个branch几乎完全同步，那这些merge动作其实没必要，造成历史线索无谓的复杂
这种情况推荐用
git pull --rebase 

1.把本地 repo从上次 pull 之后变化暂存
2.回到上次pull点
3.应用远程变更
4.再应用暂存的变更

假设合并前:
<xmp>
          D---E master
         /
A---B---C---F origin/master
</xmp>
<xmp>
merge 合并后：

         D--------E
        /                 \
A---B---C---F----G   master, origin/master
</xmp>
如果是 rebase 的方式，就不会有 G 点!

A---B---C---F---D'---E'   master, origin/master

注意，其中 D’, E’ 的 commit SHA 与本來 D, E 是不同的，因为算是砍掉重新 commit 了

有conflict怎么办?
rebase 跟 merge类似，出现 conflict 一样会暂停 rebase 动作，需要手动修复后，才能继续
这也是rebase比merge复杂一点的地方:merge发生conflict,只要解决冲突一次，然后commit出去就完成了
而 rebase 的 conflict 可能会发生在上述步骤4的每一步重新套用上,所以可能需要解决冲突好几次
(rebase时所谓的解决冲突，其实是直接修改之前的变更内容，所以上图变成D'和E')

修改比较多，预期会有较多conflict，可以用merge,不过如果是多次大范围的主题式修改，应该一开始就开一个branch
如果修改范围较小，不太会产生conflict，可以用rebase

避免每次git pull 都要 加 --rebase
.git/config 加
[branch "master"]
  remote = origin
  merge = refs/heads/master
  rebase = true

也可以直接加到 ~/.gitconfig 设全局
[branch]
  autosetuprebase = always

###git rebase 冲突了, 又不喜欢一步一步的 git rebase 怎么办?
git rebase --abort
git reset --hard HEAD
git pull
git status


