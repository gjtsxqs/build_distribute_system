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

### **三、传递全局参数**

通过requestInterceptor将全局参数通过apply\(RequestTemplate\) 方法传递到http请求的请求头里面



