---
layout: post
title: java-transactions-book
description:
- java事务设计策略
- <br/>
- 事务缘起,策略,陷阱,实践
categories:
- java
tags:
- 事务
---
###title
系统中数据问题通常由错误或没有使用事务管理策略造成
多数java应用不具备合理事务设计策略,或仅依赖数据库管理复杂事务
Spring使用的声明式事务,只是管理事务的基础,本身不是事务设计策略

####主题为利用EJB,Spring等编程框架提供的事务模型设计高效事务管理策略

###java事务模型
- 本地
- 编程式
- 声明式
####本地事务模型
不是编程框架本身管理事务，事务交给本地资源管理器管理
资源管理器是用于通信的、事实上的数据源提供者

对于数据库，资源管理器通过数据库驱动和数据库管理系统（DBMS）来实现的
对于JMS，所谓资源管理器就是通过特定的JMS 提供者（provider）实现的队列（queue）或主题（topic）的连接工厂
经由本地事务模型，**开发人员管理的是“连接”，而非“事务”**
DBMS和JMS 的提供者真正管理了本地事务


####编程式事务模型
用Java 事务API（JTA）及其底层事务服务实现
**开发人员编码对象是“事务”，而非“连接”**
使用javax.transaction.UserTransation 接口
**begin(),commit(),rollback()**
通常不鼓励大量使用编程式事务，客户端发起，远程无状态会话Bean的EJB 事务访问场景下，还是有可能用到的


####声明式事务模型
也称为**容器托管的事务模型（CMT）**
**软件框架或“容器”管理了事务的开始和结束**
仅需要告诉框架碰到异常回滚
**事务配置放在XML或Annotation**

###本地事务模型
事务被底层数据库（DBMS）或在JMS 中被底层消息服务提供者所管理
开发人员的角度,管理的并非“事务”，而是“连接”

<pre class="prettyprint">
public void updateTradeOrder(TradeOrderData order) throws Exception {
	DataSource ds = (DataSource)(new InitialContext()).lookup("jdbc/MasterDS");
	Connection conn = ds.getConnection();
	conn.setAutoCommit(false);
	Statement stmt = conn.createStatement();
	String sql = "update trade_order ... ";
	try {
		stmt.executeUpdate(sql);
		conn.commit();
	} catch (Exception e) {
		conn.rollback();
		throw e;
	} finally {
		stmt.close();
		conn.close();
	}
}
</pre>
	
Connection.setAutoCommit(false)、Connection.commit()、Connection.rollback()
setAutoCommit()**最重要**
	
<pre class="prettyprint">
Statement stmt = conn.createStatement();
String sqlOrder = "update trade_order ... ";
String sqlTrade = "update trade_fee ... ";
try {
	stmt.executeUpdate(sqlOrder);
	stmt.executeUpdate(sqlTrade);
} catch (Exception e) {
	throw e;
} finally {
	stmt.close();
	conn.close();
}
</pre>
这里没有setAutoCommit()
如果第一个executeUpdate执行后,第二个失败,整个方法抛异常,**但第一句SQL事实不会改变,违反ACID中的原子性**
其次，这两个语句造成的更新操作，作为一个逻辑工作单元（LUW，Logic Unit of Work），
并未与操作同一个table或同一些行（row）的其他处理过程相隔绝，因此违反了ACID中的独立性原则
	
####本地事务模型的缺陷
将两个更新SQL 分隔在DAO方法中：
<pre class="prettyprint">
public void updateTradeOrder(TradeOrderData order) throws Exception {
	OrderDAO orderDao = new OrderDAO();
	TradeDAO tradeDao = new TradeDAO();
	try {
		//在DAO 类中的SQL 和Connection 逻辑
		orderDao.update(order);
		tradeDao.update(order);
	} catch (Exception e) {
		logger.fatal(e);
		throw e;
	}
}
</pre>
不论在各自的DAO 方法中怎样配置自动提交和连接管理，每个更新都是分别处理的
意味着在每个DAO 的update()方法最后数据库更新都将被提交

####用连接传递尝试解决
在更上一层的方法中建立数据库连接，而后将连接对象当作参数传入DAO 的update()方法
使代码符合ACID 的准则
<pre class="prettyprint">
public void updateTradeOrder(TradeOrderData order) throws Exception {
	DataSource ds = (DataSource)(new InitialContext()).lookup("jdbc/MasterDS");
	Connection conn = ds.getConnection();
	conn.setAutoCommit(false);
	OrderDAO orderDao = new OrderDAO();
	TradeDAO tradeDao = new TradeDAO();
	try {
		//在DAO 类中的SQL 和连接逻辑
		orderDao.update(order, conn);
		tradeDao.update(order, conn);
		conn.commit();
	} catch (Exception e) {
		logger.fatal(e);
		conn.rollback();
		throw e;
	} finally {
		conn.close();
	}
}
</pre>
虽然此技术在大多数情况下是可行的，但它不是是一种高效的事务设计策略
使用连接传递极易造成错误，并且需要很大的编程工作量编码维护
如果发现自己不得不将代码改成上面那样，那是到了放弃本地事务模型，采用编程式模型或声明式模型的时候了
	
####本地事务的费神之处与限制
就连接逻辑造成错误代码的几率是很大的
必须非常关注自动提交标志的设置，特别在同一方法中进行多个更新操作时更是如此
并且，如果目标方法涉及到管理连接，开发人员在调用前必须非常仔细地审查这些方法

除非应用仅仅是简单地涉及到单表更新（single-table updates），没有一种有效的手段可以保证对单个请求总是产生单一的事务工作单元
	
另一个问题是，本地事务不能在使用XA 全局事务协调多个资源时并发地存在。当需要协调多个资源（如数据库和JMS 目标，如队列或主题）时，不可能在保证ACID 特性的前提下使用本地事务模型
**本地事务模型最好只能在单表更新的，最简单的基于WEB 的Java 应用中使用**


###编程式事务模型
和本地事务模型两者最大区别之一是，开发人员使用编程式模型，管理的是事务（transaction），而不是连接（connection）
<pre class="prettyprint">
public void updateTradeOrder(TradeOrderData order) throws Exception {
	UserTransaction txn = sessionCtx.getUserTransaction();
	txn.begin();
	try {
		TradeOrderDAO dao = new TradeOrderDAO();
		dao.updateTradeOrder(order);
		txn.commit();
	} catch (Exception e) {
		log.fatal(e);
		txn.rollback();
		throw e;
	}
}
</pre>
	
	
事务上下文（transaction context）被传递到TradeOrderDAO 对象中，因此不像本地事务模型那样，TradeOrderDAO 并不需要管理连接和事务
它所需要做的仅仅是从连接池中获得一个连接，用完后还回去
在编程式事务模型中，开发人员负责开启和终止事务,通过UserTransaction 接口完成

Spring 通过使用org.springframework.transaction 包下的TransactionTemplate 或PlatformTransactionManager 完成

选择使用编程式模型还是声明式模型是在每个bean的粒度设置的,可以混合使用两种模型,但最好不要

可以使用元数据标注（metadata annotations）的方式指明何处使用编程式事务管理
<pre class="prettyprint">
@Stateless
@TransactionManagement(TransactionManagementType.BEAN)
public class TradingServiceBean implements TradingService
{
...
}
</pre>
	
Spring 中可以选择使用TransactionTemplate 或PlatformTransactionManager
<pre class="prettyprint">
public void updateTradeOrder(TradeOrderData order)
throws Exception {
	transactionTemplate.execute(new TransactionCallback()	{
		public Object doInTransaction(TransactionStatus status)	{
			try {
				TradeOrderDAO dao = new TradeOrderDAO();
				dao.updateTradeOrder(order);
				} catch (Exception e) {
				status.setRollbackOnly();
				throw e;
			}
		}
	} );
}
</pre>
	
<pre class="prettyprint">
<xmp><bean id="transactionTemplate"
	class="org.springframework.transaction.support.TransactionTemplate">
	<property name="transactionManager">
		<ref local="transactionManager"/>
	</property>
</bean>
<bean id="tradingService" class="com.trading.server.TradingService">
	<property name="transactionTemplate">
		<ref local="transactionTemplate"/>
	</property>
</bean>
</xmp></pre>

Spring 使用事务回调（transaction callback）将包含在业务方法中的逻辑在事务上下文中包裹起来
注意，当使用此技术的时候，并非要像在EJB 中那样一定需要显示的调用begin()和commit()
并且，这里通过TransactionStatus.setRollbackOnly()处理回滚，而不像EJB 那样调用Transaction.rollback()处理之

####获取到JTA UserTransaction 的引用
当在客户端（无论web 客户端或应用客户端）使用基于EJB 的编程式事务模型时，在使用EJB 时必须获得InitialContext
<pre class="prettyprint">
InitialContext ctx = new InitialContext()
UserTransaction txn = (UserTransaction)
ctx.lookup("javax.transaction.UserTransaction")
</pre>

上代码片段中的查找名称（“javax.transaction.UserTransaction”）是与应用服务器类型相关的
基本上客户端里的这部分代码在不同的应用服务器上是不通用的。不同的应用服务器将UserTransaction绑定给不同的JNDI 名

JBoss "UserTransaction"
WebLogic "javax.transaction.UserTransaction"
WebSphere v5+ "java:comp/UserTransaction"
WebSphere v4 "jta/usertransaction"
SunONE "java:comp/UserTransaction"
JRun4 "java:comp/UserTransaction"
Resin "java:comp/UserTransaction"
Orion "java:comp/UserTransaction"
JOnAS "java:comp/UserTransaction"

####编程式事务中的编码陷阱
开发人员必须十二分的警惕异常的处理（exception handling）
看看下面的代码，我们仅仅捕获和处理了应用异常（一个被检查异常，checked exception），
而忽略了运行时异常（runtime exception）：

也就是说，一旦使用编程式事务，开发人员便有责任去管理事务。一个方法，启动了事务，它就必须负责终止该事务

####事务上下文问题