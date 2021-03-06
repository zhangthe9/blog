---
layout: post
title: c++ 流派
description:
- 主要C++流派，看看你是哪一流 
categories:
- C
---
- 经典C++流  
类是核心，例程多用C Runtime的，很少用模版，一般是正统教育的结果
- 古典C流  
基本上当C用，偶尔用用对象，不使用异常，喜欢怀旧
- MFC流  
秉承MFC的风格，主要使用MFC/ATL对象和Win32 API，不喜欢STL，用很多的宏把IDE的语法提示模块折磨到崩溃
- Portable流  
以C Runtime和STL为主要工具，使用类和模版，不跨平台毋宁死
- Functional流  
以模版和STL为主要武器，大量使用函数式语言的设计方法，并号称这才是真正的C++
- Win32流：多使用全局函数，偏爱Win32 API，但不排斥C Runtime，通常喜欢轻量级的程序，所以身材也比较苗条
- Java流  
全面使用Java的风格，不能容许任何全局成员，但允许使用STL的集合类，写很多叫Factory的类
- COM流  
喜欢AddRef()和Release()，大量使用接口，隐藏一切可以隐藏的东西，诵经的时候要把上帝替换成COM
- 戒律流  
追求完美的C++程序，计较每一个const和throw()，极力避免不安全的cast，随身一定要带一本ISO C++手册
- 混沌流  
其程序无常形，无恒道，变幻莫测，吾不知其名

<hr/>
- C流  
精通C，习惯了她的思想
- C/C++流  
精通C，对C++一般，但是为了项目或者其他需求，不得不实现混用。而且不爱写注释
- MFC/C++流  
使用类，几乎不用模版。大量派生！大量异常！
- STL/C++流  
大量使用范型，创建可移植！！能用crt的全部使用crt
- DP/C++流  
热爱设计模式，大量类，但都讲求自认为的精细，间或使用范型。大量注释，恨不得比代码还多，XP方法下的产物