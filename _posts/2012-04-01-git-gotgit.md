---
layout: post
title: git权威指南 笔记
description:
- git权威指南
categories:
- git
tags:
- git
---
!!!!!!!!!!还没弄完，慢慢更新!!!!!!!!!!!!!
##序
请买 蒋鑫 的《Git权威指南》，不啰嗦
此文大部分内容是对此书内容的摘录、纪要
期间参考了许多站上的文章

自由軟體的故事
http://www.inmediahk.net/node/299503

版本管理器的发展史
http://coolshell.cn/articles/3288.html

##前言
   版本控制(scm)是管理数据变更的艺术，数据的变更会来自个人，不同的人(团队)
   要忠实地记录每一次变更，还要帮助还原任何一次历史变更，以及团队协作

   传统集中式版本控制只有唯一的分支命名空间
   分支间合并可能因缺乏追踪导致重复合并，引发冲突
   或者分支合并时效率低下和陷阱重重(git灵活的设计可以摆脱分支管理的梦魇)
   
   最终容易导致: 

- 项目延期
- 历史修正了的BUG重现
- 问题不方便在代码中定位出
   
<hr/>
基于开源软件的二次开发，最初用SVN，使用卖主分支模型管理代码
随着增加和修改的代码的增长，与上游偏离越来越远
上游新版本发布后，最初可能几小时就完成迁移，但上游改动数量庞大后，迁移过程就极痛苦
最终不再与上游合并，不再追踪上游改动，导致无法发动全球智慧为客户创造价值

##第一篇 Git由来
Git,Linux Kernel 发起人 Linus Torvalds 设计的版本管理系统,继Linux 内核后的第二个伟大作品
官方网站是 http://git-scm.com (已被GFW墙掉)

![git-scm]({{ site.url }}/imgs/dev/git/git-scm/git-scm.png)
新站截图
![git-scm]({{ site.url }}/imgs/dev/git/git-scm/new-git-scm.jpg)


   Linus 由开始一直没有使用版本控制系统去处理由社群贡献而来的源码，在开发社群中有人用 CVS 版本控制系统来增加效率，不过 Linus 始终不愿意使用
   CVS 的单一中央储存库的架构不能完美地配合 Linux Kernel的分布式开发
   2002 年 10 月在站上只有 FTP 和 HTTP 下载和电子邮件通信论坛
   那时谁去 SourceForge 登记，都可免费使用版本控制和待办事项等开发工具
   不少其他的开源项目的网站都设计得比 http://www.kernel.org 好

   Linus 为了改善效率，使用了非开源的分布式版本控制系统－BitKeeper
   除了源码不开放外，BitKeeper 作者 Larry McVoy 更加入了好几条特别条款
   最受争议的是 BitKeeper 用户不能参与 BitKeeper 以外任何版本控制系统的开发，也不能做 BitKeeper 的逆向工程

   Stallman 对 Linus 的决定作出严厉的批评，当然还有开源运动人士也不同意，以实务为考虑的 Linus 继续使用 BitKeeper

   2005 年 Samba 开发领袖 Andrew Tridgell 开发了一个连接 BitKeeper 储存库的程序，被 McVoy 指控为逆向工程。McVoy 因此停止 BitKeeper 对 Linux的支持。批评者之前的论点结果成为了事实：

1.Linux 的开发工具被他人控制了
2.开发社群因为特别条款，好几年也不能开发如 Git 合适社群使用的版本控制系统
3.开发社群因为没有 BitKeeper 的源码而不能增强这个版本系统的功能

   而且 McVoy 也不是容易跟人合作，他也有很强的控制欲，连已付钱的用户也不可以参加开发其他版本控制系统
   Linus 小看了软件自由的重要性，结果作出了错误决定，而且社群讨论BitKeeper 时常引起争论，增加了内耗

###第1章  版本控制前世今生
另参考 版本管理的发展史
####CVS出现前，黑暗的史前时代
   秩序和工具的匮乏
   代码分散地存在不同软盘里，不知道哪个盘的代码是最优的，最新的可能并非最优，失败的重构会毁掉原来正确的代码

从未使用甚至听说版本控制的人（团队），就像停留在**史前时代，任数据自生自灭**

####CVS
为SCM确立了标准，**commit log,check in(out),tag,branch**
cvs基于rcs(针对单独文件的版本管理工具)

####SVN
集中式版本控制集大成者
全局版本号，相对cvs的每文件独立版本
除集中式版本控制固有问题，如

    单点故障后，无法提交，最坏情况是丢失所有历史，因为客户端只有前一次快照

SVN的里程碑和分支设计被证明是一个错误
虽然这个错误设计使SVN有快速创建里程碑和分支能力
但导致更多问题

- 版本库目录结构要固定

要建类似 /trunk,/branch,/tag目录,否则无法建立分支和tag

- 破坏授权

创建分支或tag时，针对/trunk及子目录的授权要重建

- 分支混乱  

可以基于/trunk建分支，也可以基于任何目录建分支，难画出有意义的分支图
svn 1.5虽有了合并追踪，但会被混乱的分支管理而被抵消

<hr/>
linus是CVS反对者，也同样反对SVN
直到2002的十余年，宁愿用diff,patch,tar包维护内核代码,也不用CVS
2002-2005用商业分布式SCM-BitKeeper，到搞出GIT间的故事
linus以文件系统专家和内核设计者角度对GIT设计，使其性能和存储能力都优秀

越来越多令人激动的使用GIT的重量级项目

开发人员不会再因频繁遭遇提交冲突而中断
经过Linux这样庞大的项目考验后，被证明可以胜任任何规模的团队
不用把“鸡蛋”放在有单点故障，可能崩溃的“篮子” (集中式SCM服务器)


###第2章 爱上GIT的理由
####避免引用辅助目录
GIT不像svn,cvs会在每个目录下生成辅助目录,grep时结果不会加倍(svn 1.7只有一个.svn目录了)
####工作进度保存(stash)
临时有紧急任务，可以开辟出干净工作区开始工作,和保存当前未完成工作的进度
####更好的差异比较
git diff(水很深，涉及暂存区，GIT最精华设计之一)
####其他
本地查看日志，不再忍受网络状况不佳
后悔药：重写提交说明，最近一次（及历史）提交内容  --amend 和 rebase
便宜的分支

太多太多....

###第3章 安装相关
git源自 Linux ,版本更新较快
1.7.7 支持stash 未追踪文件(--include-untrack或-u)
1.7.10 国际化,merge时提示输入信息

我还是主要在WIN7下用
####msysgit
msysgit挺好使，不喜欢在cygwin里再装
1.7.10
还未包含国际化文件
msysGit 正式支持 unicode，版本库彻底告别乱码。控制台录入中文有问题，最好配合Unicode版的TortoiseGit
git config --global core.quotepath false
cmd 里git status,git log 中文文件名，LOG 都没问题
####tortoisegit
图形化的tortoisegit依赖msysgit
1.7.8 的64位版支持了ceth

###第4章 初始化
####最重要的
git config --global user.name xxx
git config --global user.email xxx

###最常用别名
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
自己弄了几个批处理更省事  如 l.cmd
<pre class="prettyprint">
@echo off
if "%1"=="" (
  git log --oneline
  goto end
)

if "%1"=="s" (
  git log --oneline --stat
  goto end
)

if "%1"=="g" (
  git log --oneline --graph
  goto end
)

if "%1"=="gs" (
  git log --oneline --graph --stat
  goto end
)

if "%1"=="f" (
  git log --pretty=fuller
  goto end
)

if "%1"=="r" (
  git log --pretty=raw
  goto end
)
:end
</pre>

####三步曲
<pre class="prettyprint">
git init
git add
git commit
</pre>
####--global 和--system
--global改的C:\Users\Administrator\.gitconfig
--system改的git\etc\.gitconfig

android更好的管理，弄了Gerrit审核服务器管理GIT提交

##第二篇 git独奏
git有陡峭的学习曲线,对有其它版本控制工具使用经验的老手也不例外
**不能按照其他版本控制系统遗留习惯操作git,努力在git寻找对应物** 而
最终因为git的“别扭”而放弃

###第5章 暂存区
git最成功的设计之一，也是最难理解的
同一文件可能有**三种状态：工作区，暂存区，版本库**
git diff
git diff --head
git diff --cached


<table>
<tr>
<td>git reset HEAD</td>
<td>暂存区目录树被master分支指向目录树重写</td>
<td>工作区不受影响</td></tr>
<tr>
<td>git rm --cached</td>
<td>直接从暂存区删除文件</td>
<td>工作区不受影响</td></tr>
<tr>
<td>============</td>
<td>危险的操作</td>
<td>==============</td></tr>
<tr>
<td>git checkout .或<br/>git checkout -- <file></td>
<td>用暂存区替换工作区</td>
<td>清除工作区未暂存操作</td></tr>
<tr>
<td>git checkout head .或<br/>git checkout head -- <file>&lt;file&gt;</td>
<td>HEAD指向的分支的文件替换暂存和工作区</td>
<td>清除工作区未暂存工作和<br/>暂存区未提交改动</td></tr></table>

###第6章 git对象
什么是head,master? 什么时候可以互换使用?
git很多对象以40字节的sha1表示

`git log -1 --pretty=raw`
除了初始提交，都有三个sha1值
commit 393c463fba7ab6868c49d9a48bad8d500d6a8566
tree 4617a709b688fabb8cf181b50f7ceac079547aee
parent f513abd772716211baacf12ff1a5c6d1a08bb528

`git cat-file -t sha1`  看对象类型
`git cat-file -p sha1`  看对象内容

.git\objects目录 sha1前两位作目录名，后38位作文件名

git ls-file,cat-file,ls-tree,rev-parse等几个命令要多理解理解
生成sh1相关的几个命令
commit,文件(blob),tree的sha1生成
<xmp>
git rev-parse head
git rev-parse blob head:welcome.txt
git rev-parse blob :welcome.txt  与上面相比，上面是库中，这个是查暂存区
git rev-parse head head^{tree} 这里的^特别，就是个语法，不是表示父一级
</xmp>
在cmd中不行，要在bash中

<xmp>
git cat-file commit head |wc -c

git cat-file blob head:welcome.txt |wc -c

git cat-file tree head^{tree} |wc -c
</xmp>
分别输出大小

(print "commit 大小\000"; git cat-file commit head)| sha1sum

(print "commit 大小\000"; git cat-file blob head:welcome.txt)| sha1sum

(print "commit 大小\000"; git cat-file tree head^{tree})| sha1sum

 

第7章. reset
git reset --hard hed^
在CMD下的BUG，无法回退到前一版本，在BASH中就行

如何拘留由于此步操作导致的提交丢失

reset 默认mixed方式,注意和--soft,--hard对工作区，暂存区的影响，以及在什么时候用
