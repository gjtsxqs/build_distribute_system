### **一、Feign简介**

Feign是一个声明式的伪Http客户端，它使得写Http客户端变得更简单。使用Feign，只需要创建一个接口并注解。它具有可插拔的注解特性，可使用Feign 注解和JAX-RS注解。Feign支持可插拔的编码器和解码器。Feign默认集成了Ribbon，并和Eureka结合，默认实现了负载均衡的效果。

### **二、Feign代码逻辑 **

总到来说，Feign的源码实现的过程如下：

* 首先通过@EnableFeignCleints注解开启FeignCleint

* 根据Feign的规则实现接口，并加@FeignCleint注解

* 程序启动后，会进行包扫描，扫描所有的@ FeignCleint的注解的类，并将这些信息注入到ioc容器中。

* 当接口的方法被调用，通过jdk的代理，来生成具体的RequesTemplate

* RequesTemplate在生成Request

* Request交给Client去处理，其中Client可以是HttpUrlConnection、HttpClient也可以是Okhttp

* 最后Client被封装到LoadBalanceClient类，这个类结合类Ribbon做到了负载均衡。

### 请求是怎么转到 Feign 的？ {#h3_2}

分为两部分，第一是为接口定义的每个接口都生成一个实现方法，结果就是 SynchronousMethodHandler 对象。第二是为该服务接口生成了动态代理。动态代理的实现是 ReflectiveFeign.FeignInvocationHanlder，代理被调用的时候，会根据当前调用的方法，转到对应的 SynchronousMethodHandler。

### Feign 是怎么工作的？ {#h3_3}

当对接口的实例进行请求时（Autowire 的对象是某个ReflectiveFeign.FeignInvocationHanlder 的实例），根据方法名进入了某个 SynchronousMethodHandler 对象的 invoke 方法。

SynchronousMethodHandler 其实也并不处理具体的 HTTP 请求，它关心的更多的是请求结果的处理。HTTP 请求的过程，包括服务发现，都交给了当前 context 注册中的 Client 实现类，比如 LoadBalancerFeignClient。Retry 的逻辑实际上已经提出来了，但是 fallback 并没有在上面体现，因为我们上面分析动态代理的过程中，用的是 Feign.Builder，而如果有 fallback 的情况下，会使用 HystrixFeign.Builder，这是 Feign.Builder 的一个子类。它在创建动态代理的时候，主要改了一个一个东西，就是 invocationFactory 从默认的 InvocationHandlerFactory.Default 变成了一个内部匿名工厂，这个工厂的create 方法返回的不是 ReflectiveFeign.FeignInvocationHandler，而是 HystrixInvocationHandler。所以动态代理类换掉了，invoke 的逻辑就变了。在新的逻辑里，没有简单的将方法转到对应的 SynchronousMethodHandler 上面，而是将 fallback 和 SynchronousMethodHandler一起封装成了 HystrixMethod，并且执行该对象。

### **三、传递全局参数**

通过requestInterceptor将全局参数通过apply\(RequestTemplate\) 方法传递到http请求的请求头里面。

### **四、使用方式**

使用 Feign 涉及到了两个注解，一个是@EnableFeignClients，用来开启 Feign，另一个是@FeignClient，用来标记要用 Feign 来拦截的请求接口。

@FeignClient 要配合@RequestMapping、@RequestParam等注解使用，生成完整的http路径和参数，详见hi\_kuaibao使用实例

### **五、核心对象**

FeignClient，FeignClientFactoryBean，FeignClientSpecification，FeignContext，RequestTemplate，Targeter，RequestInterceptor，FeignAutoConfiguration，SynchronousMethodHandler，FeignInvocationHanlder

