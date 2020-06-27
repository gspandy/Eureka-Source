 

加载eureka-server.properties中的配置

 





 

 

平常如果要获取一个配置项，在一个常量类里搞一堆配置的常量，比如说下面：

```
public class Configs {

		public static final String REMOTE_REGION_TOTAL_CONNECTIONS_PER_HOST = “remote.region.total.connections.per.host”;

}

get(Configs.REMOTE_REGION_TOTAL_CONNECTIONS_PER_HOST)

```

 

但eureka-server这里，针对配置定义了一个接口，接口里通过方法暴露了大量的配置项获取的方法，直接通过这个接口来获取需要的配置项即可。

 

com.netflix.eureka.DefaultEurekaServerConfig#DefaultEurekaServerConfig()

com.netflix.eureka.DefaultEurekaServerConfig#init



EurekaServerConfig，代表了eureka-server需要的所有的配置项，通过接口定义了大量的方法，让你可以从这里获取所有你需要的配置

DefaultEurekaServerConfig，EurekaServerConfig接口的实现类，创建实例的时候，会执行一个init()方法，在这个方法中，就会完成eureka-server.properties文件中的配置项的加载，EUREKA_PROPS_FILE，对应着要加载的eureka的配置文件的名字。

 

```
private static final DynamicStringProperty EUREKA_PROPS_FILE = 
DynamicPropertyFactory
        .getInstance().getStringProperty(
        "eureka.server.props",
		"eureka-server"
		);
```

 

ConfigurationManager，是个单例，负责管理所有的配置的，ConfigurationManager是属于netfilx config开源项目的，不是属于eureka项目的源码

 

将eureka-sesrver.properties中的配置，加载到了Properties对象中去；

然后加载eureka-server-环境.properties中的配置，加载到另外一个Properties中，覆盖之前那个老的Properties中的属性。

将加载出来的Properties中的配置项都放到ConfigurationManager中去，由这个ConfigurationManager来管理

 

比如说eureka-server那个工程里，就有一个src/main/resources/eureka-server.properties文件，只不过里面是空的，全部都用了默认的配置

 

DefaultEurekaServerConfig.init()方法中，会将eureka-server.properties文件中的配置加载出来，都放到ConfdigurationManager中去，然后在DefaultEurekaServerConfig的各种获取配置项的方法中，配置项的名字是在各个方法中硬编码的，是从一个DynamicPropertyFactory里面去获取的，

DynamicPropertyFactory是从ConfigurationManager那儿来的，

因为ConfigurationManager中都包含了加载出来的配置了，

所以DynamicPropertyFactory里，也可以获取到所有的配置项

 

在从DynamicPropertyFactory中获取配置项的时候，如果没配置，那么就用默认值，预设了各个配置项的默认值，相当于所有的配置项的默认值，在DefaultEurekaServerConfig的各个方法中，如果没配置，那么就用默认值

 



加载eureka-server.properties的过程：

（1）创建了一个DefaultEurekaServerConfig对象

（2）创建DefaultEurekaServerConfig对象的时候，执行init方法

（3）先将eureka-server.properties中的配置加载到了一个Properties对象中，然后将Properties对象中的配置放到ConfigurationManager中去，此时ConfigurationManager中去就有了所有的配置了

（4）然后DefaultEurekaServerConfig提供的获取配置项的各个方法，通过硬编码的配置项名称，从DynamicPropertyFactory中获取配置项的值，DynamicPropertyFactory是从ConfigurationManager那儿来的，所以也包含了所有配置项的值

（5）在获取配置项的时候，如果没有配置，那么就会有默认的值，全部属性都是有默认值的



 

 





