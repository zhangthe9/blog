---
layout: post
title: 基于Oracle 的sql优化
description:
- 基于Oracle 的sql优化
categories:
- Oracle
tags:
---
#RBO
create table emp_temp as select * from emp;

alter session set optimizer_mode='RULE';
set autotrace traceonly explain;

create index idx_mgr_temp on emp_temp(mgr);
create index idx_deptno_temp on emp_temp(deptno);

select * from emp_temp where mgr>100 and deptno>100;

INDEX RANGE SCAN        IDX_DEPTNO_TEMP
###字典顺序
根据对象在字典创建顺序,后创建的优先，选择到 IDX_DEPTNO_TEMP
###等价改写目标SQL，让RBO生效
number,date加 0，char加 || 
select * from emp_temp where mgr>100 and deptno+0>100;
让deptno索引失效，就走到了mgr索引

drop index idx_mgr_temp 后，再创建，再执行就走了变成后创建的mgr索引

rbo优化器硬编码在代码里，15个级别，1为rowid，15为 full table scan

#CBO
###查询改写
create table t1 (c1 number,c2 varchar2(10));
create table t2 (c1 number,c2 varchar2(10));

create index idx_t2 on t2(c1);

insert into t1 values (10,'test');
insert into t2 values (10,'test');

select t1.c1,t2.c2 from t1,t2
where t1.c1=t2.c1 and t1.c1=10;

没有用到t2.c1 索引列，可结果中
 INDEX RANGE SCAN                  IDX_T2

所以谓词被加了 and t2.c1=10

#优化器基础知识
##优化器模式
####RULE
####CHOOSE
####FIRST_ROWS_n
####FIRST_ROWS
####ALL_ROWS

##数据访问方法
###表访问方法
####全表扫描
####rowid扫描

##索引访问方法
####索引唯一扫描
####索引范围扫描
####索引全扫描
####索引快速全扫描
####索引跳跃式扫描

create table emp_temp as select * from emp;
select count(empno) from emp_temp;
create unique index idx_emp_temp on emp_temp(empno);

 exec dbms_stats.gather_table_stats(ownname => 'C##SCOTT',tabname => 'EMP_TEMP',estimate_percent => 100,cascade => true,method_opt => 'for all columns size 1');

alter system flush shared_pool;
alter system flush buffer_cache;
set autotrace traceonly;
会提示权限之类问题
utlxplan.sql
grant all on plan_table to c##scott;
@Catalog.sql 很长时间
plustrce.sql
grant plustrace to c##scott;
grant select any dictionary to c##scott;


select * from emp_temp where empno=7369;

观察consistent gets
drop index idx_emp_temp ;
create index idx_emp_temp on emp_temp(empno);

alter system flush shared_pool;
alter system flush buffer_cache;
再次
select * from emp_temp where empno=7369;
会发现consistent gets多一次


##表连接
####表连接顺序
实际不管有多少表做连接，执行时只能两两连接
要先决定驱动与被驱动表，连接出结果后再和剩余表连接
####表连接方法
排序合并连接、嵌套循环连接、HASH连接、笛卡尔连接
####访问单表方法
走全表扫描还是索引，什么索引访问方法