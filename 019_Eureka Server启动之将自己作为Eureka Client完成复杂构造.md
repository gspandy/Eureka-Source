EurekaClientConfig

EurekaClient相关的一些配置项,eureka-client.properties里的一些配置



EurekaInstanceConfig

Eureka server服务实例的一些配置项

 

基于ApplicationInfoManager（包含了服务实例的信息、配置，作为服务实例管理的一个组件），eureka client相关的配置，一起构建了一个EurekaClient，但是构建的时候，用的是EurekaClient的子类，DiscoveryClient。



AppName，代表了一个服务名称，但是一个服务可能部署多台机器，

每台机器上部署的就是一个服务实例，ServiceA/001

 

 

如果是eureka server的话，在spring cloud的时候，会将这个fetchRegistry给手动设置为false，

因为如果单个eureka server启动的话，就不能设置，但是如果是eureka server集群的话，就还是要保持为true。registerWithEureka是否要设置为true。

 

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