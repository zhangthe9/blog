---
layout: post
title: 使用Git的分支和合并功能来进行版本管理的开发模型
description:
- 译文<br/>
- 分支的使用策略和发布管理<br/>
- a-successful-git-branching-model
categories:
- git
tags:
- git
---
###分支的使用策略和发布管理
<img width=100% src="{{ site.url }}/imgs/dev/git/a-successful-git-branch/01.png"/>

Git从根本上改变了开发人员对分支和合并的使用
传统的CVS/SVN，分支和合并都是高级话题，而且使用起来稍显麻烦，隔一段时间才会用一次
但是有了Git，这些操作就成了家常便饭

由于使用简单，方便重复操作，分支和合并不再是让人望而生畏的操作，版本管理工具应该尽可能地对分支/合并提供最好的支持

此模型其实是一些每个开发人员都应该遵守的步骤，如果想管理好软件的开发流程的话

###分布式但集中化
git技术层面没有中心库,实际管理时有被公认的"中心库"**origin**，Git用户都熟悉这个名字

<img width=100% src="{{ site.url }}/imgs/dev/git/a-successful-git-branch/02.png"/>

每个开发者pull和push到origin，但除了中心化的push-pull关系外，每个开发者还可以从其他开发者那pull changes
比如说，对于一个比较大的新特性，在把代码提交到origin之前，很可能会安排2个或多个开发者
上图中有几个小团队：Alice和Bob，Alice和David，Clair和David

从技术角度来说，其实就是Alice定义了一个叫Bob的Git remote，指向到Bob的仓库

###main分支
中心仓库有两个分支:

- master
- develop

origin上的master分支，Git用户应该很熟悉，跟master并行的有一个develop分支


<img width=50% src="{{ site.url }}/imgs/dev/git/a-successful-git-branch/03.png"/>

origin/master作为主要分支，源码的HEAD总是表示production-ready(可随时部署)状态
而origin/develop上的代码是为下一次的代码发布准备的。每日构建也是基于此分支

develop分支达到稳定状态并准备发布时，所有的改变都要合并到master分支，并标上版本号。如何实现的下面细说

这样每次与master合并都会有新的部署发布。这点可以自动化，如使用Git hook脚本来实现自动部署代码到线上

###支持(supporting)分支

开发模型使用了一些支持分支放在master和develop分支的旁边，方便开发小组之间的并行开发
不像main分支，这些分支是有时间限制的，因为他们最终都会被移除

会使用到的不同的分支

- Feature branches
- Release branches
- Hotfix branches

每个分支都有各自的作用，并且有严格的规定，如：只能从哪个分支上去新开分支，只能合并到那个分支。这个待会细说

###Feature branches

继承分支: develop
合并分支：develop
命名规范：除了master,develop,release-,hotfix-

Feature branches是用来开发新特性的(短期，远期都可以)。当开始开发新特性时，很可能不知道这个特性会出现在哪个目标版本。一旦开发完成就可以合并到develop，当然如果开发失败，就可以抛弃

####创建一个 Feature branch

当要创建一个新特性时，从develop分支上再进行分支

`git checkout -b myfeature develop`
Switched to a new branch "myfeature"

新特性完成时，可以合并到develop

`git checkout develop`
Switched to branch 'develop'
`git merge --no-ff myfeature`
Updating ea1b82a..05e9557
(Summary of changes)
`git branch -d myfeature`
Deleted branch myfeature (was 05e9557).
`git push origin develop`


—no-ff (译者注：no fast foward)标签，使得每一次的合并都创建一个新的commit记录。即使这个commit只是fast-foward，这样可以避免丢失信息

<img width=70% src="{{ site.url }}/imgs/dev/git/a-successful-git-branch/04.png"/>


不幸的是，我没有找到让—no-ff成为commit默认参数的方法(译者注：修改.consolerc?)，但确实应该提供一个方法

###Release branch

继承分支: develop
合并分支：develop 和 master
命名规范：release-*

Release branch 是为新的production release准备的(译者注：相当于RC版)，可以有一些小的bug，并为发布准备一些元数据(版本号，构建日期等等)。把所有的这些工作都放到 Release branch，develop branch就能更清晰地知道下一个版本要开发哪些特性

从develop分支合并到release分支的关键因素是:develop分支达到了release分支所要求的状态。至少所有针对该release的特性要被合并。至于那些将来会有的特性可以先放一放。然后就是为接下来即将要发布的版本分配一个版本号

####创建一个Release branch

Release branch是通过develop分支而创建。举个例子，假如1.1.5是当前的production release，然后会有一个比较大的版本发布。develop的状态已经可以发布版本了，经过商榷后，决定发布为1.2版本(而不是1.1.6或2.0)，所以我们创建一个release分支，并给这个分支一个新的版本号

`git checkout -b release-1.2 develop`
Switched to a new branch "release-1.2"
` ./bump-version.sh 1.2`
Files modified successfully, version bumped to 1.2.
`git commit -a -m "Bumped version number to 1.2"`
[release-1.2 74d9424] Bumped version number to 1.2
1 files changed, 1 insertions(+), 1 deletions(-)

这个新分支可能会存在一定的时间，直到可以被合并到production branch。这段时间内，bug修补可以在这个分支上进行(而不是develop分支)。添加新特性(尤其比较大的)是不允许的。最后还是要被合并到develop，然后继续在develop分支上开发，直到下一个版本

####完成一个release branch

当release branch已经准备就绪，需要做几件事。首先，release分支被合并到master分支上(每一个提交到master上的commit都是一个新版本，切记)。然后master上的commit都要添加tag，方便将来查看和回滚。最后release上所做的修改必须合并到develop分支上，保证bug已被修补

前两个步骤：
`git checkout master`
Switched to branch 'master'
`git merge --no-ff release-1.2`
Merge made by recursive.
(Summary of changes)
`git tag -a 1.2`

为了把release上的改变保存到develop，我们需要合并到develop
`git checkout develop`
Switched to branch 'develop'
`git merge --no-ff release-1.2`
Merge made by recursive.
(Summary of changes)

这个步骤可能会导致冲突，如果这样的话，解决冲突，然后再提交

现在一切都完成了，可以把release branch干掉了
`git branch -d release-1.2`
Deleted branch release-1.2 (was ff452fe).

###Hotfix branch
继承分支: master
合并分支：develop 和 master
命名规范：hotfix-*

Hotfix branch和Release branch有几分相似，都是为了新的production release而准备的。比如运行过程中发现了bug，就必须快速解决，这时就可以创建一个Hotfix branch，解决完后合并到master分支上。好处是开发人员可以继续工作，有专人来负责搞定这个bug
####创建Hotfix branch

Hotfix是从master分支上创建的。假如当前运行版本是1.2，然后发现有bug，但是develop还在开发中，不太稳定，这时就可以新开一个Hotfix branch，然后开始解决问题
`git checkout -b hotfix-1.2.1 master`
Switched to a new branch "hotfix-1.2.1"
`./bump-version.sh 1.2.1`
Files modified successfully, version bumped to 1.2.1.
`git commit -a -m "Bumped version number to 1.2.1"`
[hotfix-1.2.1 41e61bb] Bumped version number to 1.2.1
1 files changed, 1 insertions(+), 1 deletions(-)

解决问题，一次或几次commit
`git commit -m "Fixed severe production problem"`
[hotfix-1.2.1 abbe5d6] Fixed severe production problem
5 files changed, 32 insertions(+), 17 deletions(-)

####完成Hotfix branch

当结束时，bugfix要被合并到master，同时也要合并到develop，保证下个版本发布时该bug已被修复。这跟release branch完成时一样

首先更新master和tag release
`git checkout master`
Switched to branch 'master'
`git merge --no-ff hotfix-1.2.1`
Merge made by recursive.
(Summary of changes)
`git tag -a 1.2.1`

接下来与develop合并
`git checkout develop`
Switched to branch 'develop'
`git merge --no-ff hotfix-1.2.1`
Merge made by recursive.
(Summary of changes)

有一个例外，就是当一个release branch存在时，bugfix要被合并到release而不是develop，因为release最终会被合并到develop

最后移除branch
`git branch -d hotfix-1.2.1`
Deleted branch hotfix-1.2.1 (was abbe5d6).

###总结

这个开发模型其实没有什么新颖的。这是很优雅的一个模型，很容易实现，也容易在团队成员之间达成一致