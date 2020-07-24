**1、心跳检测**   

又名renew机制

eureka客户端，默认会每隔30秒发送一次心跳的eureka注册中心

eureka服务端在90秒内没收到一个eureka客户端的心跳，那么就摘除这个服务实例，别人就访问不到这个服务实例了

 

心跳检测 服务续约 renew机制

 

eureka.instance.leaseRenewallIntervalInSeconds

eureka.instance.leaseExpirationDurationInSeconds

  

如果90秒内没收到一个client的服务续约，eviction机制，将服务实例从注册表里给摘除掉

 

**2、注册表抓取**

 

默认情况下，客户端每隔30秒去服务器抓取最新的注册表，然后缓存在本地，通过下面的参数可以修改。

 

eureka.client.registryFetchIntervalSeconds

 

**3、自定义元数据**

 

eureka:

instance:

hostname: localhosto

**metadata-map**:

company-name: orange

 

### 通过配置静态设置：

表单的任何配置 `eureka.metadata.mykey=myvalue`都会将k：v对`mykey:myvalue`添加到eureka的元数据映射中。

### 通过代码动态设置：

要动态地执行此操作，您首先需要提供自己的[EurekaInstanceConfig](https://github.com/Netflix/eureka/blob/master/eureka-client/src/main/java/com/netflix/appinfo/EurekaInstanceConfig.java)接口的自定义实现。然后，您可以重载该`public Map getMetadataMap()`方法以返回包含所需元数据值的元数据映射。有关提供上述基于配置的系统的示例实现，请参见[PropertiesInstanceConfig](https://github.com/Netflix/eureka/blob/master/eureka-client/src/main/java/com/netflix/appinfo/PropertiesInstanceConfig.java)。



**4、自我保护模式**

 

如果在eureka控制台看到下面的东西：

 

#### ***\*EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.\****

 

这就是eureka进入了自我保护模式

如果客户端的心跳失败了超过一定的比例，或者说在一定时间内（15分钟）接收到的服务续约低于85%，那么就会认为是自己网络故障了，导致client无法发送心跳。(错误的说法)

eureka注册中心会先给保护起来，不会立即把失效的服务实例摘除，在测试的时候一般都会关闭这个自我保护模式：

 

eureka.server.enable-self-preservation: false



在生产环境里面，他怕自己因为自己有网络问题，导致别人没法给自己发心跳，就不想胡乱把别人给摘除，他就进入保护模式，不再摘除任何实例，等到自己网络环境恢复。



 自我保存阈值，请设置属性：`eureka.renewalPercentThreshold=[0.0, 1.0]`。

如果Eureka服务器检测到数量超过预期的注册客户端已经以不正当的方式终止了他们的连接，并且同时正等待逐出，则它们将进入自我保存模式。这样做是为了确保灾难性的网络事件不会清除eureka注册表数据，并将其传播到所有客户端。

为了更好地理解自我保存，首先了解尤里卡客户如何“结束”他们的注册生命周期将很有帮助。Eureka协议要求客户端永久离开时执行明确的注销操作。例如，在提供的Java客户端中，这是通过shutdown（）方法完成的。任何连续3次心跳续订失败的客户端将被视为不正常的终止，并且将由后台驱逐过程逐出。当当前注册表中的> 15％处于此更高状态时，将启用自我保存。

在自我保存模式下，eureka服务器将停止逐出所有实例，直到发生以下情况之一：

1. 它看到的心跳续订次数又回到了预期的阈值之上，或者
2. 自我保护功能已禁用（请参见下文）

默认情况下会启用自我保留，并且启用自我保留的默认阈值>当前注册表大小的15％。

 

 **云配置**

如果您在云环境中运行，则需要传递java命令行属性-Deureka.datacenter = cloud，以便Eureka客户端/服务器知道初始化特定于AWS云的信息

Deureka.datacenter = cloud







eureka至少需要配置以下内容：

```
Application Name (eureka.name)
Application Port (eureka.port)
Virtual HostName (eureka.vipAddress)
Eureka Service Urls (eureka.serviceUrls)
```