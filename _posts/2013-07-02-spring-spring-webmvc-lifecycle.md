---
layout: post
title: Spring mvc的生命周期
description: Spring mvc的生命周期
---
HandlerMapping,HandlerAdapter,HandlerExceptionResovler,ViewResolver都有order属性
这些接口每一个都可以注册多个实现，order越小越先执行，一般匹配到了后面的就不执行了

DispatcherServlet：整个Spring MVC的前端控制器，接管来自客户端的请求
HandlerMapping:DispatcherServlet会通过它处理客户端请求到各个(Controller)处理器的映射
HandlerAdapter:HandlerMapping根据它来调用Controller里需要被执行的方法
HandlerExceptionResolver:spring mvc处理流程中，异常抛出时交给它处理
ViewResolver:HandlerAdapter会把Controller中调用返回值最终包装成ModelAndView
ViewResolver会检查其中的view，如果view是一个字符串，它就负责处理这个字符串并返回一个真正的View
如果view是一个真正的View则不会交给它处理

为什么view即可以是字符串又会是View?
View:对应MVC 中的V， 此接口只有一个方法 render,用于视图展现
ModelAndView 对于解决上面介绍ViewResoler或者下面图片的疑惑，这个类中的view这个属性是 Object 类型的，它可以是一个视图名也可以是一个实际的View，这点我们观察其源码可以很清楚的看出来

private Object view;
 public void setViewName(String viewName) {
　　 this.view = viewName;
}

 public String getViewName() {
　　return (this.view instanceof String ? (String) this.view : null);
}

 public void setView(View view) {

　　this.view = view;

 }

 public View getView() {

　　return (this.view instanceof View ? (View) this.view : null);

 }     

![spring-webmvc-lifecycle]({{ site.url }}/imgs/java/spring/webmvc/lifecircycle.png)