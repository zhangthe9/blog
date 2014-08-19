---
layout: post
title: git flow
description:
- git flow
categories:
- git
tags:
- git flow
---
http://zeroq.me/p/451
http://www.cnblogs.com/wubaiqing/archive/2011/12/18/2271724.html
###git flow 安装
git clone --recursive git://github.com/nvie/gitflow.git gitflow
此命令连同包含的submodule一同下载

http://gnuwin32.sourceforge.net/packages/util-linux-ng.htm
下载util-linux package,包含需要用的getopt.exe,libintl3.dll,直接copy到git\bin

gitflow\contrib\msysgit-install.cmd

git flow使用经验小记
http://www.ooso.net/archives/588

实际使用过程中有一些小小的意外流程，完全照搬git flow的模型不太容易处理好
好在git本身就很灵活，碰到问题基本上都有办法绕过去

一些特例情况下的处理办法

####测试/共享单独一个feature

有时候我们需要将一个feature独立测试，或者share给多人一块开发，那么可以将这个feature推到远程git库上，这可以利用git flow的publish功能搞定:

git flow feature publish my_cool_feature

这会将 feature/my_cool_feature 分支push到远程git库，多人开发或者单独测试毫无压力

####feature在development分支测试完成，准备release的时候有另外一个未经测试的feature合并进来

已经完成测试的development被未经测试的提交污染了，这时候可以先本地回滚development分支，然后再进行git flow的release流程

git checkout development
git reset --hard 测试通过的git rev
git flow release start v1.0.1
git flow release finish v1.0.1

####development上有个feature需要测试比较长时间，影响了一些耗时较短的feature发布

development分支上有个feature测试时间比较长一直释放不了，怎么办？—— 果断采用hotfix功能

git branch -m feature/another_cool_feature hotfix/another_cool_feature

把耗时短的feature直接转换为hotfix，然后采用git flow的hotfix流程可以直接合并到master分支发布

###another
http://ihower.tw/blog/archives/5140/comment-page-1#comment-60322

push 一个feature branch 到远端：
`git flow feature publish some_awesome_feature`
本质是`git push origin feature/some_awesome_feature`

追踪一个远端branch:
`git flow feature track some_awesome_feature`
本质是`git checkout -b feature/some_awesome_feature -t origin/feature/some_awesome_feature`

刪除远端branch：
`git push origin :feature/some_awesome_feature`

####git flow feature finish 出现错误:
warning: not deleting branch 'feature/some_awesome_feature' that is not yet merged to
'refs/remotes/origin/feature/some_awesome_feature', even though it is merged to HEAD.
error: The branch 'feature/some_awesome_feature' is not fully merged.
If you are sure you want to delete it, run 'git branch -D feature/some_awesome_feature'.


原因是这个 feature branch 一开始是从远端 checkout 出来的，以及这个 feature branch 有 commit 没有 push 回去 
所以 git flow 不敢帮你删除 local branch，这时候其实 merge 动作已经完成了
所以可以手动输入 git branch -D feature/some_awesome_feature 强制删除 local branch 即可
(小结论：git-flow 只是个辅助工具，了解 git 还是必要的)

####关于 feature branch 的合并
如果是开发时间比较久的 feature branch，很可能会因为
1. 不定时的 merge develop 与新版同步
2. 实验性质的修改
3. 需求的变更 等等因素，而让这个 feature branch 的 commit 记录变成脏的，这时候用以下的方式来做 merge 动作：

1. 先对 feature branch 做 `git rebase develop`
会很苦，但是弄完会很有成就感，整个 branch commit history 会变成很干净
请学 interactive mode，可以让你拿掉一些 commit、合并或修改，可以 rebase 多次直到满意为止

2. 在从 develop bracnh 做 `git merge feature/some_awesome_feature –no-ff`，–no-ff 的意思是会强制留一个 merge commit log 记录
这可以让 commit tree 看清楚发生了 merge 动作
(因为刚做了 rebase，而 git 默认的合并模式是 fast-forward，所以如果不加 –no-ff 是不会有 merge commit 的) 
这个 merge commit 的另一个额外方便之处是，如果想要 reset/revert 整个 branch 只要 reset/revert 这个 commit 就可以了

3. 如果此 feature branch 有 remote branch，要先砍掉 
`git push origin :feature/some_awesome_feature` 再
`git push origin develop` 
(这是因为 rebase 一个已经 push 出去的 repository，然后又把修改的 history push 出去，会造成超级大灾难啊~)
先 rebase 再 merge –no-ff 这样做的好处到底是什么? 

![merge]({{ site.url }}/imgs/dev/git/git-flow/01.jpg)

每一次的 merge 就代表了一个 feature 完成，也可以很清楚看到这个 feature branch 底下包含哪些 commit


###Git and Github 演講投影片
http://ihower.tw/blog/archives/5391