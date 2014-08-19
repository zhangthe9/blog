---
layout: post
title: zookeeper
description: zookeeper
categories:
- zookeeper
tags:
- zookeeper
---
Yahoo借鉴Google Chubby设计思想开发Zookeeper并开源交到Apache,成为Hadoop子项目
协调系统，作用的对象是分布式系统,解决分布式系统“部分失败”

异步读同步写
对外提供低级的原语，可实现复杂的分布式算法，如queue、barrier和lock等
做可靠容错，进行订阅分发服务，或者其他应用

paxos的变种-原子多播协议Zab


##Paxos
基于消息传递的分布式一致性算法，用于分布式计算
前提：没有拜占庭将军问题。即只有在一个可信的计算环境中才能成立，这个环境是不会被入侵所破坏的


Paxos小岛(Island)
议员(Senator)
确定，不能更改的 议员的总数(Senator Count)
提议(Proposal)
提议编号(PID)

提议超过半数((Senator Count)/2 +1)议员同意生效
议员只同意大于当前编号提议


ZK Server里Paxos
小岛(Island)——ZK Server Cluster
议员(Senator)——ZK Server
提议(Proposal)——ZNode Change(Create/Delete/SetData…)
提议编号(PID)——Zxid(ZooKeeper Transaction Id)
正式法令——所有ZNode及其数据

Paxos岛上的议员人人平等，ZK Server有Leader概念
如果议员人人平等，在某种情况下会由于提议的冲突而产生一个“活锁”（所谓活锁我的理解是大家都没有死，都在动，但是一直解决不了冲突问题）
议员中设立一个总统，有权发出提议，议员提议必须发给总统来提出
总统——ZK Server Leader


总统怎么选出来的？
http://rdc.taobao.com/blog/cs/?p=162

ZK Server属于自己特性的东西：Session, Watcher，Version等


在zookeeper的集群中，各个节点共有下面3种角色和4种状态：
角色：leader,follower,observer
状态：leading,following,observing,looking

##ZNode
Persistent Nodes: 永久有效地节点,除非client显式的删除,否则一直存在
Ephemeral Nodes: 临时节点,仅在创建该节点client保持连接期间有效,一旦连接丢失,zookeeper会自动删除该节点
Sequence Nodes: 顺序节点,client申请创建该节点时,zk会自动在节点路径末尾添加递增序号,这种类型是实现分布式锁,分布式queue等特殊功能的关键

###Watch
分布式回调
可以watch的event:
KeeperState:Disconnected,SyncConnected,Expired
EventType:None,NodeCreated,NodeDeleted,NodeDataChanged,NodeChildrenChanged



##分布式系统几种典型一致性算法简述
