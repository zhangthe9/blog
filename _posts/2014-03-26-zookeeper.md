---
layout: post
title: zookeeper
description: zookeeper
---
http://mirror.bit.edu.cn/apache/zookeeper/current/zookeeper-3.4.6.tar.gz

分布式服务框架 Zookeeper -- 管理分布式环境中的数据
http://www.ibm.com/developerworks/cn/opensource/os-cn-zookeeper/


Apache Hadoop 子项目，解决分布式应用数据管理问题，如：统一命名服务、状态同步服务、集群管理、分布式应用配置项的管理等
维护和监控数据变化，解决分布式集群中一致性问题

###典型的应用场景
配置文件、集群、同步锁、Leader 选举、队列管理

###配置
zoo_sample.cfg >zoo.cfg
tickTime=5000
dataDir=e:/tmp/zookeeper/data
clientPort=2181
initLimit=5
syncLimit=2

bin\zkServer.cmd

####tickTime
zk server之间或client与zk server间心跳时间
####clientPort
client 连zk server端口
####initLimit
zk接受Follower 服务器初始化连接时最长能忍受多少个心跳时间间隔数
超过10个心跳(tickTime)后 zk server还没有收到Follower返回信息，认为Follower连接失败
####syncLimit
Leader 与 Follower 请求和应答时间长度，最长不能超过多少个 tickTime 的时间长度


###角色
leader,follower,observer
###状态
leading,following,observing,looking

###znode类型
Persistent Nodes: 永久有效地节点,除非client显式的删除,否则一直存在
Ephemeral Nodes: 临时节点,仅在创建该节点client保持连接期间有效,一旦连接丢失,zookeeper会自动删除该节点
Sequence Nodes: 顺序节点,client申请创建该节点时,zk会自动在节点路径末尾添加递增序号,这种类型是实现分布式锁,分布式queue等特殊功能的关键

###ZAB: ZooKeeper的Atomic Broadcast协议
http://breezes.lofter.com/post/d325_1aa98b
Paxos一致性不能达到ZooKeeper要求
leader P1发起了两个事务<t1, v1>（t1为序号,要写的值是v1）和<t2, v2>，然后挂掉
新来个P2 leader发起<t1, v1'>
而后又来新P3 leader 汇总最终的执行序列<t1, v1'>和<t2, v2>，即P2的t1在前，P1的t2在后

树形结构的ZooKeeper，很多操作都要先检查才能确定能不能执行
P1事务t1可能创建节点/a,t2创建节点/a/aa，只有先创建了父节点/a，才能创建子节点/a/aa
而P2所发起的事务t1可能变成了创建/b
这样P3汇总后序列是先创建/b再创建/a/aa，由于/a还没建，创建a/aa就出错
ZAB保证同一个leader的发起的事务要按顺序被apply和只有先前的leader的所有事务都被apply之后，新选的leader才能在发起事务

ZAB核心思想:任意时刻只有一个leader，所有更新事务由leader发起去更新所有复本（follower），更新时用两阶段提交协议，多数节点prepare成功，就通知他们commit
各follower要按当初leader让他们prepare的顺序来apply事务
ZAB事务永远不会回滚，ZAB的2PC做了点优化，多个事务只要通知zxid最大的那个commit，之前的各follower会统统commit

如果没有节点失效，那ZAB上面这样搞下就完了，麻烦在于leader失效或leader得不到多数节点的支持时怎么处理

关键点：
1、leader所follower之间通过心跳来检测异常
2、检测到异常之后的节点若试图成为新的leader，首先要获得大多数节点的支持，然后从状态最新的节点同步事务，完成后才可正式成为leader发起事务
3、区分新老leader的关键是一个会一直增长的epoch


除了能保证顺序外，ZAB性能也能不错，千兆网络5节点部署的TPS达到25000左右，响应时间只有约6ms


###读写性能测试
http://www.ostest.cn/archives/469?utm_source=weibolife