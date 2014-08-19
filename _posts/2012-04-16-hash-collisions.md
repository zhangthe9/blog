---
layout: post
title: hash 冲突
categories:
- 无
description: hash冲突
---
这种攻击方式最早在03年的时候就被提出来了，但是当初各种攻击的方式太多了，像这种不能拿服务器权限的ddos攻击方式中的一种没有引起大家太多的关注，当时出的pdf文档如下：

http://www.cs.rice.edu/~scrosby/hash/CrosbyWallach_UsenixSec2003.pdf

最近有人又提出了这种攻击方式，并且统计了一下目前大多数的语言都受到了影响

目前已知的受影响的语言以及版本有::

Java, 所有版本、JRuby <= 1.6.5、PHP <= 5.3.8, <= 5.4.0RC3、Python, 所有版本、Rubinius, 所有版本、Ruby <= 1.8.7-p356、Apache Geronimo, 所有版本、Apache Tomcat <= 5.5.34, <= 6.0.34, <= 7.0.22、Oracle Glassfish <= 3.1.1、Jetty, 所有版本、Plone, 所有版本、Rack, 所有版本、V8 JavaScript Engine, 所有版本

不受此影响的语言或者修复版本的语言有::

PHP >= 5.3.9, >= 5.4.0RC4、JRuby >= 1.6.5.1、Ruby >= 1.8.7-p357, 1.9.x、Apache Tomcat >= 5.5.35, >= 6.0.35, >= 7.0.23、Oracle Glassfish

攻击主要原理（以java语言为例）：

1. HashMap的工作原理

当客户端提交一个请求并附带参数的时候，web应用服务器会把我们的参数转化成一个HashMap存储，这个HashMap的逻辑结构如下：

key1-->value1;

key2-->value2;

key3-->value3;

…………

但是物理存储结构是不同的，key值会被转化成Hashcode，这个hashcode有会被转成数组的下标：

0-->value1;

1-->value2;

2-->value3;

…………

hashcode是一个32位的整数，2^32次方大约在40多亿，就是说总数是一定的，但是key值有无限种可能，所以不同的string就会产生相同hashcode而导致碰撞，碰撞后的物理存储结构可能如下：

0-->value1-->value2;

1-->空;

2-->value3;

…………

2. Hash碰撞对性能的影响

当产生一个碰撞的hashcode，需要插入到这个HashMap的时候他回去对应的下标序列顺序查找所有的项，看是否已经存在，所以产生一个新的hashcode碰撞插入到一个已经存在n个相同hashcode的数组下标项中的时候消耗的性能大概为O(n)，但是如果是n个相同hashcode插入到已经n个hashcode的数组下标中时，需要比较消耗的性能就是O(n*n)了

3. 怎么生成相同的hashcode

java中生成hashcode的方法为String.hashCode();

public int hashCode()   { 

int h=hash; 

if(h==0){ 

int off = offset; 

char val[]=value; 

int len = count; 

for(int i=0;i<len;i++)   { 

h=31*h+val[off++]; 

}

hash=h;

}

换成推导公式就是 s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1] （注：使用 int 算法，这里 s[i] 是字符串的第 i 个字符，n 是字符串的长度，^ 表示求幂）这里的自定义乘数为什么要用“31”，这个就是数学上的一些经验吧，具体我也不是很清楚，但是选31这个质数能保证hash碰撞的几率最小。
从这个算法中我们可以看到是没有随机化，也就是说我们可以自己构造具有相同hashcode的string，甚至可以根据想要的hashcode值来反推出大量我们想要的string字符串，具体的代码就不放出来了，但是我们可以根据这个公式得到一些比较简单的具有相同hashcode的string，比如：
Aa和BB，两次for循环，Aa的第一次for循环得到h的值为A的asc值65，第二次就是31*65+(a的asc值97)也就是2112
同样可以算出BB的hashcode也是2112，这里就有一个碰撞了

根据这两个，我们可以知道如下4个也具有相同的hashcode：
 "AAAA", "AABb", "BbAA", "BbBb"
进一步，我们可以知道如下字符串在java里面具有相同的hashcode：

"AaAaAaAa","AaAaBBBB","AaAaAaBB","AaAaBBAa",
"BBBBAaAa","BBBBBBBB","BBBBAaBB","BBBBBBAa",
"AaBBAaAa","AaBBBBBB","AaBBAaBB","AaBBBBAa",
"BBAaAaAa","BBAaBBBB","BBAaAaBB","BBAaBBAa",

其实Bb和CC、Cc和DD都具有一样的hashcode，他们的组合BbCCCcDD和CCCCCcDD也有一样的hashcode

复杂一点的：

8SImjKsEaWZX[Uqp_

a5alI\II=QZglaY`o

pfx>yDOoSL[^mhVck

dEY<A`@FBw[k[c_i`

<>^T;B=4Ss[osjZV]

>KCD?7f[>>Uiqrra_

…………

都具有相同的hashcode：123456

4. 怎么攻击

1. 构造几千个具有相同hashcode的参数名

2. 构造POST表单进行提交（当然GET也可以，就是限制比POST大）像"AaAa=&AaBB=……"

3. 构造多个线程重复提交，单机就可以打死小站

5. 怎么防御

1. 限制post和get的参数个数，越少越好

2. 限制post数据包的大小

3. WAF