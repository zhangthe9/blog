---
layout: post
title: 分布式Java应用基础与实践
description: spring
---
####强引用
A a=new A(); 对象主动释放引用后才被GC
####软引用
JVM内存不足、或认为不经常活动时被回收，适用于缓存
 -XX:SoftRefLRUPolicyMSPerMB=1000


HotSpot JVM garbage collection options cheat sheet
http://blog.ragozin.info/2011/09/hotspot-jvm-garbage-collection-options.html