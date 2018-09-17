### 一、Zuul简介 {#一zuul简介}

Zuul的主要功能是路由转发和过滤器。路由功能是微服务的一部分，比如／api/user转发到到user服务，/api/shop转发到到shop服务。zuul默认和Ribbon结合实现了负载均衡的功能。

### **Zuul 可以做什么？**

* 身份认证

* 审查与监控
* 压力测试
* 金丝雀测试
* 动态路由
* 服务迁移
* 负载分配
* 安全
* 静态响应处理
* 主动/主动流量管理

### Zuul的核心

Filter是Zuul的核心，用来实现对外服务的控制。Filter的生命周期有4个，分别是“PRE”、“ROUTING”、“POST”、“ERROR”，整个生命周期可以用下图来表示。![](/assets/import26.png)Zuul大部分功能都是通过过滤器来实现的，这些过滤器类型对应于请求的典型生命周期。

* PRE：
   这种过滤器在请求被路由之前调用。我们可利用这种过滤器实现身份验证、在集群中选择请求的微服务、记录调试信息等。
* ROUTING：
  这种过滤器将请求路由到微服务。这种过滤器用于构建发送给微服务的请求，并使用Apache HttpClient或Netfilx Ribbon请求微服务。
* POST：
  这种过滤器在路由到微服务以后执行。这种过滤器可用来为响应添加标准的HTTP Header、收集统计信息和指标、将响应从微服务发送给客户端等。
* ERROR：
  在其他阶段发生错误时执行该过滤器。 除了默认的过滤器类型，Zuul还允许我们创建自定义的过滤器类型。例如，我们可以定制一种STATIC类型的过滤器，直接在Zuul中生成响应，而不将请求转发到后端的微服务。

### Zuul中默认实现的Filter

| 类型 | 顺序 | 过滤器 | 功能 |
| :--- | :--- | :--- | :--- |
| pre | -3 | ServletDetectionFilter | 标记处理Servlet的类型 |
| pre | -2 | Servlet30WrapperFilter | 包装HttpServletRequest请求 |
| pre | -1 | FormBodyWrapperFilter | 包装请求体 |
| route | 1 | DebugFilter | 标记调试标志 |
| route | 5 | PreDecorationFilter | 处理请求上下文供后续使用 |
| route | 10 | RibbonRoutingFilter | serviceId请求转发 |
| route | 100 | SimpleHostRoutingFilter | url请求转发 |
| route | 500 | SendForwardFilter | forward请求转发 |
| post | 0 | SendErrorFilter | 处理有错误的请求响应 |
| post | 1000 | SendResponseFilter | 处理正常的请求响应 |

禁用指定的Filter

可以在application.yml中配置需要禁用的filter，格式：

![](/assets/import27.png)

## 自定义Filter

实现自定义Filter，需要继承ZuulFilter的类，并覆盖其中的4个方法。

## 路由熔断

当我们的后端服务出现异常的时候，我们不希望将异常抛出给最外层，期望服务可以自动进行一降级。Zuul给我们提供了这样的支持。当某个服务出现异常时，直接返回我们预设的信息。

我们通过自定义的fallback方法，并且将其指定给某个route来实现该route访问出问题的熔断处理。主要继承ZuulFallbackProvider接口来实现，ZuulFallbackProvider默认有两个方法，一个用来指明熔断拦截哪个服务，一个定制返回内容。

## 路由重试

有时候因为网络或者其它原因，服务可能会暂时的不可用，这个时候我们希望可以再次对服务进行重试，Zuul也帮我们实现了此功能，需要结合Spring Retry 一起来实现。下面我们以上面的项目为例做演示。

首先在spring-cloud-zuul项目中添加Spring Retry依赖。

![](/assets/import29.png)

开启Zuul Retry

再配置文件中配置启用Zuul Retry

```
#是否开启重试功能
zuul.retryable=true
#对当前服务的重试次数
ribbon.MaxAutoRetries=2
#切换相同Server的次数
ribbon.MaxAutoRetriesNextServer=0
```

### **路由配置详解**

显式声明路由配置：

```
zuul.routes.user.path: /user/**
zuul.routes.user.service-id: user-service
```

指定Path和Url

```
zuul:
routes:
    hello-service:
        path:/api-hello/**   
        #路由路径
        url:http://localhost:9001/#指定URL地址
```

### Zuul的高可用

**一 Zuul客户端也注册到Eureka Server上**

这种情况下，Zuul的高可用非常简单，只须将多个Zuul节点注册到Eureka Server上，就可实现Zuul的高可用。此时Zuul的高可用与其他微服务的高可用没什么区别。

**二 Zuul客户端未注册到Eureka Server上**

现实中，这种场景更多，例如，Zuul客户端是一个手机APP——不可能让所有的手机终端都注册到Eureka Server上。这种情况下，可借助一个额外的负载均衡器来实现Zuul的高可用，例如Nginx、HAProxy、F5等。

如下图，Zuul客户端请求发送到负载均衡器，负载均衡器将请求转发到其代理的其中一个Zuul节点。这样，就可以实现Zuul的高可用。

![](/assets/import30.png)

