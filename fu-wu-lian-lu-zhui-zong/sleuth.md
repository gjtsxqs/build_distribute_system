### 什么是Sleuth

Spring-Cloud-Sleuth是Spring Cloud的组成部分之一，为SpringCloud应用实现了一种分布式追踪解决方案，其兼容了

Zipkin, HTrace和log-based追踪。

### 基本术语

Spring Cloud Sleuth采用的是Google的开源项目Dapper的专业术语。

* Span：基本工作单元，发送一个远程调度任务 就会产生一个Span，Span是一个64位ID唯一标识的，Trace是用另一个64位ID唯一标识的，Span还有其他数据信息，比如摘要、时间戳事件、Span的ID、以及进度ID。
* Trace：一系列Span组成的一个树状结构。请求一个微服务系统的API接口，这个API接口，需要调用多个微服务，调用每个微服务都会产生一个新的Span，所有由这个请求产生的Span组成了这个Trace。
* Annotation：用来及时记录一个事件的，一些核心注解用来定义一个请求的开始和结束 。这些注解包括以下：
  * cs - Client Sent -客户端发送一个请求，这个注解描述了这个Span的开始
  * sr - Server Received -服务端获得请求并准备开始处理它，如果将其sr减去cs时间戳便可得到网络传输的时间。
  * ss - Server Sent （服务端发送响应）–该注解表明请求处理的完成\(当请求返回客户端\)，如果ss的时间戳减去sr时间戳，就可以得到服务器请求的时间。
  * cr - Client Received （客户端接收响应）-此时Span的结束，如果cr的时间戳减去cs时间戳便可以得到整个请求所消耗的时间。

# sleuth 结合zipkin {#sleuth-结合zipkin}

### 构建zipkin-server工程 {#构建zipkin-server工程}

新建一个Module工程，取名为zipkin-server，其pom文件继承了主Maven工程的pom文件；作为Eureka Client，引入Eureka的起步依赖spring-cloud-starter-eureka，引入zipkin-server依赖，以及zipkin-autoconfigure-ui依赖，后两个依赖提供了Zipkin的功能和Zipkin界面展示的功能。

![](/assets/import42.png)

启动类添加

* `@EnableDiscoveryClient`和`@EnableZipkinServer`注解
* 配置文件`application.yml`

![](/assets/import44.png)

### 客户端整合zipkin步骤 {#客户端整合zipkin步骤}

客户端添加依赖

![](/assets/import41.png)

配置文件添加

![](/assets/import45.png)

指定了zipkin server的地址，下面制定需采样的百分比，默认为0.1，即10%，这里配置1，是记录全部的sleuth信息，是为了收集到更多的数据（仅供测试用）。在分布式系统中，过于频繁的采样会影响系统性能，所以这里配置需要采用一个合适的值。

### zipkin改进 {#zipkin改进}

在这里对zipkin进行改进，主要包含两方面  
- 通过消息中间件收集sleuth数据  
- 持久化sleuth数据

**1、通过消息中间件收集sleuth数据**

通过消息中间件可以将zipkin server和微服务解耦，微服务无需知道zipkin server地址，只需将sleuth数据传入消息中间件。同时，也可以解决zipkin server与微服务网络不通情况。

**改造服务端**

- 修改zipkin server（trace项目）配置

![](/assets/import46.png)

配置文件application.yml增加

```
  rabbitmq:
    host: localhost
    
port
: 
5673

    username: guest
    password: guest

```

这是sleuth数据来源  
- 启动类`@EnableZipkinServer`改为`@EnableZipkinStreamServe`  
以上服务端改造完毕，下面改造客户端（以`helloworld-feign项目为例`）

**改造客户端**  
以`helloworldfeign`项目为例

- 修改依赖

![](/assets/import58.png)

配置文件增加

```
spring:

  rabbitmq:
    host: localhost
    port: 
5673

    username: guest
    password: guest

```

并删除下面指向zipkin server的配置

```
spring:
  zipkin:
    
base
-url: http:
//127.0.0.1:9411
```







