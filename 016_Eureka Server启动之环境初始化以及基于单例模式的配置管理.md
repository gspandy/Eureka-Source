 

web容器（tomcat还是jetty）启动的时候，把eureka-server作为一个web应用运行，

eureka-server的初始化的逻辑为EurekaBootStrap监听器，com.netflix.eureka.EurekaBootStrap



eureka-core里面，EurekaBootStrap监听器的执行初始化的方法，

com.netflix.eureka.EurekaBootStrap#contextInitialized

contextInitialized()方法

这个方法就是整个eureka-server启动初始化的一个入口。



 源码分析

```java
    @Override
    public void contextInitialized(ServletContextEvent event) {
        try {
            initEurekaEnvironment();
            initEurekaServerContext();

            ServletContext sc = event.getServletContext();
            sc.setAttribute(EurekaServerContext.class.getName(), serverContext);
        } catch (Throwable e) {
            logger.error("Cannot bootstrap eureka server :", e);
            throw new RuntimeException("Cannot bootstrap eureka server :", e);
        }
    }

```



initEurekaEnvironment()，初始化eureka-server的环境

 com.netflix.config.ConfigurationManager

调用ConfigurationManager.getConfigInstance()方法，

初始化ConfigurationManager的实例，配置管理器的初始化过程。

管理eureka自己的所有的配置，读取配置文件里的配置到内存里，供后续的eureka-server运行来使用。

 

ConfigurationManager配置管理器，是一个单例，用的是单例模式double check + volatile



ConfigurationManager（配置管理器，初始化单例的过程）



com.netflix.config.ConfigurationManager#getConfigInstance()

com.netflix.config.ConfigurationManager#getConfigInstance(boolean)

com.netflix.config.ConfigurationManager#createDefaultConfigInstance





（1）创建一个 com.netflix.config.ConcurrentCompositeConfiguration实例，代表了eureka需要的所有的配置。

在初始化这个实例的时候，调用了clear()方法，

com.netflix.config.ConcurrentCompositeConfiguration#ConcurrentCompositeConfiguration()

com.netflix.config.ConcurrentCompositeConfiguration#clear



fireEvent()发布了一个事件（EVENT_CLEAR）

fireEvent()这个方法是父类的方法，事件监听器

牵扯比较复杂的另外一个项目（ConfigurationManager本身不是属于eureka的源码，是属于netflix config项目的源码）。



（2）ConcurrentCompositeConfiguration实例加入其他config，返回了这个单例

（3）初始化数据中心的配置，如果没有配置的话，就是DEFAULT data center

（4）初始化eurueka运行的环境，如果你没有配置的话，默认就







archaius是Netflix公司开源项目之一，基于java的配置管理类库，主要用于多配置存储的动态获取

