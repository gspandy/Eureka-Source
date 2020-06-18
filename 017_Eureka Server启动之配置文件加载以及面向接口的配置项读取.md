 

加载eureka-server.properties中的配置

 

EurekaServerConfig，这是个接口，这里面有一堆getXXX()的方法，包含了eureka server需要使用的所有的配置，都可以通过这个接口来获取

 

想象一下，eureka-sever.properties文件里，都是一个一个的key=value的很多的配置项，肯定是将这些key-value格式的配置项加载到内存的Properties对象去存放，Map。一般来说，如果让我们自己来设计这个读取properties文件的配置的代码，也许我们就是做到将配置加载到Properties对象中就结束了。

 

EurekaServerConfig，代表了eureka-server需要的所有的配置项，通过接口定义了大量的方法，让你可以从这里获取所有你需要的配置

 

Properties prop = new Properties();

prop.load(inputStream)

 

如果要获取一个配置项，提供一个方法，get(String key)，return prop.get(key);。

 

在外面如果要获取一个配置项，可能会这样子，在一个常量类里搞一堆配置的常量，比如说下面：

 

public class Configs {

 

public static final String REMOTE_REGION_TOTAL_CONNECTIONS_PER_HOST = “remote.region.total.connections.per.host”;

 

}

 

get(Configs.REMOTE_REGION_TOTAL_CONNECTIONS_PER_HOST)

 

通过这种方式去获取你这个框架配置的各种配置项

 

这种方式也不错，因为比如说流行的spark源码里面，就是大量的基于这种常量的方式来获取配置属性的

 

但是eureka-server这里，使用了另外一种思想，人家没有用大量的常量，而是针对配置定义了一个接口，接口里通过方法暴露了大量的配置项获取的方法，你呢直接通过这个接口来获取你需要的配置项，即可。

 

DefaultEurekaServerConfig，是上面那个接口的实现类，创建实例的时候，会执行一个init()方法，在这个方法中，就会完成eureka-server.properties文件中的配置项的加载，EUREKA_PROPS_FILE，对应着要加载的eureka的配置文件的名字。

 

eureka server的配置文件的默认的名称，就是eureka-server

 

ConfigurationManager，是个单例，负责管理所有的配置的，ConfigurationManager是属于netfilx config开源项目的，不是属于eureka项目的源码，所以我们大概看一下就可以了，不要去深究了。eureka-server跟.properties给拼接起来了，拼接成一个eureka-server.properties，代表了eureka server的配置文件的名称。

 

将eureka-sesrver.properties中的配置，加载到了Properties对象中去；然后会加载eureka-server-环境.properties中的配置，加载到另外一个Properties中，覆盖之前那个老的Properties中的属性。

 

将加载出来的Properties中的配置项都放到ConfigurationManager中去，由这个ConfigurationManager来管理

 

比如说eureka-server那个工程里，就有一个src/main/resources/eureka-server.properties文件，只不过里面是空的，全部都用了默认的配置

 

DefaultEurekaServerConfig.init()方法中，会将eureka-server.properties文件中的配置加载出来，都放到ConfdigurationManager中去，然后在DefaultEurekaServerConfig的各种获取配置项的方法中，配置项的名字是在各个方法中硬编码的，是从一个DynamicPropertyFactory里面去获取的，你可以认为DynamicPropertyFactory是从ConfigurationManager那儿来的，因为ConfigurationManager中都包含了加载出来的配置了，所以DynamicPropertyFactory里，也可以获取到所有的配置项

 

在从DynamicPropertyFactory中获取配置项的时候，如果你没配置，那么就用默认值，全部都给你弄好了各个配置项的默认值，相当于所有的配置项的默认值，在DefaultEurekaServerConfig的各个方法中，都可以看到，如果你没配置，那么就用这里的默认值就可以了

 

加载eureka-server.properties的过程：

 

（1）创建了一个DefaultEurekaServerConfig对象

（2）创建DefaultEurekaServerConfig对象的时候，在里面会有一个init方法

（3）先是将eureka-server.properties中的配置加载到了一个Properties对象中，然后将Properties对象中的配置放到ConfigurationManager中去，此时ConfigurationManager中去就有了所有的配置了

（4）然后DefaultEurekaServerConfig提供的获取配置项的各个方法，都是通过硬编码的配置项名称，从DynamicPropertyFactory中获取配置项的值，DynamicPropertyFactory是从ConfigurationManager那儿来的，所以也包含了所有配置项的值

（5）在获取配置项的时候，如果没有配置，那么就会有默认的值，全部属性都是有默认值的

 

我希望大家，这节课过后，顺着上面的思路，创建DefaultEurekaServerConfig对象开始，把上面的思路给捋一遍，自己看一遍源码

 

重点体会一个东西，eureka-server的配置管理的机制和思想是什么？不是通过一大堆的常量类获取配置项的，提供了一个获取配置的接口，接口里包含大量的方法，每个方法可以获取一个配置，获取配置的方法里对配置的名称进行硬编码了

 

 





