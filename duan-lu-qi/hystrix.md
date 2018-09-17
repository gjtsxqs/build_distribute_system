### **Netflix简介**

Netflix开源了Hystrix组件，实现了断路器模式，SpringCloud对这一组件进行了整合。 在微服务架构中，一个请求需要调用多个服务是非常常见的。较底层的服务如果出现故障，会导致连锁故障。当对特定的服务的调用的不可用达到一个阀值（Hystric 是5秒20次） 断路器将会被打开。

![](/assets/import15.png)



## 流程说明 {#流程说明}

hystrix运行流程说明:  
1:每次调用创建一个新的HystrixCommand,把依赖调用封装在run\(\)方法中.  
2:执行execute\(\)/queue做同步或异步调用.  
3:判断熔断器\(circuit-breaker\)是否打开,如果打开跳到步骤8,进行降级策略,如果关闭进入步骤.  
4:判断线程池/队列/信号量是否跑满，如果跑满进入降级步骤8,否则继续后续步骤.  
5:调用HystrixCommand的run方法.运行依赖逻辑  
    5a:依赖逻辑调用超时,进入步骤8.  
6:判断逻辑是否调用成功  
    6a:返回成功调用结果  
    6b:调用出错，进入步骤8.  
7:计算熔断器状态,所有的运行状态\(成功, 失败, 拒绝,超时\)上报给熔断器，用于统计从而判断熔断器状态.  
8:getFallback\(\)降级逻辑.  
    8a:没有实现getFallback的Command将直接抛出异常  
    8b:fallback降级逻辑调用成功直接返回  
    8c:降级逻辑调用失败抛出异常  
9:返回执行成功结果

### 在ribbon使用断路器 {#三在ribbon使用断路器}

改造serice-ribbon 工程的代码，首先在pox.xml文件中加入spring-cloud-starter-netflix-hystrix的起步依赖：

![](/assets/import17.png)

在程序的启动类ServiceRibbonApplication 加@EnableHystrix注解开启Hystrix：

![](/assets/import18.png)

改造HelloService类，在hiService方法上加上@HystrixCommand注解。该注解对该方法创建了熔断器的功能，并指定了fallbackMethod熔断方法，熔断方法直接返回了一个字符串，字符串为”hi,”+name+”,sorry,error!”，代码如下：

![](/assets/impor19t.png)

### Feign中使用断路器 {#四feign中使用断路器}

Feign是自带断路器的，在D版本的Spring Cloud之后，它没有默认打开。需要在配置文件中配置打开它，在配置文件加以下代码：

```
 feign.hystrix.enabled=true!
```

基于service-feign工程进行改造，只需要在FeignClient的SchedualServiceHi接口的注解中加上fallback的指定类就行了：

![](/assets/import21.png)

SchedualServiceHiHystric需要实现SchedualServiceHi 接口，并注入到Ioc容器中，代码如下：

![](/assets/import22.png)

