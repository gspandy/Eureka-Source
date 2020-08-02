



# Eureka源码开发

**eureka-client**，**eureka-core**       代码包就这2个 ，web页面就一个  **eureka-server**





## Eureka 原理图



![springCloud-study-theory](../images/springCloud-study-theory.png)



## 常用配置

eureka.instance.leaseRenewallIntervalInSeconds

eureka.instance.leaseExpirationDurationInSeconds

eureka.client.registryFetchIntervalSeconds

eureka.server.enableSelfPreservation（enable-self-preservation）

eureka.server.renewalPercentThreshold

eureka.metadata.mykey=myvalue               在Eureka中传递外部元数据



fetchRegistry

registerWithEureka

如果是eureka server的话，在spring cloud的时候，会将这个fetchRegistry给手动设置为false，

因为如果单个eureka server启动的话，就不能设置，但是如果是eureka server集群的话，就还是要保持为true。registerWithEureka是否要设置为true。

 两个配置均为false， 那么Discovery的初始化将直接结束，表示该客户端既不进行服务注册也不进行服务发现



## Eureka 角色

Application    提供的服务接口

Applications   本地注册信息的缓存

```java
private final AtomicReference<Applications> localRegionApps = new AtomicReference<Applications>();
```

InstanceInfo    集群实例，服务可能会部署在多台机器上，每台机器上部署的就是一个服务实例。

（1）主机名、ip地址、端口号、url地址

（2）lease（租约）的信息：保持心跳的间隔时间，最近心跳的时间，服务注册的时间，服务启动的时间

eureka server

eureka client

ApplicationInfoManager   包含了服务实例的信息、配置，作为服务实例管理的一个组件

InstanceRegisterManager  实例注册管理器，专门来管理实例注册

appName，APPLICATION0，服务名称，ServiceA，或者是别的什么名称

instanceId，i-0000001，服务实例id，

AbstractInstanceRegistry， 和子类PeerAwareInstanceRegistryImpl 用来处理注册业务

ResponseCache   缓存



注册表

```
{
	“ServiceA”: {
		“001”: Lease<InstanceInfo>,
		“002”: Lease<InstanceInfo>,
		“003”: Lease<InstanceInfo>
	},
	“ServiceB”: {
		“001”: Lease<InstanceInfo>
	}
}

```



## Eureka 启动











## Eureka客户端注册



在两种情况下客户端会主动向服务端发送自己的注册信息

1.当客户端的instance信息发生改变时，Eureka-Client和Server端信息不一致时

2.当客户端刚刚启动的时候。



InstanceInfoReplicator是一个负责服务注册的线程任务， 有两个地方可以执行这个任务

1.定时线程，每40秒执行一次。

2.当instance的状态发生变更（除去DOWN这个状态）的时候，会有statusChangeListener 这个监听器监听到









## Eureka服务端接受注册



在这个eureka core的resources包下面，有一堆的resources，这些resource相当于是spring web mvc的controller，用来接收这个http请求的。

ApplicationResource的addInstance()方法，是接收post请求的服务实例的注册。



本地注册表

清理缓存

复制到同等服务节点上去



```java
@Override
public void register(final InstanceInfo info, final boolean isReplication) {
    // 租约的过期时间，默认90秒，也就是说当服务端超过90秒没有收到客户端的心跳，则主动剔除该节点。
    int leaseDuration = Lease.DEFAULT_DURATION_IN_SECS;
    if (info.getLeaseInfo() != null && info.getLeaseInfo().getDurationInSecs() > 0) {
        // 如果客户端自定义了，那么以客户端为准
        leaseDuration = info.getLeaseInfo().getDurationInSecs();
    }
    // 节点注册
    super.register(info, leaseDuration, isReplication);
    // 复制到同等服务节点上去
    replicateToPeers(Action.Register, info.getAppName(), info.getId(), info, null, isReplication);
}
```



```java
public void register(InstanceInfo registrant, int leaseDuration, boolean isReplication) {
    try {
        // 上只读锁
        read.lock();
        // 从本地MAP里面获取当前实例的信息。
        Map<String, Lease<InstanceInfo>> gMap = registry.get(registrant.getAppName());
        // 增加注册次数到监控信息里面去。
        REGISTER.increment(isReplication);
        if (gMap == null) {
            // 如果第一次进来，那么gMap为空，则创建一个ConcurrentHashMap放入到registry里面去
            final ConcurrentHashMap<String, Lease<InstanceInfo>> gNewMap = new ConcurrentHashMap<String, Lease<InstanceInfo>>();
            // putIfAbsent方法主要是在向ConcurrentHashMap中添加键—值对的时候，它会先判断该键值对是否已经存在。
            // 如果不存在（新的entry），那么会向map中添加该键值对，并返回null。
            // 如果已经存在，那么不会覆盖已有的值，直接返回已经存在的值。
            gMap = registry.putIfAbsent(registrant.getAppName(), gNewMap);
            if (gMap == null) {
                // 表明map中确实不存在，则设置gMap为最新创建的那个
                gMap = gNewMap;
            }
        }
        // 从MAP中查询已经存在的Lease信息 （比如第二次来）
        Lease<InstanceInfo> existingLease = gMap.get(registrant.getId());
        // 当Lease的对象不为空时。
        if (existingLease != null && (existingLease.getHolder() != null)) {
            // 当instance已经存在是，和客户端的instance的信息做比较，时间最新的那个，为有效instance信息
            Long existingLastDirtyTimestamp = existingLease.getHolder().getLastDirtyTimestamp(); // server
            Long registrationLastDirtyTimestamp = registrant.getLastDirtyTimestamp();   // client
            logger.debug("Existing lease found (existing={}, provided={}", existingLastDirtyTimestamp, registrationLastDirtyTimestamp);
            if (existingLastDirtyTimestamp > registrationLastDirtyTimestamp) {
                logger.warn("There is an existing lease and the existing lease's dirty timestamp {} is greater" +
                        " than the one that is being registered {}", existingLastDirtyTimestamp, registrationLastDirtyTimestamp);
                logger.warn("Using the existing instanceInfo instead of the new instanceInfo as the registrant");
                registrant = existingLease.getHolder();
            }
        } else {
            // 这里只有当existinglease不存在时，才会进来。 像那种恢复心跳，信息过期的，都不会进入这里。
            //  Eureka-Server的自我保护机制做的操作，为每分钟最大续约数+2 ，同时重新计算每分钟最小续约数
            synchronized (lock) {
                if (this.expectedNumberOfRenewsPerMin > 0) {
                    // Since the client wants to cancel it, reduce the threshold
                    // (1
                    // for 30 seconds, 2 for a minute)
                    this.expectedNumberOfRenewsPerMin = this.expectedNumberOfRenewsPerMin + 2;
                    this.numberOfRenewsPerMinThreshold =
                            (int) (this.expectedNumberOfRenewsPerMin * serverConfig.getRenewalPercentThreshold());
                }
            }
            logger.debug("No previous lease information found; it is new registration");
        }
        // 构建一个最新的Lease信息
        Lease<InstanceInfo> lease = new Lease<InstanceInfo>(registrant, leaseDuration);
        if (existingLease != null) {
            // 当原来存在Lease的信息时，设置他的serviceUpTimestamp, 保证服务开启的时间一直是第一次的那个
            lease.setServiceUpTimestamp(existingLease.getServiceUpTimestamp());
        }
        // 放入本地Map中
        gMap.put(registrant.getId(), lease);
        // 添加到最近的注册队列里面去，以时间戳作为Key， 名称作为value，主要是为了运维界面的统计数据。
        synchronized (recentRegisteredQueue) {
            recentRegisteredQueue.add(new Pair<Long, String>(
                    System.currentTimeMillis(),
                    registrant.getAppName() + "(" + registrant.getId() + ")"));
        }
        // This is where the initial state transfer of overridden status happens
        // 分析instanceStatus
        if (!InstanceStatus.UNKNOWN.equals(registrant.getOverriddenStatus())) {
            logger.debug("Found overridden status {} for instance {}. Checking to see if needs to be add to the "
                            + "overrides", registrant.getOverriddenStatus(), registrant.getId());
            if (!overriddenInstanceStatusMap.containsKey(registrant.getId())) {
                logger.info("Not found overridden id {} and hence adding it", registrant.getId());
                overriddenInstanceStatusMap.put(registrant.getId(), registrant.getOverriddenStatus());
            }
        }
        InstanceStatus overriddenStatusFromMap = overriddenInstanceStatusMap.get(registrant.getId());
        if (overriddenStatusFromMap != null) {
            logger.info("Storing overridden status {} from map", overriddenStatusFromMap);
            registrant.setOverriddenStatus(overriddenStatusFromMap);
        }
    
        // Set the status based on the overridden status rules
        InstanceStatus overriddenInstanceStatus = getOverriddenInstanceStatus(registrant, existingLease, isReplication);
        registrant.setStatusWithoutDirty(overriddenInstanceStatus);
    
        // If the lease is registered with UP status, set lease service up timestamp
        // 得到instanceStatus，判断是否是UP状态，
        if (InstanceStatus.UP.equals(registrant.getStatus())) {
            lease.serviceUp();
        }
        // 设置注册类型为添加
        registrant.setActionType(ActionType.ADDED);
        // 租约变更记录队列，记录了实例的每次变化， 用于注册信息的增量获取、
        recentlyChangedQueue.add(new RecentlyChangedItem(lease));
        registrant.setLastUpdatedTimestamp();
        // 清理缓存 ，传入的参数为key
        invalidateCache(registrant.getAppName(), registrant.getVIPAddress(), registrant.getSecureVipAddress());
        logger.info("Registered instance {}/{} with status {} (replication={})",
                registrant.getAppName(), registrant.getId(), registrant.getStatus(), isReplication);
    } finally {
        read.unlock();
    }
}
```



## 获取服务端注册信息

两种获取地方，启动和定时任务获取



启动获取

在客户端应用启动时，初始化DiscoverClient的时候，会主动去全量获取一次注册信息

DiscoveryClient  的  initScheduledTasks（）





定时器获取

registryFetchIntervalSeconds : 默认值为30秒 ，每30秒刷新一次，全量或者增量

DiscoveryClient  的 CacheRefreshThread（）







1.发起http请求，将服务端的客户端变化的信息拉取过来，如： register， cancle,  modify 有过这些操作的数据

2.上锁，防止某次调度网络请求时间过长，导致同一时间有多线程拉取到增量信息并发修改

3.将请求过来的增量数据和本地的数据做合并

4.计算hashCode

5.如果hashCode不一致，或者clientConfig.shouldLogDeltaDiff() = true 的话，则又会去服务端发起一次全量获取

6.数据合并

7.发布缓存刷新的事件

8.更新本地应用的状态



## 服务端返回注册信息

com.netflix.eureka.resources.ApplicationsResource

http://localhost:8080/v2/apps



### 全量获取

@GET

public Response getContainers

把服务端本地的CurrentHashMap里面存储的客户端信息，封装成Application实体，然后返回



### 增量获取

@Path("delta")

@GET

public Response getContainerDifferential

因为客户端本地已经有了缓存的Applications，所以再次向Eureka服务端抓取注册表的时候，走的是增量抓取的策略





Eureka服务端会在客户端发生变化时 如： 注册，下线，过期等操作的实例数据存入recentlyChangedQueue，

租约变化队列里面的数据默认保存3分钟，会有一个定时器每30秒清理一次获取到了这些变化的客户端信息，返回Eureka Clien 之后，通过集合合并，就可以得到最新的缓存数据了。

Eureka Clien对合并以后的注册表，会计算一个hash值，跟eureka server端的全量注册表的hash值进行一个比对；如果说不一样的话，说明本地注册表跟server端不一样了，此时就会重新从eureka server拉取全量的注册表到本地来更新到缓存里去





## Eureka缓存机制



### 缓存的更新

readOnlyCacheMap  只会30秒定时更新



readWriteCacheMap  180秒定时，或者服务下线， 过期，注册，状态变更的时候会清除





readWriteCacheMap ： 此处存放的是google的gauva缓存， 当服务下线，过期，注册，状态变更，都会来清除这个缓存里面的数据。 然后通过CacheLoader进行缓存加载，在进行readWriteCacheMap.get(key)的时候，首先看这个缓存里面有没有该数据，如果没有则通过CacheLoader的load方法去加载，加载成功之后将数据放入缓存，同时返回数据

readOnlyCacheMap ： 这是一个JVM的CurrentHashMap只读缓存，这个主要是为了供客户端获取注册信息时使用，其缓存更新，依赖于定时器的更新，通过和readWriteCacheMap 的值做对比，如果数据不一致，则以readWriteCacheMap 的数据为准。



### 缓存的读取



useReadOnlyCache ： shouldUseReadOnlyResponseCache ，可以配置是否使用只读缓存，默认是true

readWriteCacheMap.get(key) : 这个使用的是gauva 的缓存机制，如果当前的缓存里面这个key没有，那么

会直接调用CacheLoader.load()方法，从最上面的代码可以看到， load方法，主要是执行了generatePayload()



payload = getPayLoad(key, registry.getApplicationDeltas());

payload = getPayLoad(key, registry.getApplications());





## Eureka 心跳



### 心跳发送

DiscoverClient这个类初始化的时候，会初始化定期任务HeartbeatThread，每30秒执行一次，用来发送心跳

启动一个线程，然后线程执行renew()方法， 最终发送心跳给Eureka-Server

接口地址： apps/ + appName + /' + id ，





### 心跳接受

服务端  InstanceResource   的  renewLease方法 接受续约



如果接口返回值为404，就是说不存在，从来没有注册过，那么重新走注册流程



在调用续约的方法之后，Eureka Server 会对请求过来的lastDirtyTimestamp和本地的做对比，如果

请求lastDirtyTimestamp>本地的时间，则认为当前实例是无效的，返回404错误，客户端重新发起注册。

如果是集群同步请求，本地的时间，大于其他Eureka Server传过来的时间，则返回 “冲突” 这个状态回去，

以本地的时间大的为准，注意是集群同步请求，如果是客户端传过的，是不会有这个规则的。



PeerAwareInstanceRegistryImpl    集群续约

AbstractInstanceRegistry		客户端续约



## Eureka 主动下线



取消定时任务（心跳，缓存刷新等）

设置实例的状态为DOWN

发送HTTP请求执行下线



PeerAwareInstanceRegistryImpl 的父类AbstractInstanceRegistry中是取消下线的主要逻辑。



InstanceResource中，调用注册表的cancelLease()方法，调用父类的canel()方法，interlCancel()方法

com.netflix.eureka.registry.AbstractInstanceRegistry#internalCancel

将服务实例从eureka server的map结构的注册表中移除掉

最最核心的是调用了Lease的cancel()方法，里面保存了一个evictionTimestamp，就是服务实例被清理掉，服务实例下线的时间戳

将服务实例放入最近变化的队列中去，让所有的eureka client下一次拉取增量注册表的时候，可以拉取到这个服务实例下线的这么一个变化

服务实例变更过了，必须将之前的缓存都清理掉，从readWriteCacheMap中清理掉

定时的任务，每隔30秒，将readWriteCacheMap和readOnlyCacheMap进行一个同步

下次所有的eureka client来拉取增量注册表的时候，都会发现readOnlyCacheMap里没有，会找readWriteCacheMap也会发现没有，然后就会从注册表里抓取增量注册表，此时就会将上面那个recentCHangedQuuee中的记录返回





## 自动故障感知机制,自动摘除

EvictionTask 

1. 算出延迟补偿时间.定时器自身所在的JVM发送GC或者linux服务器时间延迟导致时间变长.避免EvictionTask两次调度的时间超过了默认设置的60s.补偿时间的机制.
2. 遍历注册表中所有的服务实例，然后调用Lease的isExpired()方法，来判断当前这个服务实例的租约是否过期了，是否失效了，服务实例故障了，如果是故障的服务实例，加入一个列表.
3. 





additionalLeaseMs 延迟补偿时间计算方法:  

additionalLeaseMs  =  过期任务定时器调度执行时间  -  currentTimeMillis  



lastUpdateTimestamp 心跳续约时间的计算方法:

lastUpdateTimestamp = currentTimeMillis + 90s

每一次续约过来时,会把当前时间加90秒,(90秒内没收到一个client的服务续约) ,由此得出lastUpdateTimestamp.



evictionTimestamp  判断续约Lease的isExpired()的时间计算方法

evictionTimestamp   =  lastUpdateTimestamp+90秒+延迟补偿时间.

当前时间是否大于上一次心跳时间也就是



这里有个错误，因为续约的时候，更新这个时间的时候，加上了duration ， 但是在最终做判断的时候lastUpdateTimestamp + duration + additionalLeaseMs ， 这个地方还加了一遍，也就导致了，当前时间必须要大于实际最后更新时间180秒，才会认为他过期.



