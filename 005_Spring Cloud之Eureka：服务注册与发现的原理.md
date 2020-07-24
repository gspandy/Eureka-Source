

Eureka客户端从服务器获取注册表信息，并将其本地缓存。之后，客户端使用该信息来查找其他服务。通过获取上一个获取周期与当前获取周期之间的增量更新，定期（每30秒）更新此信息。增量信息在服务器中保留的时间更长（大约3分钟），因此增量获取可能会再次返回相同的实例。Eureka客户端会自动处理重复信息。

获得增量之后，Eureka客户端通过比较服务器返回的实例计数来与服务器协调信息，如果信息由于某种原因不匹配，则会再次获取整个注册表信息。

来自Eureka客户端的所有操作可能需要一些时间才能反映在Eureka服务器以及随后的其他Eureka客户端中。这是因为在eureka服务器上缓存了有效负载，该负载会定期刷新以反映新信息。尤里卡客户还定期获取增量。因此，更改最多可能需要2分钟才能传播到所有Eureka客户端。

尤里卡客户需要通过每30秒发送一次心跳来续订租约。续订通知Eureka服务器该实例仍在运行。如果服务器在90秒钟内未看到续订，则会将实例从其注册表中删除。建议不要更改更新间隔，因为服务器使用该信息来确定客户端与服务器之间的通信是否存在广泛传播的问题。

Eureka客户端在关闭时向Eureka服务器发送取消请求。这样会将实例从服务器的实例注册表中删除，从而有效地消除了实例的流量。

当Eureka客户端关闭时，将执行此操作，并且应用程序应确保在关闭过程中调用以下命令。

```
     DiscoveryManager.getInstance().shutdownComponent()
```



Eureka客户端尝试与同一区域中的Eureka Server通信。如果在与服务器通信时遇到问题，或者服务器不在同一区域中，则客户端将故障转移到其他区域中的服务器。

服务器开始接收流量后，在服务器上执行的所有[操作](https://github.com/Netflix/eureka/wiki/Understanding-eureka-client-server-communication)都将复制到服务器知道的所有对等节点。如果某个操作由于某种原因失败，则该信息将在下一个检测信号上进行协调，该检测信号也将在服务器之间复制。

当Eureka服务器启动时，它将尝试从相邻节点获取所有实例注册表信息。如果从节点获取信息有问题，则服务器会在放弃之前尝试所有对等节点。如果服务器能够成功获取所有实例，则服务器将根据该信息设置应接收的续订阈值。如果有任何时间，续订次数均低于为此值配置的百分比（在15分钟内低于85％），则服务器将停止使实例到期以保护当前实例注册表信息。

在Netflix中，以上保护措施称为**自我保护**模式，主要用于在一组客户端和Eureka Server之间存在网络分区的情况下的保护措施。在这些情况下，服务器尝试保护它已经拥有的信息。在大规模中断的情况下，可能会出现某些情况，这可能导致客户端获取不再存在的实例。客户端必须确保它们可以抵抗eureka服务器返回不存在或无响应的实例。在这些情况下，最好的保护是快速超时并尝试其他服务器。

在这种情况下，如果服务器无法从相邻节点获取注册表信息，它将等待几分钟（5分钟），以便客户端可以注册其信息。服务器通过将流量仅偏向一组实例并导致容量问题，努力不向客户端提供部分信息。

Eureka服务器使用[此处](https://github.com/Netflix/eureka/wiki/Understanding-eureka-client-server-communication)所述的Eureka客户端与服务器之间使用的相同机制相互通信。















**appID**是应用程序的名称，**instanceID**是与**实例**关联的唯一ID。在AWS云中，instanceID是**实例的实例ID**，在其他数据中心中，instanceID是**实例**的**主机名**。

对于JSON / XML，提供的内容类型必须为**application / xml**或**application / json**。



| **Operation**                                                | **HTTP action**                                              | **Description**                                              |
| :----------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Register new application instance                            | POST /eureka/v2/apps/**appID**                               | Input: JSON/XML payload HTTP Code: 204 on success            |
| De-register application instance                             | DELETE /eureka/v2/apps/**appID**/**instanceID**              | HTTP Code: 200 on success                                    |
| Send application instance heartbeat                          | PUT /eureka/v2/apps/**appID**/**instanceID**                 | HTTP Code: * 200 on success * 404 if **instanceID** doesn’t exist |
| Query for all instances                                      | GET /eureka/v2/apps                                          | HTTP Code: 200 on success Output: JSON/XML                   |
| Query for all **appID** instances                            | GET /eureka/v2/apps/**appID**                                | HTTP Code: 200 on success Output: JSON/XML                   |
| Query for a specific **appID**/**instanceID**                | GET /eureka/v2/apps/**appID**/**instanceID**                 | HTTP Code: 200 on success Output: JSON/XML                   |
| Query for a specific **instanceID**                          | GET /eureka/v2/instances/**instanceID**                      | HTTP Code: 200 on success Output: JSON/XML                   |
| Take instance out of service                                 | PUT /eureka/v2/apps/**appID**/**instanceID**/status?value=OUT_OF_SERVICE | HTTP Code: * 200 on success * 500 on failure                 |
| Move instance back into service (remove override)            | DELETE /eureka/v2/apps/**appID**/**instanceID**/status?value=UP (The value=UP is optional, it is used as a suggestion for the fallback status due to removal of the override) | HTTP Code: * 200 on success * 500 on failure                 |
| Update metadata                                              | PUT /eureka/v2/apps/**appID**/**instanceID**/metadata?key=value | HTTP Code: * 200 on success * 500 on failure                 |
| Query for all instances under a particular **vip address**   | GET /eureka/v2/vips/**vipAddress**                           | * HTTP Code: 200 on success Output: JSON/XML * 404 if the **vipAddress** does not exist. |
| Query for all instances under a particular **secure vip address** | GET /eureka/v2/svips/**svipAddress**                         | * HTTP Code: 200 on success Output: JSON/XML * 404 if the **svipAddress** does not exist. |











## 对等之间的网络中断期间会发生什么？

如果对等点之间发生网络中断，则可能会发生以下情况。

- 对等方之间的心跳复制可能会失败，并且服务器会检测到这种情况，并进入保护当前状态的自保留模式。
- 注册可能发生在孤立的服务器上，有些客户端可能会反映新的注册，而其他客户端则可能没有。

网络连接恢复到稳定状态后，情况会自动更正。当对等方能够正常通信时，注册信息会自动传输到没有对等方的服务器。最重要的是，在网络中断期间，服务器将尝试尽可能地具有弹性，但是在此期间，客户端可能会对服务器有不同的看法。

## 为什么不使用HA代理进行负载平衡？

在AWS云中，实例由于其内在原因而进出流量。ASG可以根据流量扩展实例或销毁实例。无论我们是否使用HAProxy，这里面临的挑战是处理这种动态特性。即使您使用HAProxy，也需要对进出实例进行教育，这正是Eureka所做的。

我之所以想在中间层使用代理的原因之一是当您需要粘性会话时。如果需要，HAProxy可以补充Eureka。但是，由于在我们的场景中大多数中间层服务都是非粘性的，因此我们没有理由通过避免网络跳来通过代理进行访问。这样做的另一个积极的副作用是，它使我们的客户对Eureka服务器中断具有相当的弹性，因为客户拥有所有信息来联系需要与之交谈的服务。通过HA Proxy的一个缺点是，您不能对代理中断有弹性。

## 为什么不将Curator / Zookeeper用作服务注册表？

Zookeeper和Eureka在某些方面存在某些重叠，尤其是在复制注册表信息方面。尤里卡可以使用zookeeper缓存注册表信息并进行复制，但是复制只是尤里卡提供的一小部分。

除了复制，Eureka还处理其他各种事情：

- REST端点处理注册，续订，到期和取消。
- 使实例信息保持最新状态，以应对复杂的EIP绑定，部署回滚和弹性扩展。
- 能够抵抗客户端和服务器之间以及同级之间的网络中断。

Zookeeper的功能通过领导者选举，有序更新，分布式同步及其一致性保证（仲裁）而脱颖而出。

除了复制注册表之外，以上所有内容均不适用于Eureka，以证明我们必须处理以下复杂问题的其他依赖关系：

- 您现在必须找到一种与Eureka类似的将EIP分配给Zookeeper的方法。
- 当Zookeeper失败时处理失败。

而且，Eureka经过精心打造，没有任何外部组件的严格依赖。

- 大多数服务依靠Eureka进行自我引导。
- 降低复杂度。
- 避免出现另一个故障点。

















