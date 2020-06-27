



Application：如果是一个eureka client客户端，扮演的角色其实是一个服务，作为一个服务会向eureka server去注册。Application，就是一个eureka client -> Application，服务。

 

EurekaInstanceConfig，将eureka-client.properties文件中的配置加载到ConfigurationManager中去，然后基于EurekaInstanceConfig对外暴露的接口来获取这个eureka-client.properties文件中的一些配置项的读取，而且提供了所有配置项的默认值

 

EurekaInstanceConfig是服务实例相关的一些配置。

eureka server同时也是一个eureka client，因为他可能要向其他的eureka server去进行注册，组成一个eureka server的集群。

eureka server把自己也当做是一个eureka client，也就是一个服务实例，所以这里有Application、Instance等概念的

InstanceInfo，你可以认为就是当前这个服务实例的信息，直接用了构造器模式，用InstanceInfo.Builder来构造一个复杂的代表一个服务实例的InstanceInfo对象。

核心的思路是，从之前的那个EurekaInstanceConfig中，读取各种各样的服务实例相关的配置信息，再构造了几个其他的对象，最终完成了InstanceInfo的构建。

 

eureka server自己本身代表的一个服务实例，把自己作为一个服务注册到别的eureka server上去

com.netflix.appinfo.EurekaInstanceConfig

com.netflix.appinfo.providers.EurekaConfigBasedInstanceInfoProvider

构造器模式: InstanceInfo.Builder，拿到静态内部类的对象，InstanceInfo.Builder.newBuilder()，这个里面就构造了一个InstanceInfo。

基于builder去set各种需要的属性和配置，最终完成的一个复杂的InstanceInfo服务实例对象。

 

com.netflix.appinfo.ApplicationInfoManager

直接基于EurekaInstanceConfig和InstnaceInfo，构造了一个ApplicationInfoManager，后面会基于这个ApplicationInfoManager对服务实例进行一些管理。

 



（1）加载eureka-client.properties文件的配置，对外提供EurekaInstanceConfig接口的逻辑，去看一下，巩固一下基于接口的配置项读取的思路

（2）基于构造器模式完成的InstanceInfo（服务实例）的构造的一个过程，精华，闪光点，设计模式是怎么用的，怎么玩儿的

（3）EurekaInstanceConfig（代表了一些配置），搞了InstanceInfo（服务实例），基于这俩玩意儿，搞了一个ApplicationInfoManager，作为服务实例的一个管理器

 

 构造器模式 

 com.netflix.appinfo.InstanceInfo   

com.netflix.appinfo.InstanceInfo.Builder   静态内部类  构造器

 

 