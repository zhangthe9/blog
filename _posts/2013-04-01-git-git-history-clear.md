---
layout: post
title: git只保留最新代码,删除历史记录
description:
- git只保留最新代码,删除历史记录
categories:
- git
tags:
- git
---
	git cat-file commit master^X | sed -e '/^parent/ d' > tmpfile
	git rebase --onto $(git hash-object -t commit -w tmpfile) master
	rm -f tmpfile


其中X是要保留的记录条数
这个时候,你的log里已经没有历史的提交了,但是历史的数据还存在于本地,|
要想完全删除的话,执行以下代码

    rm -rf .git/logs
    git gc 

注意,这里只对master进行了操作,如果你还有其它branch或tag,都需要类似于这样地处理一遍.

实战的时候，貌似有点不对的地方 填了数值后，是回退到master ^n 的地方了