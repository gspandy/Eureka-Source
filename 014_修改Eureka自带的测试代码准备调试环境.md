如果你要看这个源码，你从哪儿来入手？

 

第一个流程，一定是看eureka server（注册中心）的启动，启动一个eureka client（服务），注册服务，获取服务列表

 

其他的就是底层的机制：心跳机制、抓取注册表、自我保护机制

 

要不然就是自己基于这个源码来写个HelloWorld程序，加上断点来跑来看，要不然就是从他们的单元测试、集成测试的代码作为入口，来打断点，跟进去调试

 

eureka，非常适合的是通过测试代码，打断点跟进去看

 

**1、手动启动eureka注册中心**

 

eureka-server的src/test/java下，有一个com.netflix.eureka.resources包，里面有一个EurekaClientServerRestIntegrationTest类，我们就通过这个类的示例代码，尝试手动启动Eureka注册中心吧。

 

有一个setUp()方法，就是会在运行集成测试之前，调用startServer()方法先启动注册中心。

 

所以我们在startServer()里面替换掉一些代码，直接就把注册中心给跑起来。

 

server = new Server(8080);

 

WebAppContext webAppCtx = new WebAppContext(new File("./eureka-server/src/main/webapp").getAbsolutePath(), "/");

webAppCtx.setDescriptor(new File("./eureka-server/src/main/webapp/WEB-INF/web.xml").getAbsolutePath());

webAppCtx.setResourceBase(new File("./eureka-server/src/main/resources").getAbsolutePath());

webAppCtx.setClassLoader(Thread.currentThread().getContextClassLoader());

server.setHandler(webAppCtx);

server.start();

 

eurekaServiceUrl = "http://localhost:8080/v2";

 

说白了，上面的那段代码啥意思呢？本来很麻烦，他是要你先把eureka-server打成一个war包，然后他这里找到那个war包去启动，但是太麻烦了啊，每次都打包，耗费时间，不好调试。现在这段代码，直接就是自己加载eureka-server的各种资源，比如web.xml，配置文件啥的，然后直接就启动注册中心了

 

在setUp()方法里加入：Thread.sleep(Long.MAX_VALUE)，就是让他启动之后就hang住，然后可以用别的client去注册他。

 

这里的启动注册中心的代码都可以看看

 

后面咱们可以通过这个为入口，这里不是好多单元测试方法么，其实把单元测试方法都调试一遍，就差不多了，每个单元测试方法都是集成测试一个功能，都看一遍，你对eureka server提供的各种功能理解的就差不多了

 

**2、启动eureka客户端**

 

eureka-examples里面，src/main/java的com.netflix.eureka包下，有个ExampleEurekaClient类，就用这个类来模拟eureka客户端好了。

 

在之前那个集成测试类里，有个injectEurekaConfiguration()方法，复制到这个ExampleEurekaClient类里，然后在这个类的main()的第一行，就加入这个方法的调用，就是提前设置好一些参数罢了。

 

然后就可以运行main方法，往注册中心发送请求就可以开始调试了

 

而且eureka-examples里面，有别的例子，都可以按照上面的方法，来分别调试，基本把这些东西都弄完，整个eureka的原理就全部搞懂了

 

 

