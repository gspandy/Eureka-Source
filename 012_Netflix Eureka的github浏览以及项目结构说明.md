 



 

https://github.com/spring-cloud/spring-cloud-netflix

https://github.com/Netflix/eureka

 

spring cloud eureka，就是对netflix的eureka做了一层封装，加了一些注解和其他的机制来跟spring boot进行配合，netflix eureka的源码，包含了eureka所有的服务注册和发现的机制

 

spring cloud Edgware.SR3对应的是netflix eureka的1.7.2的版本

 

（1）**eureka-client**：这个就是指的eureka的客户端，注册到eureka上面去的一个服务，就是一个eureka client，无论是你要注册，还是要发现别的服务，无论是服务提供者还是服务消费者，都是一个eureka客户端。

（2）**eureka-core**：这个就是指的eureka的服务端，其实就是eureka的注册中心

（3）eureka-resources：这个是基于jsp开发的eureka控制台，web页面，可以看到各种注册服务

（4）eureka-server：这是把eureka-client、eureka-core、eureka-resources打包成了一个war包，eureka-server自己本身也是一个eureka-client，同时也是注册中心，同时也提供eureka控制台。真正的使用的注册中心

（5）eureka-examples：eureka使用的例子

（6）eureka-test-utils：eureka的单元测试工具类

 

**eureka-client**，**eureka-core**         两个重要的包



eureka-server先启动注册中心的

 

 