---
layout: post
title: struts1 流程
description:
- struts1 流程
categories:
- java
- struts1
---
struts启动
启动容器 -> 读取web.xml文件 -> load-on-startup -> 创建ActionServlet 

ActionServlet的构建
 ActionServlet[init()] 
 -> initInternal()日志错误警告信息 
 -> initOther()读取web.xml config参数
 -> initServlet()读取web.xml的DTD验证struts-config语法 
 -> initModuleConfig()读取struts文件返回ModuleConfig对象放到ServletContext里
 -> initModuleMessageResources()读取struts文件的MessageResources 放到ServletContext里
 —> initModuleDataSources() 读取struts文件的DataSources
 -> initModulePlugIns() 读取struts文件的所有插件


用户View --请求.do--> ActionServlet
 ActionServlet接收到请求{doPost doGet}
 ->调用prcess()方法[在此方法中实现如下]
 ->调用ModuleUtils类的 selectModule方法来选择处理此请求的模块
 ->把应用模块相关的ModuleConfig MessageResources放到request里
 ->获得一个RequestProccessor实例调用其process()帮助处理

RequestProccessor的process处理细节
 ->processMultipart()要是提交方式为post而且类型为Multipart则返回从新包装的HttpServletRequest对象
 ->processPath()获取url路径以便选择struts Action组件
 ->processLocale()把用户提交的Locale信息保存到session中
 ->processContent()设置response.setContentType的值
 ->processNoCache()设置报头Cache-Control Pragma Expires的值
 ->processPreprocess()直接返回true 子类可以覆盖该方法实现自己的操作
 ->processMapping()查找对应的ActionMapping 不存在则返回错误信息
 ->processRoles()判断对应的Action是否配置了安全角色 用户是否有权限
 ->processActionForm()查找ActionMapping是否配置ActionForm 有则查找该实例不存在则创建
 ->processPopulate()查找ActionMapping是否配置ActionForm 有则调用reset方法清空再装载表单数据
 ->processValidate()调用ActionForm的validate方法返回ActionErrors 要是存在ActionMessage对象则验证失败
 ->processForward()查找ActionMapping是否配置forward 有则采用forward结束流程
 ->processInclude()查找ActionMapping是否配置include 有则采用include结束流程
 ->processActionCreate()查找是否有对应的Action实例 不存在则创建
 ->processActionPerform()调用Action的execute方法
 ->processForwardConfig() 最后这个不用我说了吧?! 