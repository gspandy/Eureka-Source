

EurekaClientConfig

com.netflix.discovery.EurekaClientConfig

com.netflix.discovery.DefaultEurekaClientConfig#DefaultEurekaClientConfig()

EurekaClient相关的一些配置项,eureka-client.properties里的一些配置



EurekaInstanceConfig

com.netflix.appinfo.EurekaInstanceConfig

Eureka server服务实例的一些配置项

 



```
        //3 ，初始化eureka-server内部的一个eureka-client（用来跟其他的eureka-server节点进行注册和通信的）
        if (eurekaClient == null) {
            EurekaInstanceConfig instanceConfig = isCloud(ConfigurationManager.getDeploymentContext())//实例配置
                    ? new CloudInstanceConfig()
                    : new MyDataCenterInstanceConfig();
            
            applicationInfoManager = new ApplicationInfoManager(
                    instanceConfig, new EurekaConfigBasedInstanceInfoProvider(instanceConfig).get());
            
            EurekaClientConfig eurekaClientConfig = new DefaultEurekaClientConfig();//客户端配置
            eurekaClient = new DiscoveryClient(applicationInfoManager, eurekaClientConfig);//客户端
        } else {
            applicationInfoManager = eurekaClient.getApplicationInfoManager();
        }
```



## DiscoveryClient

基于ApplicationInfoManager（包含了服务实例的信息、配置，作为服务实例管理的一个组件），eureka client相关的配置，一起构建了一个EurekaClient，但是构建的时候，用的是EurekaClient的子类，DiscoveryClient。



com.netflix.discovery.DiscoveryClient#DiscoveryClient(com.netflix.appinfo.ApplicationInfoManager, com.netflix.discovery.EurekaClientConfig, com.netflix.discovery.AbstractDiscoveryClientOptionalArgs, com.netflix.discovery.shared.resolver.EndpointRandomizer)

com.netflix.discovery.DiscoveryClient#DiscoveryClient(com.netflix.appinfo.ApplicationInfoManager, com.netflix.discovery.EurekaClientConfig, com.netflix.discovery.AbstractDiscoveryClientOptionalArgs, javax.inject.Provider<com.netflix.discovery.BackupRegistry>, com.netflix.discovery.shared.resolver.EndpointRandomizer)



```
appPathIdentifier = instanceInfo.getAppName() + "/" + instanceInfo.getId();
```

AppName，代表了一个服务名称，但是一个服务可能部署多台机器，

每台机器上部署的就是一个服务实例，ServiceA/001

 

 

```java
        if (config.shouldFetchRegistry()) {
            this.registryStalenessMonitor = new ThresholdLevelsMetric(this, METRIC_REGISTRY_PREFIX + "lastUpdateSec_", new long[]{15L, 30L, 60L, 120L, 240L, 480L});
        } else {
            this.registryStalenessMonitor = ThresholdLevelsMetric.NO_OP_METRIC;
        }

        if (config.shouldRegisterWithEureka()) {
            this.heartbeatStalenessMonitor = new ThresholdLevelsMetric(this, METRIC_REGISTRATION_PREFIX + "lastHeartbeatSec_", new long[]{15L, 30L, 60L, 120L, 240L, 480L});
        } else {
            this.heartbeatStalenessMonitor = ThresholdLevelsMetric.NO_OP_METRIC;
        }

```

配置

fetchRegistry

registerWithEureka

如果是eureka server的话，在spring cloud的时候，会将这个fetchRegistry给手动设置为false，

因为如果单个eureka server启动的话，就不能设置，但是如果是eureka server集群的话，就还是要保持为true。registerWithEureka是否要设置为true。

 两个配置均为false， 那么Discovery的初始化将直接结束，表示该客户端既不进行服务注册也不进行服务发现



（1）读取EurekaClientConfig，包括TransportConfig

（2）保存EurekaInstanceConfig和InstanceInfo

（3）处理了是否要注册以及抓取注册表，如果不要的话，释放一些资源

（4）支持调度的线程池

（5）支持心跳的线程池

（6）支持缓存刷新的线程池

（7）EurekaTransport，支持底层的eureka client跟eureka server进行网络通信的组件，对网络通信组件进行了一些初始化的操作

（8）如果要抓取注册表的话，在这里就会去抓取注册表了，但是如果说你配置了不抓取，那么这里就不抓取了

（9）初始化调度任务：如果要抓取注册表的话，就会注册一个定时任务，按照你设定的那个抓取的间隔，每隔一定时间（默认是30s），去执行一个CacheRefreshThread，给放那个调度线程池里去了；如果要向eureka server进行注册的话，会搞一个定时任务，每隔一定时间发送心跳，执行一个HeartbeatThread；创建了服务实例副本传播器，将自己作为一个定时任务进行调度；创建了服务实例的状态变更的监听器，如果你配置了监听，那么就会注册监听器

 

