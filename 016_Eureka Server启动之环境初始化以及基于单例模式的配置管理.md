 

web容器（tomcat还是jetty）启动的时候，把eureka-server作为一个web应用给带起来的时候，eureka-server的初始化的逻辑，监听器，EurekaBootStrap。eureka-core里面，监听器的执行初始化的方法，是contextInitialized()方法，这个方法就是整个eureka-server启动初始化的一个入口。

 

快捷键，在IntelliJ IDEA里面读源码，如果要跟进一个方法里，那么就用ctrl + 鼠标左键，按住ctrl键，鼠标左击一下，就可以跟进一个方法里去看，或者是一个类里。如果要回退到上一个地方，就按ctrl + alt + 方向键左。

 

initEurekaEnvironment()，初始化eureka-server的环境

 

在这里，其实会调用ConfigurationManager.getConfigInstance()方法，这个方法，其实就是初始化ConfigurationManager的实例，也就是一个配置管理器的初始化的这么一个过程。ConfigurationManager是什么呢？看字面意思都猜的出来，配置管理器，管理eureka自己的所有的配置，读取配置文件里的配置到内存里，供后续的eureka-server运行来使用。

 

配置管理器，是一个单例，用的是单例模式

 

经典的单例模式的使用，比较广泛的运用，倒不是说用那个静态内部类的方法，很多人觉得那样写加了一个内部类，比较麻烦。在各种开源项目里，你看源码，人家用的比较多的，其实是double check + volatile。

 

（看到这里，如果没有看过面试突击第二季里的volatile原理的同学，麻烦先把面试突击第二季的volatile原理给看一下，因为在人家源码里会大量的看到volatile的使用，所以我们要先熟悉这个关键词的原理，不是轻量级锁，保证多线程可见性的）

 

这里，重点理解和熟悉人家的源码的单例模式是怎么用的，单例模式一个经典的作用，就是将配置作为单例，但是一般来说如果你要自己解析配置文件，读取配置的话，一般来说除非你是底层自研的框架。此时你可以设计一个ConfigurationManager，就是一个配置管理器，一般这个会做成单例的。

 

如果是单例模式的话，看看人家是如何用double check + volatile来实现线程安全的单例模式的。。。。可以参考一下

 

将ConfigurationManager（配置管理器，初始化单例的过程）

 

（1）创建一个ConcurrentCompositeConfiguration实例，这个东西，其实就是代表了所谓的配置，包括了eureka需要的所有的配置。在初始化这个实例的时候，调用了坑爹的clear()方法，fireEvent()发布了一个事件（EVENT_CLEAR），fireEvent()这个方法其实是父类的方法，牵扯比较复杂的另外一个项目（ConfigurationManager本身不是属于eureka的源码，是属于netflix config项目的源码）。

 

（2）就是往上面的那个ConcurrentCompositeConfiguration实例加入了一堆别的config，然后搞完了以后，就直接返回了这个实例，就是作为所谓的那个配置的单例

 

（3）看源码，不要较真儿，你是不可能100%把一个源码的每一行代码的细节都仔仔细细的看完，都看懂，都背下来的，是不可能的，抓大放小，搞清楚大的流程、架构以及核心的一些实现的机制和细节

 

（4）初始化数据中心的配置，如果没有配置的话，就是DEFAULT data center

 

（5）初始化eurueka运行的环境，如果你没有配置的话，默认就给你设置为test环境

 

（6）initEurekaEnvironment的初始化环境的逻辑，就结束了

 

 

读源码，以后每一讲结束之后，我都会给大家做点总结，把思路给拉回来，给大家布置课后自己看源码的作业和思路：

 

（1）自己把ConfigurationManager的单例初始化的过程看一下

（2）重点理解，ConfigurationnManager源码中体现的double chehck + volatile的单例实现模式的思想和技巧

（3）理解initEurekaEnvironment，初始化环境的逻辑，数据中心 + 运行环境，没设置的话，都给你搞成默认的和测试的







archaius是Netflix公司开源项目之一，基于java的配置管理类库，主要用于多配置存储的动态获取

