

 

第一个流程

eureka server（注册中心）的启动，启动一个eureka client（服务），注册服务，获取服务列表

底层的机制：心跳机制、抓取注册表、自我保护机制

eureka，非常适合的是通过测试代码，打断点跟进



com.netflix.eureka.resources.EurekaClientServerRestIntegrationTest    	服务端测试

com.netflix.eureka.ExampleEurekaClient														 客户端测试

 

**1、手动启动eureka注册中心**

 



eureka-server的src/test/java

com.netflix.eureka.resources包

com.netflix.eureka.resources.EurekaClientServerRestIntegrationTest

手动启动Eureka注册中心



com.netflix.eureka.resources.EurekaClientServerRestIntegrationTest#setUp

setUp()方法，在运行集成测试之前，调用startServer()方法先启动注册中心。



com.netflix.eureka.resources.EurekaClientServerRestIntegrationTest#startServer

在startServer()里面替换掉一些代码，方便启动注册中心

eurekaServiceUrl = "http://localhost:8080/v2"; 

```java
    private static void startServer() throws Exception {
		
        /*        
		File warFile = findWar();
        server = new Server(8080);
        WebAppContext webapp = new WebAppContext();
        webapp.setContextPath("/");
        webapp.setWar(warFile.getAbsolutePath());
        server.setHandler(webapp);
        server.start();
        eurekaServiceUrl = "http://localhost:8080/v2";
        */

        server = new Server(8080);
        WebAppContext webAppCtx = new WebAppContext(new File("./eureka-server/src/main/webapp").getAbsolutePath(), "/");
        webAppCtx.setDescriptor(new File("./eureka-server/src/main/webapp/WEB-INF/web.xml").getAbsolutePath());
        webAppCtx.setResourceBase(new File("./eureka-server/src/main/resources").getAbsolutePath());
        webAppCtx.setClassLoader(Thread.currentThread().getContextClassLoader());
        server.setHandler(webAppCtx);
        server.start();
        eurekaServiceUrl = "http://localhost:8080/v2";

    }
```

 原有逻辑把eureka-server打成一个war包，然后找到那个war包去启动。

现在这段代码，直接就是自己加载eureka-server的各种资源，比如web.xml，配置文件啥的，然后直接就启动注册中心了



在setUp()方法里加入：

Thread.sleep(Long.MAX_VALUE)

启动之后就hang住，然后可以用别的client去注册他。

 

 

**2、启动eureka客户端**

 

eureka-examples

src/main/java的com.netflix.eureka包下



ExampleEurekaClient类

com.netflix.eureka.ExampleEurekaClient

模拟eureka客户端

 

在集成测试类里com.netflix.eureka.ExampleEurekaClient的

com.netflix.eureka.ExampleEurekaClient#injectEurekaConfiguration

injectEurekaConfiguration()方法

复制到ExampleEurekaClient类里，然后在这个类的main()的第一行，就加入这个方法的调用，提前设置好一些参数。

然后就可以运行main方法，往注册中心发送请求就可以开始调试了



 

 

