---
layout: post
title: spring 技术内幕
description: spring 技术内幕
categories:
- spring
tags:
- spring
---
interface BeanFactory
 	String FACTORY_BEAN_PREFIX = "&";
	getBean(...)
	containsBean(String paramString);
	isSingleton(String paramString)
  	isPrototype(String paramString)
    isTypeMatch(String paramString, Class<?> paramClass)
	getType(String paramString)
    getAliases(String paramString)
    	
HierarchicalBeanFactory
	getParentBeanFactory(),使得具备双亲IoC容器管理功能
	
另一条接口设计线路以ApplicationContext
ListableBeanFactory extends BeanFactory，细化接口功能
ApplicationContext继承 MessageSource, ApplicationEventPublisher, ResourcePatternResolver等,在BeanFactory简单IoC基础上加入高级容器特性支持