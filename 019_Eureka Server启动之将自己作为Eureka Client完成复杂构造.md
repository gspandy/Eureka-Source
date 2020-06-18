EurekaClientConfig，这个东西也是个接口，也是对外暴露了一大堆的配置项，看名字就知道了啊，这里包含的是EurekaClient相关的一些配置项。也是去读eureka-client.properties里的一些配置，只不过他关注的是跟之前的那个EurekaInstanceConfig是不一样的，代表了服务实例的一些配置项，这里的是关联的这个EurekaClient的一些配置项。

 

基于ApplicationInfoManager（包含了服务实例的信息、配置，作为服务实例管理的一个组件），eureka client相关的配置，一起构建了一个EurekaClient，但是构建的时候，用的是EurekaClient的子类，DiscoveryClient。

 

说一下基于接口的配置项读取的优势，跟直接拿常量来读取配置项的区别

 

假如说你现在要获取一个配置，你调用了Config.get(ConfigKeys.XX_XX_XX)，结果有可能说，你不小心把常量给打错了，或者是搞混了。但是如果你是基于接口，Config.getXxXxXx()，这种方式，还是不错的，不太容易会搞错了。

 

优势，Config.get(ConfigKeys.XX_XX_XX)这行代码出现在了20个地方，结果坑爹的事情是，有一天，版本升级，要修改常量的名字，常量名字一修改，你可能要到20个地方去修改，很麻烦，很容易出错。Config.getXxXxXx()，20个地方都调用了这个方法罢了，如果要调整这个常量的名称，直接在方法里修改即可，对调用这个配置项的地方，都是透明的。

 

很值得大家来吸收和使用

 

AppName，代表了一个服务名称，但是一个服务可能部署多台机器，每台机器上部署的就是一个服务实例，ServiceA/001

 

讲一个看源码的技巧，你就顺着整个你平时用这个技术的一个思路去看就好了，而且在看的过程中，记住一个原则：抓大放小，把握大的架构、流程、机制，核心的细节看一下，千万别强迫自己把每个细节每行代码都要看懂。到了后面，针对各个功能、流程以及高级特性，有针对性的看细节的代码，包括各个参数是怎么用的。

 

AtomicLong和AtomicReference，都看一下面试突击第二季，原子性操作的一些类

 

如果是eureka server的话，我们在玩儿spring cloud的时候，会将这个fetchRegistry给手动设置为false，因为如果就单个eureka server启动的话，就不能设置，但是如果是eureka server集群的话，就还是要保持为true。registerWithEureka是否要设置为true。

 

（1）读取EurekaClientConfig，包括TransportConfig

（2）保存EurekaInstanceConfig和InstanceInfo

（3）处理了是否要注册以及抓取注册表，如果不要的话，释放一些资源

（4）支持调度的线程池

（5）支持心跳的线程池

（6）支持缓存刷新的线程池

（7）EurekaTransport，支持底层的eureka client跟eureka server进行网络通信的组件，对网络通信组件进行了一些初始化的操作

（8）如果要抓取注册表的话，在这里就会去抓取注册表了，但是如果说你配置了不抓取，那么这里就不抓取了

（9）初始化调度任务：如果要抓取注册表的话，就会注册一个定时任务，按照你设定的那个抓取的间隔，每隔一定时间（默认是30s），去执行一个CacheRefreshThread，给放那个调度线程池里去了；如果要向eureka server进行注册的话，会搞一个定时任务，每隔一定时间发送心跳，执行一个HeartbeatThread；创建了服务实例副本传播器，将自己作为一个定时任务进行调度；创建了服务实例的状态变更的监听器，如果你配置了监听，那么就会注册监听器

 

课后的作业，你就把EurekaClient初始化的过程的源码，按照上面的思路，自己仔细去看一遍，但是注意，抓大放小，不要过度纠结于细节，把握大的流程就可以了