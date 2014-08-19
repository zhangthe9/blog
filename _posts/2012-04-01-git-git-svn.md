---
layout: post
title: git-svn
description:
- git svn
categories:
- git
tags:
- git svn
---
###Nginx与github
nginx目前源码还是SVN管理，假设把它源码用git svn 弄过来
然后自己在github建个库,存放修改

git svn clone 默认从svn第一个版本开始同步,太慢
svn info http://your-svn 先看最后版本

`svn info svn://svn.nginx.org/nginx/trunk`
	路径: trunk
	RL: svn://svn.nginx.org/nginx/trunk
	版本库根: svn://svn.nginx.org/nginx
	版本库 UUID: 73f98a42-aea0-e011-b76d-00259023448c
	版本: 4600
	节点种类: 目录
	最后修改的作者: mdounin
	最后修改的版本: 4600
	最后修改的时间: 2012-04-18 22:47:10 +0800 (周三, 2012-04-18)


git svn clone -r版本号:HEAD --prefix=svn/ http://your-svn
`git svn clone -r4600:HEAD --prefix=svn/ svn://svn.nginx.org/nginx/trunk`
或
`git svn clone -s -r4600:HEAD --prefix=svn/ svn://svn.nginx.org/nginx`

这两个命令区别，前者目录叫trunk,后者就叫nginx


使本地svn分支对应svn的远程仓库，本地master分支对应github的远程仓库
默认在master分支，改名为svn,本地再建个master分支
`git branch -m svn`
`git remote add origin git@github.com:user/repoXX.git`
git checkout -b master

现在可以按照平常的习惯在git下更改代码，然后用git push到github上
如果需要更新代码到svn上，按这个流程操作就可以了：
`git checkout svn`
`git merge master`
`git svn dcommit`


###例子2
用户bob
<pre class="prettyprint">
z:
cd \
md repos
cd repos
svnadmin create hello

cd \
svn co file:///z:/repos/hello hello-svn
cd hello-svn
svn mkdir branches tags trunk
svn ci -m init

cd trunk
echo "printf("hello\n");"> hello.c
svn add hello.c
svn ci -m "My first program"

cd ..
svn cp trunk tags/v0.1
svn ci -m "tag v0.1"
</pre>
Bob 已经建好v0.1 版

Alice 用git-svn 取出 repository

`git svn clone --stdlayout file:///z:/repo/hello hello-git`
会报错
Couldn't open a repository: Unable to open an ra_local session to URL: Unable to open repository 'file:///z:/repo/hello': Expected FS format '2'; found format '4' at c:\git/libexec/git-core\git-svn line 1343
直接以file:/// 访问不行，启个svn服务，就行了

svnserve -d -r z:\repos\hello

`cd \`
`git svn clone --stdlayout svn://127.0.0.1 hello-git`
`cd hello-git`
`git reset --hard remotes/trunk`

使用 stdlayout，所以 tag/branch 等会自动对应 remote branch

cd hello-git
git branch -r
  tags/v0.1
  trunk

Alice 更新bob代码后 可以用git add -p 选择要commit的部分,剩下部分stash起来
然后 git svn dcommit提交给bob的svn repository

edit hello.c
git add -p
git commit -m'Fix everything'
git stash
git svn rebase 这等于在svn update 检查一下远端是否有其他人checkin了新东西
git svn dcommit

过几天，发现Bob有更新，要同步
同时，有一个local branch,所以在更新完master后，还要把fancy branch跟master rebase

git svn fetch

git svn rebase
First, rewinding head to replay your work on top of it...
Fast-forwarded master to refs/remotes/trunk.

git checkout fancy
git rebase master
First, rewinding head to replay your work on top of it...
Applying: Inital fancy output branch