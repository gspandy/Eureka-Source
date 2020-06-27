

 

（1）构造 PeerAwareInstanceRegistry

 com.netflix.eureka.registry.PeerAwareInstanceRegistry



PeerAware，可以识别eureka server集群的：peer，多个同样的东西组成的一个集群，peers集群，peer就是集群中的一个实例

 

InstanceRegistry：服务实例注册表，这个里面放了所有的注册到这个eureka server上来的服务实例，就是一个服务实例的注册表

 

com.netflix.eureka.registry.PeerAwareInstanceRegistryImpl

com.netflix.eureka.registry.AbstractInstanceRegistry#AbstractInstanceRegistry

```
this.recentCanceledQueue = new CircularQueue<Pair<Long, String>>(1000);//最近摘除实例
this.recentRegisteredQueue = new CircularQueue<Pair<Long, String>>(1000);//最近注册实例
```

PeerAwareInstanceRegistry：可以感知eureka server集群的服务实例注册表，eureka client（作为服务实例）过来注册的注册表，而且这个注册表是可以感知到eureka server集群的。





假如有一个eureka server集群的话，这里包含了其他的eureka server中的服务实例注册表的信息的。

 

（2）构造PeerEurekaNodes

 

PeerEurekaNodes，代表了eureka server集群，peers大概来说多个相同的实例组成的一个集群，peer就是peers集群中的一个实例，PeerEurekaNodes，大概来说，猜测，应该是代表的是eureka server集群

 com.netflix.eureka.cluster.PeerEurekaNodes#start

com.netflix.eureka.cluster.PeerEurekaNodes#updatePeerEurekaNodes

 

（3）构造了一个东西：EurekaServerContext

 

代表了当前这个eureka server的一个服务器上下文，包含了服务器需要的所有的东西。将这个东西放在了一个holder中，以后谁如果要使用这个EurekaServerContext，直接从这个holder中获取就可以了。这个也是一个比较常见的用法，就是将初始化好的一些东西，放在一个holder中，然后后面的话呢，整个系统运行期间，谁都可以来获取，在任何地方任何时间，谁都可以获取这个上下文，从里面获取自己需要的一些组件。

 

（4）EurekaServerContext.initialize()          peerEurekaNodes.start();

 

```
public void initialize() {
    logger.info("Initializing ...");
    peerEurekaNodes.start();  //定时更新集群注册信息
    try {
        registry.init(peerEurekaNodes);//初始化注册表
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
    logger.info("Initialized");
}
```

将eureka server集群给启动起来，更新一下eureka server集群的信息，让当前的eureka server感知到所有的其他的eureka server。然后搞一个定时调度任务，就一个后台线程，每隔一定的时间，更新eureka server集群的信息。

 

registry.init(peerEurekaNodes);

 基于eureka server集群的信息，来初始化注册表，肯定是将eureka server集群中所有的eureka server的注册表的信息，都抓取过来，放到自己本地的注册表里去

 

（5）registry.syncUp();

从相邻的一个eureka server节点拷贝注册表的信息，如果拷贝失败，就找下一个

 

（6）EurekaMonitors.registerAllStats();

跟eureka自身的监控机制相关联的

 
