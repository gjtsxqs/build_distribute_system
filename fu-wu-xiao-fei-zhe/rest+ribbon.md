### 一、ribbon简介 {#一ribbon简介}

ribbon是一个负载均衡客户端，可以很好的控制htt和tcp的一些行为。Feign默认集成了ribbon。

ribbon 已经默认实现了这些配置bean：

* IClientConfig ribbonClientConfig: DefaultClientConfigImpl

* IRule ribbonRule: ZoneAvoidanceRule

* IPing ribbonPing: NoOpPing

* ServerList ribbonServerList: ConfigurationBasedServerList

* ServerListFilter ribbonServerListFilter: ZonePreferenceServerListFilter

* ILoadBalancer ribbonLoadBalancer: ZoneAwareLoadBalancer



#### 二、建一个服务消费者 {#三建一个服务消费者}

在工程的启动类中,通过@EnableDiscoveryClient向服务中心注册；并且向程序的ioc注入一个bean: restTemplate;并通过@LoadBalanced注解表明这个restRemplate开启负载均衡的功能。![](/assets/import11.png)

写一个测试类HelloService，通过之前注入ioc容器的restTemplate来消费service-hi服务的“/hi”接口，在这里我们直接用的程序名替代了具体的url地址，在ribbon中它会根据服务名来选择具体的服务实例，根据服务实例在请求的时候会用具体的url替换掉服务名，代码如下：

![](/assets/import12.png)

写一个controller，在controller中用调用HelloService 的方法，代码如下：

![](/assets/import13.png)

在浏览器上多次访问[http://localhost:8764/hi?name=forezp](http://localhost:8764/hi?name=forezp)，浏览器交替显示：

> hi forezp,i am from port:8762
>
> hi forezp,i am from port:8763

这说明当我们通过调用restTemplate.getForObject\(“[http://SERVICE-HI/hi?name=](http://service-hi/hi?name=)“+name,String.class\)方法时，已经做了负载均衡，访问了不同的端口的服务实例。

### 三、此时的架构 {#四此时的架构}

![](/assets/import14.png)









