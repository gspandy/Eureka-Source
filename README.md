# EurekaSource
 EurekaSource 源码





eureka分区              eureka.preferSameZone参数

https://segmentfault.com/a/1190000014107639   



常用配置

[http://blog.leanote.com/post/mzoro/eureka-%E4%B8%80%E4%BA%9B%E5%B8%B8%E7%94%A8%E7%9A%84%E9%85%8D%E7%BD%AE](http://blog.leanote.com/post/mzoro/eureka-一些常用的配置)



jersey

Jersey是一个RESTFUL请求服务JAVA框架，与常规的JAVA编程使用的struts框架类似，它主要用于处理业务逻辑层。

与springmvc 的区别：

1. jersey同样提供DI，是由glassfish hk2实现，也就是说，如果想单独使用jersey一套，需要另外学习Bean容器；

2. MVC出发点即是WEB，但jersey出发点确实RESTFull，体现点在与接口的设计方面，
如MVC返回复杂结构需要使用ModelAndView,而jersey仅仅需要返回一个流或者文件句柄；

3. jersey提供一种子资源的概念，这也是RESTFull中提倡所有url都是资源；

4. jersey直接提供application.wadl资源url说明；

5. MVC提供Session等状态管理，jersey没有，这个源自RESTFull设计无状态化；

6. Response方法支持更好返回结果，方便的返回Status，包括200，303，401，403；

7. 提供超级特别方便的方式访问RESTFull;

SpringMVC在开发REST应用时，是不支持JSR311/JSR339标准的。如果想要按照标准行事，最常用的实现了这两个标准的框架就是Jersey和CxF了。但是，因为Jersey是最早的实现，也是JSR311参考的主要对象，所以，可以说Jersey就是事实上的标准（类似Hibernate是JPA的事实上的标准），也是现在使用最为广泛的REST开发框架之一。

Jersey提供了在SE环境下的部署，即使用内置的服务器来发布REST服务。提供了对Jetty，JDK HttpServer，Grizzly HttpServer，和一个内置的Simple HttpServer来部署。





依赖注入

@ProvidedBy      

com.google.inject.ProvidedBy             

Guice是一个轻量级的依赖注入框架。用于对象之间的依赖的注入。



@Inject                 

javax.inject.Inject	              

一个新的注入依赖规范，既能支持Google Gucie，还能支持Spring



  \* @Inject - Identifies injectable constructors, methods, and fields
  \* @Qualifier - Identifies qualifier annotations
  \* @Scope - Identifies scope annotations
  \* @Named - String-based qualifier
  \* @Singleton - Identifies a type that the injector only instantiates once

@Inject的optional默认为false，注入时如果依赖不存在，则报错停止，当使用@Inject(optional = true)时可达到忽然依赖是否存在的效果



@JsonCreator

当json在反序列化时，默认选择类的无参构造函数创建类对象，当没有无参构造函数时会报错，@JsonCreator作用就是指定反序列化时用的无参构造函数。构造方法的参数前面需要加上@JsonProperty,否则会报错。







https://zhuanlan.zhihu.com/p/58093669   秒懂设计模式之建造者模式

**当一个类的构造函数参数个数超过4个，而且这些参数有些是可选的参数，考虑使用构造者模式。**











