---
layout: post
title: TopGit
categories:
- git
tags:
- topgit
description:
- topgit原理及win下安装
---
###Topgit 原理
下面的分支图，是一个近似的 Topgit 实现图（略去了重要的 top-bases 分支）
<pre>
<xmp>
                    +---b1--M1---M3--- （特性分支B: refs/heads/t/B）
                    |              /        /
   +---a1---a2-------a3----M2---   （特性分支A: refs/heads/t/A）
   |                                  /
---V1--------------------V2---     （主线/卖主分支: master）
   |                                  \
   +---------------c1---M4---   （特性分支C: refs/heads/t/C）
</xmp>
</pre>
参见
http://www.worldhello.net/2010/10/28/2033.html

主线上的 V1 是上游的版本(master)的一次提交
特性分支 A 和 C 直接依赖主线 master，而特性分支 B 依赖特性分支 A
提交 M1 是特定分支 B 因为特性分支 A 更新(a3)而做的一次迁移
提交 M2 和 M4，分别是特性分支 A 和 C 因为上游(master)出现了新版本 V2 而做的迁移
当然特性分支 B 也要做相应的迁移-- M3

上述的描述非常粗糙，因为这样的设计很难实现特性分支导出为补丁文件。例如特性分支B的补丁，实际上应该是 M3 和 M2 之间的差异，而绝不是 M3 到 a2 之间的差异

Topgit 为了能够实现分支导出为补丁，又为每个特性的开发引入了第二个分支，追踪分支依赖的“变基”
<pre>
<xmp>
   +---a1---a2
   |                 |
   |                 +---a3---M2--- （特性 B 的变基分支: refs/top-bases/t/B）
   |                                 /
   +-------------------V2---   （特性 A 的变基分支: refs/top-bases/t/A）
   |                              /
---V1----------------V2---     （主线/卖主分支: master）
   |                              \
   +-------------------V2---   （特性 C 的变基分支: refs/top-bases/t/C）
</xmp>
</pre>


把上面的两张分支图重合，实际上就可以获得各个特性分支在任一点的特性补丁文件
上面的特性分支 B 还只是依赖一个分支，如果出现一个分支依赖多个特性分支的话，情况就会更加的复杂，更会体现出这种设计方案的精妙
Topgit 还在每个特性分支工作区的根目录引入两个文件，用以记录分支的依赖以及关于此分支的说明

- 文件 .topdeps 记录该分支所依赖的分支列表  
该文件通过 tg create 命令在创建特性分支时自动创建，或者通过 tg depend add 命令来添加新依赖
- 文件 .topmsg 记录该分支的描述信息  
该文件通过 tg create 命令在创建特性分支时创建，也可以手动编辑

###Topgit 在Win下安装
topgit完全就是Linux 里的shell脚本
像linux中的源码安装一样,要用到make等工具,把源码里的部分内容,如路径,替换,然后复制到目标路径
win下模拟linux环境的主要有cygwin,msys,mingw
简单脚本,我不喜欢cygwin,在外面套了层,下载还麻烦,虽然用cygwin没以下这几步操作
我要直接在win下的cmd里进去,敲命令就行
win下jekyll安装的时候,用到的railsinstallert里面自带了msys环境

要正常使用,需要做几步操作
####修改tg-create文件(回车换行问题)
line 119
echo "$deps" | sed 's/$/\r/g' >"$root_dir/.topdeps"
line 137
} | sed 's/$/\r/g'>"$root_dir/.topmsg"
参考 [linux shell 用sed命令在文本的行尾或行首添加字符](http://www.cnblogs.com/aaronwxb/archive/2011/08/19/2145364.html)
####建fgrep(报错问题)
railsinstaller安装后的devkit\bin有fgrep.exe
topgit脚本里会用到fgrep命令,调用到fgrep.exe会报错
解决:
建个fgrep,无扩展名,放git\bin目录(从msys提取的)
\#!/bin/sh
grep -F "$@"
####下载新版mktemp.exe,放git\bin(报错问题)
mktemp -v
mktemp version 1.6

不修改tg-create
Z:\dd>sh
sh-3.1$ 
`tg create t/hack1`
tg: Automatically marking dependency on master
tg: Creating t/hack1 base from master...
Switched to a new branch 't/hack1'
warning: LF will be replaced by CRLF in .topdeps.
The file will have its original line endings in your working directory.
warning: LF will be replaced by CRLF in .topmsg.
The file will have its original line endings in your working directory.
tg: Topic branch t/hack1 set up. Please fill .topmsg now and make initial commit.
tg: To abort: git rm -f .top* && git checkout master && tg delete t/hack1

不建fgrep
0 [main] fgrep 4492 open_stackdumpfile: Dumping stack trace to fgrep.exe.stackdump
mv: cannot move \`.git/hooks/pre-commit+' to \`.git/hooks/pre-commit'
.git/hooks/pre-commit: line 2: /pre-commit: No such file or directory
.git/hooks/pre-commit: line 2: exec: /pre-commit: cannot execute: No such file or directory

msysgit自带mktemp在sh里被执行时,同上,会报错,生成dump
1.6版的mktemp执行时,会输出一些版权之类说明,我用ollydbg去掉了,不影响

###TopGit的使用技巧
[TopGit的使用技巧1](http://www.worldhello.net/2010/03/10/796.html)
[TopGit的使用技巧2](http://www.worldhello.net/2010/03/10/800.html)
[TopGit的使用技巧3](http://www.worldhello.net/2010/03/10/802.html)


TopGit是Git的一个分支及补丁管理工具,可以很好的管理基于分支的开发模式,尤其对于二次开发人员来说，就是杀手锏
首先原型版本的代码始化成Git库，然后每给原型增加一个功能或者修改一个Bug都用TopGit创建一个分支（tg create t/feture1 或者 tg create t/bug1-fix）,这样每一个功能或者Bug修复都是一个TopGit分支

原型版本升级后，只要切换到原型分支(master)，然后导入原型的新版本代码，最后将TopGit的分支一个一个迁移到新的原型之上，就完成了一次版本的升级

####关于分支update
- 没有冲突的update操作

git init 
...
git add .
git commit -m init

`tg create t/hack1`
...
git add .
git commit -m init
`tg summary`        #查看当前TopGit分支的状态
t/hack1                         [PATCH] t/hack1
`git rev-parse master top-bases/t/hack1 t/hack1`      #查看master   top-bases/t/hack1   t/hack1指向的提交号
前两个相同,最后一个不同(master和top-base相同,t/hack1修改过)

修改master,致使 t/hack1过期 

`tg summary`     #D 代表t/hack1已经过期了，需要update
D   t/hack1                         [PATCH] t/hack1
`git rev-parse master top-bases/t/hack1 t/hack1`
三个都不同(master修改了,top-base不变,指向原有master,t/hack1不变)

更新t/hack1分支 
`git checkout t/hack1`
`tg update`
先是master Fast-forward
然后 Merge made by recursive.

- 含有冲突的update操作
- 含有分支依赖且产生冲突的update操作

 
