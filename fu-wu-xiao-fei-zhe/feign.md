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





### **三、传递全局参数**

通过requestInterceptor将全局参数通过apply\(RequestTemplate\) 方法传递到http请求的请求头里面。

### **四、使用方式**

使用 Feign 涉及到了两个注解，一个是@EnableFeignClients，用来开启 Feign，另一个是@FeignClient，用来标记要用 Feign 来拦截的请求接口。

@FeignClient 要配合@RequestMapping、@RequestParam等注解使用，生成完整的http路径和参数，详见hi\_kuaibao使用实例

### **五、核心对象**

FeignClient，FeignClientFactoryBean，FeignClientSpecification，FeignContext，RequestTemplate，Targeter，RequestInterceptor，FeignAutoConfiguration，SynchronousMethodHandler，FeignInvocationHanlder

