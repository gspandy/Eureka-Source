 

默认情况下，所有的服务，比如服务A和服务B，都会自动给eureka注册中心同步心跳，续约，每隔一段时间发送心跳，如果某个服务实例挂了，那么注册中心一段时间内没有感知到那个服务的心跳，就会把那个服务下线



如果实现一个服务的健康检查的机制，自己来检查服务是否宕机，

比如说，如果底层依赖的MQ、数据库挂了，就宣布自己挂了，通知注册中心

 

在服务中加入以下依赖：

 

```
<dependency>

	<groupId>org.springframework.boot</groupId>

	<artifactId>spring-boot-starter-actuator</artifactId>

	<version>1.5.13.RELEASE</version>

</dependency>
```

 

http://localhost:8080/health，可以看到服务的健康状态

 

正常情况下就这样就可以了，但是有一个问题，就是可能这个服务依赖的其他基础设施，比如redis、mysql、mq，都挂掉了，或者底层的基础服务挂掉了，此时这个服务已经不可用了，那么这个服务就可以认定自己是不可用了

 

所以可以自己实现一个健康检查器，就是自己检查自己依赖的基础设施，或者是基础服务，是否挂掉了，来决定自己是否还是健康的，并执行eureka下线逻辑

 

```java
@Component
public class ServiceAHealthIndicator implements HealthIndicator {


@Override
public Health health() {

// 这里可以通过返回UP或者DOWN来指示服务的状态

	return new Health.Builder(Status.UP).build();

	}
}

 
@Component
public class ServiceAHealthCheckHandler implements HealthCheckHandler {


@Autowired
private ServiceAHealthIndicator indicator;

	public InstanceStatus getStatus(InstanceStatus currentStatus) {

		Status status = indicator.health().getStatus();

// 根据这个status，可以决定这里返回什么

		return InstanceStatus.UP;

	}
}	
```

 

eureka client里面会有一个定时器，不断调用那个HealthCheckHandler的getStatus()方法，然后检查当前这个服务实例的状态，如果状态变化了，就会通知eureka注册中心。如果服务实例挂掉了，那么eureka注册中心就会感知到，然后下线这个服务实例。

 

不过其实一般很少自己去实现这个健康检查，在大规模的部署中，每个服务都很复杂，不可能都这样去搞一堆健康检查的。

大部分情况下，我们就是会对外暴露一个/health接口，然后专门外部来定时调用各个服务的/health接口来判断当前这个服务能够调通。

 

eureka默认是client通过心跳机制跟eureka注册中心保持心跳通信，如果心跳不及时或者没有心跳了，那么就说明那个服务挂了，然后eureka注册中心就会摘除这个服务实例。这个机制就足够了。

 