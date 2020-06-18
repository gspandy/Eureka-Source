剩余的初始化的步骤

 

（1）构造了一个东西：PeerAwareInstanceRegistry

 

从名字上我们先来猜测一下，这个是个什么东东？看源码，英文还是要有点功底，源码是用英文写的，人家是老外写的，人家对英文的使用，就跟我们对汉文的使用是一样的。人家对类名、变量名、方法名，全都是通过英文名字来表述出对应的意思的。所以写的好的代码，光是通过上面3个名字，就能让你看懂了，可读性极强。注释不怎么写都可以。

 

PeerAware，可以识别eureka server集群的：peer，多个同样的东西组成的一个集群，peers集群，peer就是集群中的一个实例

 

InstanceRegistry：实例注册，服务实例注册，注册表，这个里面放了所有的注册到这个eureka server上来的服务实例，就是一个服务实例的注册表

 

PeerAwareInstanceRegistry：可以感知eureka server集群的服务实例注册表，eureka client（作为服务实例）过来注册的注册表，而且这个注册表是可以感知到eureka server集群的。假如有一个eureka server集群的话，这里包含了其他的eureka server中的服务实例注册表的信息的。

 

抓大放小：抓住主流程、主架构、主要的机制，放掉很多细节性的代码

 

连蒙带猜：刚开始你对这个源码没有一个整体上的认识，所以很多东西你开始拿捏不稳，猜测一下

 

看源码的技巧，为什么说自己看不懂源码？因为其实你们完全掌握错了看源码的思路、方法还有技巧。像我刚才那样看，直接懵了，一个源码，绝对不是那样看的。一开始，必须是按照一个最基本的主流程，主流程看明白了之后，你回过头来，再把之前看过的每个步骤的细节，再扣进去看，看一些细节，那个时候你才能看懂源码的细节。

 

（2）构造了一个东西：PeerEurekaNodes

 

猜，PeerEurekaNodes，代表了eureka server集群，peers大概来说多个相同的实例组成的一个集群，peer就是peers集群中的一个实例，PeerEurekaNodes，大概来说，猜测，应该是代表的是eureka server集群

 

连蒙带猜的时候，可以根据人家的类的注释，来猜测一下他的大概的用途，写的不一定很清晰，大体上可以猜测出来的

 

（3）构造了一个东西：EurekaServerContext

 

将上面构造好的所有的东西，都一起来构造一个EurekaServerContext，代表了当前这个eureka server的一个服务器上下文，包含了服务器需要的所有的东西。将这个东西放在了一个holder中，以后谁如果要使用这个EurekaServerContext，直接从这个holder中获取就可以了。这个也是一个比较常见的用法，就是将初始化好的一些东西，放在一个holder中，然后后面的话呢，整个系统运行期间，谁都可以来获取，在任何地方任何时间，谁都可以获取这个上下文，从里面获取自己需要的一些组件。

 

（4）EurekaServerContext.initialize()

 

peerEurekaNodes.start();

 

这里呢，就是将eureka server集群给启动起来，这里干的事情，我们猜测一下，就是更新一下eureka server集群的信息，让当前的eureka server感知到所有的其他的eureka server。然后搞一个定时调度任务，就一个后台线程，每隔一定的时间，更新eureka server集群的信息。

 

registry.init(peerEurekaNodes);

 

猜测一下，基于eureka server集群的信息，来初始化注册表，大概猜测，肯定是将eureka server集群中所有的eureka server的注册表的信息，都抓取过来，放到自己本地的注册表里去，多事跟eureka server集群之间的注册表信息互换有关联的

 

（5）registry.syncUp();

 

从相邻的一个eureka server节点拷贝注册表的信息，如果拷贝失败，就找下一个

 

鱼，没有教你渔

 

（6）EurekaMonitors.registerAllStats();

 

跟eureka自身的监控机制相关联的

 

读源码，千万不要有强迫症， 很多时候，刚开始看源码的时候，要允许自己对很多细节都不太清楚，但是能大体把握住大的流程就ok了