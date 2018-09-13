## Eureka介绍 {#autoid-1-0-1}

Eureka是一个基于REST\(Representational State Transfer\)的服务，主要用于AWS cloud， 提供服务定位\(locating services\)、负载均衡\(load balancing\)、故障转移\(failover of middle-tier servers\)。我们把它叫做**Eureka Server**. Eureka也提供了基于Java的客户端组件，**Eureka Client**,内置的负载均衡器可以实现基本的round-robin负载均衡能力。

### Eureka架构 {#autoid-2-5-0}

![](https://raw.githubusercontent.com/Netflix/eureka/master/images/eureka_architecture.png)

## 一 Eureka服务治理体系

### 1.1 服务治理

服务治理是微服务架构中最为核心和基础的模块，它主要用来实现各个微服务实例的自动化注册和发现。

Spring Cloud Eureka是Spring Cloud Netflix微服务套件中的一部分，它基于Netflix Eureka做了二次封装。主要负责完成微服务架构中的服务治理功能。

### **Eureka服务治理体系如下：**

![](/assets/import.png)

### 1.2 服务注册

在服务治理框架中，通常都会构建一个注册中心，每个服务单元向注册中心登记自己提供的服务，包括服务的主机与端口号、服务版本号、通讯协议等一些附加信息。注册中心按照服务名分类组织服务清单，同时还需要以心跳检测的方式去监测清单中的服务是否可用，若不可用需要从服务清单中剔除，以达到排除故障服务的效果。

### 1.3 服务发现

在服务治理框架下，服务间的调用不再通过指定具体的实例地址来实现，而是通过服务名发起请求调用实现。服务调用方通过服务名从服务注册中心的服务清单中获取服务实例的列表清单，通过指定的负载均衡策略取出一个服务实例位置来进行服务调用。

### Eureka服务端

```
Eureka服务端，即服务注册中心。它同其他服务注册中心一样，支持高可用配置。依托于强一致性提供良好的服务实例可用性，可以应对多种不同的故障场景。
```

Eureka服务端支持集群模式部署，当集群中有分片发生故障的时候，Eureka会自动转入自我保护模式。它允许在分片发生故障的时候继续提供服务的发现和注册，当故障分配恢复时，集群中的其他分片会把他们的状态再次同步回来。集群中的的不同服务注册中心通过异步模式互相复制各自的状态，这也意味着在给定的时间点每个实例关于所有服务的状态可能存在不一致的现象。

### Eureka客户端

Eureka客户端，主要处理服务的注册和发现。客户端服务通过注册和参数配置的方式，嵌入在客户端应用程序的代码中。在应用程序启动时，Eureka客户端向服务注册中心注册自身提供的服务，并周期性的发送心跳来更新它的服务租约。同时，他也能从服务端查询当前注册的服务信息并把它们缓存到本地并周期行的刷新服务状态。

### 高可用服务注册中心

#### 1 高可用服务注册中心的概念

考虑到发生故障的情况，服务注册中心发生故障必将会造成整个系统的瘫痪，因此需要保证服务注册中心的高可用。

Eureka Server在设计的时候就考虑了高可用设计，在Eureka服务治理设计中，所有节点既是服务的提供方，也是服务的消费方，服务注册中心也不例外。

Eureka Server的高可用实际上就是将自己做为服务向其他服务注册中心注册自己，这样就可以形成一组互相注册的服务注册中心，以实现服务清单的互相同步，达到高可用的效果。

#### 2 构建服务注册中心集群

Eureka Server的同步遵循着一个非常简单的原则：只要有一条边将节点连接，就可以进行信息传播与同步。可以采用两两注册的方式实现集群中节点完全对等的效果，实现最高可用性集群，任何一台注册中心故障都不会影响服务的注册与发现

在同一台电脑下可模拟如下：

（1）创建application-peer1.properties

server.port=1111

eureka.instance.hostname=master

eureka.client.register-with-eureka=false

eureka.client.fetch-registry=false

eureka.instance.preferIpAddress=true

eureka.server.enableSelfPreservation=false

eureka.client.serviceUrl.defaultZone=[http://backup1:1112/eureka/,http://backup2:1113/eureka/](http://backup1:1112/eureka/,http://backup2:1113/eureka/)

（2）创建application-peer2.properties

server.port=1112

eureka.instance.hostname=backup1

eureka.client.register-with-eureka=false

eureka.client.fetch-registry=false

eureka.instance.preferIpAddress=true

eureka.server.enableSelfPreservation=false

eureka.client.serviceUrl.defaultZone=[http://master:1111/eureka/,http://backup2:1113/eureka/](http://master:1111/eureka/,http://backup2:1113/eureka/)

（3）创建application-peer3.properties

server.port=1113

eureka.instance.hostname=backup2

eureka.client.register-with-eureka=false

eureka.client.fetch-registry=false

eureka.instance.preferIpAddress=true

eureka.server.enableSelfPreservation=false

eureka.client.serviceUrl.defaultZone=[http://master:1111/eureka/,http://backup1:1112/eureka/](http://master:1111/eureka/,http://backup1:1112/eureka/)

\(4\) 在hosts文件中增加如下配置

127.0.0.1 master

127.0.0.1 backup1

127.0.0.1 backup2

### 3.6 失效剔除

有些时候，我们的服务实例并不一定会正常下线，可能由于内存溢出、网络故障等原因使服务不能正常运作。而服务注册中心并未收到“服务下线”的请求，为了从服务列表中将这些无法提供服务的实例剔除，Eureka Server在启动的时候会创建一个定时任务，默认每隔一段时间（默认为60秒）将当前清单中超时（默认为90秒）没有续约的服务剔除出去。

### 3.7 自我保护

服务注册到Eureka Server后，会维护一个心跳连接，告诉Eureka Server自己还活着。Eureka Server在运行期间会统计心跳失败的比例在15分钟以之内是否低于85%，如果出现低于的情况，Eureka Server会将当前实例注册信息保护起来，让这些实例不会过期。这样做会使客户端很容易拿到实际已经不存在的服务实例，会出现调用失败的情况。因此客户端要有容错机制，比如请求重试、断路器。

以下是自我保护相关的属性：

eureka.server.enableSelfPreservation=true. 可以设置改参数值为false，以确保注册中心将不可用的实例删除

### 3.8 region（地域）与zone（可用区）

region和zone（或者Availability Zone）均是AWS的概念。在非AWS环境下，我们可以简单地将region理解为地域，zone理解成机房。一个region可以包含多个zone，可以理解为一个地域内的多个不同的机房。不同地域的距离很远，一个地域的不同zone间距离往往较近，也可能在同一个机房内。

region可以通过配置文件进行配置，如果不配置，会默认使用us-east-1。同样Zone也可以配置，如果不配置，会默认使用defaultZone。

Eureka Server通过eureka.client.serviceUrl.defaultZone属性设置Eureka的服务注册中心的位置。



## 四 服务提供者

### 4.1 服务注册

服务提供者在启动的时候会通过REST请求的方式将自己注册到Eureka Server上，同时带上自身服务的一些元数据信息。Eureka Server接收到这个Rest请求之后，将元数据信息存储在一个双层结构的Map中，其中第一层的key是服务名。第二层的key 是具体服务的实例名。

在服务注册时，需要确认一下eureka.client.register-with-eureka=true参数是否正确，该值默认为true。若设置为fasle将不会启动注册操作。

### 4.2 服务同步

从eureka服务治理体系架构图中可以看到，不同的服务提供者可以注册在不同的服务注册中心上，它们的信息被不同的服务注册中心维护。

此时，由于多个服务注册中心互相注册为服务，当服务提供者发送注册请求到一个服务注册中心时，它会将该请求转发给集群中相连的其他注册中心，从而实现服务注册中心之间的服务同步。通过服务同步，提供者的服务信息就可以通过集群中的任意一个服务注册中心获得。

### 4.3 服务续约

在注册服务之后，服务提供者会维护一个心跳用来持续高速Eureka Server，“我还在持续提供服务”，否则Eureka Server的剔除任务会将该服务实例从服务列表中排除出去。我们称之为服务续约。



## 五 服务消费者

### 5.1 获取服务

消费者服务启动时，会发送一个Rest请求给服务注册中心，来获取上面注册的服务清单。为了性能考虑，Eureka Server会维护一份只读的服务注册清单来返回给客户端，同时该缓存清单默认会每隔30秒更新一次。

下面是获取服务的两个重要的属性：

（1）      eureka.client.fetch-registry

是否需要去检索寻找服务，默认是true



（2）eureka.client.registry-fetch-interval-seconds

表示eureka client间隔多久去拉取服务注册信息，默认为30秒，对于api-gateway，如果要迅速获取服务注册状态，可以缩小该值，比如5秒

### 5.2 服务调用

服务消费者在获取服务清单后，通过服务名可以获取具体提供服务的实例名和该实例的元数据信息。因为有这些服务实例的详细信息，所以客户端可以根据自己的需要决定具体调用哪个实例，在Ribbon中会默认采用轮询的方式进行调用，从而实现客户端的负载均衡。

### 5.3 服务下线

在系统运行过程中必然会面临关闭或重启服务的某个实例的情况，在服务关闭操作时，会触发一个服务下线的Rest服务请求给Eureka Server，告诉服务注册中心：“我要下线了。”服务端在接收到该请求后，将该服务状态置位下线（DOWN），并把该下线事件传播出去。

