---
layout: post
title: 版本管理的发展史
description:
- 版本管理器的发展史
categories:
- git
tags:
- git
---
http://coolshell.cn/articles/3288.html

![git-scm]({{ site.url }}/imgs/dev/git/scmhistory.png)

分成四个时期：

史前时期：1982年的RCS

古典时期：1990年的CVS（经典的SCM管理器，不能track目录和文件名的改变），1985年的PVCS，1992年的clearcase（价格贵，功能复杂，今天也有很多公司在用），微软的VSS，90年代中期的Perforce(P4，这个工具今天都还在被广泛地使用，尤其是那些中等大小却有着大量开发团队的公司，现在是Google内部最大的代码管理器)

中世纪时期：SVN（Linus也很不喜欢SVN，2006年引入了Git），AccuRev(强力支持branch和merge，其扮演了一个很重要角色－帮助社区脱离clearcase和CVS)

文艺复兴时期：BitKeeper——Sun的内部管理工具，Linux的内核代码2002年用它

Linus改变了历史——中世纪已经过去了，现在的世界由分布式系统主宰，以及消除branch和merge的恐惧

Git 基于 DAG 结构 (Directed Acyclic Graph)，其运行起来相当的快。在Git发布后的来年，几乎所有的大型的开源项目全部从Subversion迁移到了Git上，www.github.com真是很大，这可能是这具星球上最强大最牛最酷的SCM系统了。Git可能并不是最简单的，但它一定会是未来十年的主流。（有空读读这本书——Git Internals）

Mercurial (Hg) 第一次出现在2005年4月，也是因为BitKeeper不免费了。Hg可以和Git在一起使用，见：http://mercurial.selenic.com/wiki/HgGit。但是Hg和Git在设计上不一样，他们对提交/变更的概念是一样的，只不过Git用tree来实现，而Hg则是用扁平的文件和目录来实现（revlog），设计细节可参看：http://mercurial.selenic.com/wiki/Design和 http://mercurial.selenic.com/wiki/DeveloperInfo

Darcs (Darcs Advanced Revision Control System)是另一个让你摆脱Subversion和CVS的工具
它的优势是性能，以及他与众不同的历史版本管理——管理patches而不是snapshot（提交/修改），当然，这样一来，历史改变看上去很不好懂

Bazaar (bzr) 是另一个开源的 DVCS，它试图给SCM的世界里带来一些新的东西。其由Canonical(Ubuntu那公司)，2008年成为GNU

Plastic在2006年出现，强力地支持branch和merge，其还提供了强大的图示，包括3D的版本树，Plastic主要是为了让中等开发团队使用，介于大型的团队（ClearCase）和小型的团队（Subversion）之间

Team Foundation Server (TFS)，微软的新一代SCM工具，主要是为了VSS的失败负责，但是他还不是版本管理上还是很强，只不过，他集成了一大堆各种各样的工具，比如：issue tracking，test management等



###评论
Mensch88 2010年11月17日17:37

把Linus造成神并进行崇拜对我们的学习并没有帮助。
Git的第一个版本是Linux之父Linus Torvalds亲手操刀设计和实现的，但Linus不是凭空设计出来的，他是站在巨人的肩膀上，很大程度上借鉴了BitKeeper和Monotone的设计。Git的开发从2005年4月3日开始，2005年4月7日完成大约1300行代码，实现基本功能并selfhosting
http://marc.info/?l=git&m=117254154130732

“无论是反对还是喜欢，Linus的确是改变了历史——中世纪已经过去了，现在的世界由分布式系统主宰，以及消除branch和merge的恐惧。” 没有Linus，历史依然向前进，分布式系统依然会占上风。Bazaar在2005年3月立项，Mecurial在2005年4月19日也已经完成第一个版本

“Git 基于 DAG 结构 (Directed Acyclic Graph)，其运行起来相当的快。”
Bazaar和Mecurial也都基于DAG结构。另外DAG有中文名称，叫有向无环图

“在Git发布后的来年，世界上所有的大型的开源项目全部从Subversion迁移到了Git上，www.github.com真是很大，这可能是这具星球上最强大最牛最酷的SCM系统了。Git可能并不是最简单的，但它一定会是未来十年的主流。”
兄弟，你在说些什么？世界上所有的大型的开源项目全部从Subversion迁移到了Git上？www.github.com真是很大？最强大最牛最酷的SCM系统？
我承认由于Linus个人因素的影响，Linux kernel和不少Linux下的开源项目都使用了Git，但请不要忽视别的DVCS的存在。 Ubuntu在哪里？Octave在哪里？Emacs在哪里？Vim在哪里？
另外请注意：www.github.com只是ruby搭起来的一个使用Git版本控制的商业网站，它属于GitHub公司，与Git和Linus没有内在的联系。而我还真没看见过有多少大型开源项目把代码托管到了github上

