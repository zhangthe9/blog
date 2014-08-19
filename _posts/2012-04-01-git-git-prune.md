---
layout: post
title: git永久删除文件(包括历史记录)
description: 命令备忘
categories:
- git
tags:
- git
---

注意Windows下用**双引号**
<pre class="prettyprint">
git filter-branch --index-filter 'git rm -r --cached --ignore-unmatch path/to/your/file' HEAD
git push origin master --force
rm -rf .git/refs/original/
git reflog expire --expire=now --all
git gc --prune=now
git gc --aggressive --prune=now
</pre>
参考

GitHub的帮助页面[Removing sensitive data](http://help.github.com/remove-sensitive-data)

http://yihui.name/cn/2010/12/animation-update-1-1-5

http://blog.csdn.net/meteor1113/article/details/4407209

两者区别是 --index-filter  比--tree-filter 效率更高

 