---
layout: post
title: Java Singleton 
description:
- Java 单例
categories:
- java
tags:
- 模式
---
通常会使用的这样的写法来实现单例：
<pre class="prettyprint">
public class Singleton {
	private Singleton() {
	}

	private static Singleton instance = null;

	public static Singleton getInstance() {
		if (Singleton.instance == null) {
			Singleton.instance = new Singleton();
		}
		return Singleton.instance;
	}
}
</pre>

单例为了保证运行时Singleton类只有唯一的一个实例，最常用的地方比如拿到数据库的连接，Spring的中创建BeanFactory这些开销比较大的操作，而这些操作都是调用他们的方法来执行某个特定的动作

####单例会带来什么问题？

多个线程同时调用这个实例，会有线程安全问题

####怎么实现线程安全的单例模式?
1、如果不执行修改对象的操作的情况下，单单执行一个读取操作，还有没有进行同步的必要？
2、保证单例的线程安全使用synchronized会产生什么样的问题？
3、不使用synchronized，有什么方式来保证线程安全？

使用哪种单例实现取决于不同的环境
上面的懒汉式，适合于单线程程序，多线程情况下需要保护getInstance()方法，否则可能会产生多个Singleton对象的实例
在此基础上确保getInstance()方法一次只能被一个线程调，getInstance()前加 synchronized ，锁定整个方法
<pre class="prettyprint">
 public class Singleton {
	private static Singleton instance = null;

	private Singleton() {
	}

	public static synchronized Singleton getInstance() {
		if (Singleton.instance == null) {
			Singleton.instance = new Singleton();
		}
		return Singleton.instance;
	}
}
</pre>
	
通常认为锁定整个方法比较耗费资源，此代码实际会产生多线程访问问题的只有 instance = new Singleton(); 
为降低 synchronized 块性能方面影响，只锁定instance = new Singleton(); 这一句
<pre class="prettyprint">
public class Singleton {
	private static Singleton instance = null;

	private Singleton() {
	}

	public static Singleton getInstance() {
		if (Singleton.instance == null) {
			synchronized (Singleton.class) {
				Singleton.instance = new Singleton();
			}
		}
		return Singleton.instance;
	}
}
</pre>
	
分析这种实现方式，两个线程可以并发地进入第一次判断instance是否为空的if 语句内部，第一个线程执行new操作，第二个线程阻断，当第一个线程执行完毕之后，第二个线程没有进行判断就直接进行new操作，所以这样做也并不是安全的
为了避免第二次进入synchronized块没有进行非空判断的情况发生，添加第二次条件判断
<pre class="prettyprint">
public static Singleton getInstance(){      
    if(instance == null){      
        synchronize{      
           if(instance == null){      
              instance =  new Singleton();       
           }      
        }      
    }      
    return instance;   
}  
</pre>
这样就产生了二次检查，但是二次检查自身会存在比较隐蔽的问题
“双重检查锁定背后的理论是完美的。不幸地是，现实完全不同。双重检查锁定的问题是：并不能保证它会在单处理器或多处理器计算机上顺利运行。双重检查锁定失败的问题并不归咎于 JVM 中的实现 bug，而是归咎于 Java 平台内存模型。内存模型允许所谓的“无序写入”，这也是这些习语失败的一个主要原因。”

 
使用二次检查的方法也不是完全安全的，原因是 java 平台内存模型中允许所谓的“无序写入”会导致二次检查失败，所以使用二次检查的想法也行不通了

Peter Haggar在最后提出这样的观点：“无论以何种形式，都不应使用双重检查锁定，因为您不能保证它在任何 JVM 实现上都能顺利运行。”

 
懒汉式的lazy方式实现单例弯弯绕太多，单线程编程的情况下懒汉式单例实现没有任何问题
多线程的情况下，需要比较小心，对getInstances()方法加上synchronized关键字，这样虽然可能有一些性能上的牺牲，但是更加的安全。绕了这么大的一个弯，又回来了：
<pre class="prettyprint">
/* 安全的方式 1 */
public class Singleton {
	private static Singleton instance = null;

	private Singleton() {
	}

	public static synchronized Singleton getInstance() {
		if (Singleton.instance == null) {
			Singleton.instance = new Singleton();
		}
		return Singleton.instance;
	}
}
</pre>
另外一种实现方式是这样的，放弃使用 synchronized 关键字，而使用 static 关键字：
<pre class="prettyprint">
/* 安全的方式 2 */
public class Singleton {
	private static Singleton instance = new Singleton();

	private Singleton() {
	}

	public static Singleton getInstance() {
		return Singleton.instance;
	}
}
</pre>
这种方式没有使用同步，并且确保了调用static getInstance()方法时才创建Singleton的引用（static 的成员变量在一个类中只有一份）


更加灵巧，没有使用同步但保证了只有一个实例，还同时具有了Lazy的特性
<pre class="prettyprint">
/* 安全的方式 3 */
public class ResourceFactory {
	private static class ResourceHolder {
		public static Resource resource = new Resource();
	}

	public static Resource getResource() {
		return ResourceFactory.ResourceHolder.resource;
	}

	static class Resource {
	}
}
</pre>
上面的方式是值得借鉴的，在ResourceFactory中加入了一个私有静态内部类ResourceHolder ，对外提供的接口是 getResource()方法，也就是只有在ResourceFactory .getResource()的时候，Resource对象才会被创建，
这种写法的巧妙之处在于ResourceFactory 在使用的时候ResourceHolder 会被初始化，但是ResourceHolder 里面的resource并没有被创建，这里隐含了一个是static关键字的用法，使用static关键字修饰的变量只有在第一次使用的时候才会被初始化，而且一个类里面static的成员变量只会有一份，这样就保证了无论多少个线程同时访问，所拿到的Resource对象都是同一个。

饿汉式的实现方式虽然貌似开销比较大，但是不会出现线程安全的问题，也是解决线程安全的单例实现的有效方式


至于ThreadLocal，还是应该由使用场景来决定


在《Java与模式》中，作者提出：“饿汉式单例类可以在Java语言实现，但不易在C++内实现，因为静态初始化在C++里没有固定的顺序，因而静态的instance变量的初始化与类的加载顺序没有保证，可能会出问题。这就是为什么GoF在提出单例类的概念时，举的例子是懒汉式的。他们的书影响之大，以致Java语言中单例类的例子也大多是懒汉式的。实际上，**饿汉式单例类更符合Java语言本身的特点。**”

由此可见在应用设计模式的同时，分析具体的使用场景来选择合适的实现方式是非常必要的