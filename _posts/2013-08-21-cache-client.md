---
layout: post
title: cache-client
description: cache-client
---
###入口
src/main/resources/META-INF/spring.handlers

http\://www.handpay.com.cn/schema/cache=com.handpay.framework.cache.client.config.HandpayCacheNamespaceHandler

HandpayCacheNamespaceHandler extends NamespaceHandlerSupport


@Override
public void init() {
	registerBeanDefinitionParser("cache-executor", new CacheExecutorParser());
	registerBeanDefinitionParser("annotation-driven", new AnnotationDrivenParser());
}
###配置
<pre class="prettyprint"><xmp>
<?xml version="1.0" encoding="GBK"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:hpcache="http://www.handpay.com.cn/schema/cache"
	xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd  
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.handpay.com.cn/schema/cache
        http://www.handpay.com.cn/schema/cache/handpay-cache.xsd">

	<hpcache:cache-executor appcode="dev">

		<hpcache:server-address>
			<hpcache:http
				endpoint="http://127.0.0.1:8080/cache-monitor/ws/cache-server.htm" />
		</hpcache:server-address>
	</hpcache:cache-executor>

	<bean id="cacheClient" class="com.ddatsh.maven.CacheExecutorExample">
		<property name="executor" ref="cacheExecutor" />
	</bean>

</beans></xmp></pre>

###使用
<pre class="prettyprint">
ApplicationContext ctx = new ClassPathXmlApplicationContext("application-cache.xml");
CacheExecutor executor = (CacheExecutor) ctx.getBean("cacheExecutor");

executor.set("name", "dd");

executor.executeInTransaction(new TransactionAction() {
	public void doInTxAction(CacheExecutorOperation operation) throws CacheClientException {
		operation.set("name", "dd");
	}
});

</pre>

<pre class="prettyprint">
interface CacheExecutor extends TransactionCacheExecutor

interface TransactionCacheExecutor extends IntegratedOperation
	void executeInTransaction(TransactionAction tx) throws CacheClientException;

interface IntegratedOperation extends GetOperation, SetOperation,
    DeleteOperation, AtomicCounterOperation, KeyOperation, AppendableOperation 
</pre>

##IntegratedOperation
<pre class="prettyprint">
子接口有两个
interface CacheExecutorOperation extends IntegratedOperation
interface TransactionCacheExecutor extends IntegratedOperation
	void executeInTransaction(TransactionAction tx) throws CacheClientException
</pre>

###CacheExecutorOperation实现之DefaultCacheExecutorOperation
<pre class="prettyprint">
被事务回调用
class DefaultCacheExecutorOperation implements CacheExecutorOperation
/**
* 委托使用的操作对象
*/
    private IntegratedOperation operation;

    public DefaultCacheExecutorOperation(IntegratedOperation operation) {
        this.operation = operation;
    }

各种get,set委托
</pre>

###TransactionCacheExecutor 两个子接口
<pre class="prettyprint">
interface CacheExecutor extends TransactionCacheExecutor
interface InternalCacheExecutor extends TransactionCacheExecutor
	 int DEFAULT_EXPIRATION_SECONDS = 1800;
</pre>