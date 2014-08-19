---
layout: post
title: windows git 邮件通知
description: windows下git邮件配置
categories:
- git
tags:
- git
---
###准备工具
####开源mail server
[hmail](http://www.hmailserver.com/)

####命令行发邮件的工具
[blat](http://www.blat.net/examples)

初始化下blat,注意域名的修改
blat -install localhost admin@localmail.ddatsh.com

blat -body test -to admin@localmail.ddatsh.com -s "miss you" -u admin -pw admin
blat  -base64  -charset GB2312 -body 中文 -to admin@localmail.ddatsh.com -s "测试" -debug


####建空仓库
git init --bare

####设置接收人员
git config hooks.mailinglist "admin@localmail.ddatsh.com "

git config hooks.envelopesender "admin@localmail.ddatsh.com"

本质是

.git\config
<pre>
[hooks]
    mailinglist = admin@localmail.ddatsh.com
    envelopesender = admin@localmail.ddatsh.com
    emailprefix = "[GIT] "
    showrev = "git show -C %s; echo"
</pre>
####post-receive.sample
搜索post-receive.sample,打开后内容有
post-receive-email,在gitster里有
内容COPY过来，改名为post-receive
<pre class="prettyprint">
send_mail()
{
    if [ -n "$envelopesender" ]; then
        #sendemail -t admin@localmail.ddatsh.com -f admin@localmail.ddatsh.com -m
        blat -to admin@localmail.ddatsh.com -s "git"  -q
        #/usr/sbin/sendmail -t -f "$envelopesender"
</pre>

<pre class="prettyprint">
cmd /c "echo 1 >> init.txt"
cmd /c "git add ."
cmd /c "git commit -m 1"
cmd /c "git push origin master"
</pre>

<pre class="prettyprint">
To: admin@localmail.ddatsh.com
Subject: [GIT] test branch master updated. a63fc11d7c086f719c8d08d36df78c0cf7cbb6bf
X-Git-Refname: refs/heads/master
X-Git-Reftype: branch
X-Git-Oldrev: 45384097f58cfae3c65a1b43aaa295d11615fdf8
X-Git-Newrev: a63fc11d7c086f719c8d08d36df78c0cf7cbb6bf

This is an automated email from the git hooks/post-receive script. 
It was generated because a ref change was pushed to the repository containing the project
 "test".

The branch, master has been updated
       via  a63fc11d7c086f719c8d08d36df78c0cf7cbb6bf (commit)
      from  45384097f58cfae3c65a1b43aaa295d11615fdf8 (commit)

Those revisions listed above that are new to this repository have not appeared on 
any other notification email; so we list those revisions in full, below.

- Log -----------------------------------------------------------------
commit a63fc11d7c086f719c8d08d36df78c0cf7cbb6bf
Author: ddatsh <admin@localmail.ddatsh.com>
Date:   Thu Feb 9 14:17:29 2012 +0800

    1

diff --git a/init.txt b/init.txt
index 77fe233..eccec7d 100644
--- a/init.txt
+++ b/init.txt
@@ -11,3 +11,4 @@
 1
 1
 1
+1

-----------------------------------------------------------------------

Summary of changes:
 init.txt |    1 +
 1 files changed, 1 insertions(+), 0 deletions(-)


hooks/post-receive
-- 

test
</pre>
 
 