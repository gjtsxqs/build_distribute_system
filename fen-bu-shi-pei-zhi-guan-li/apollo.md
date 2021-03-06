## Apollo简介 {#二apollo简介}

Apollo（阿波罗）是携程框架部门研发的开源配置管理中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性。

Apollo支持4个维度管理Key-Value格式的配置：

application \(应用\)  
environment \(环境\)  
cluster \(集群\)  
namespace \(命名空间\)  
同时，Apollo基于开源模式开发，开源地址：[https://github.com/ctripcorp/apollo](https://github.com/ctripcorp/apollo)

## 配置基本概念 {#三配置基本概念}

既然Apollo定位于配置中心，那么在这里有必要先简单介绍一下什么是配置。

按照我们的理解，配置有以下几个属性：  
1、配置是独立于程序的只读变量  
**\#**配置首先独立于程序的，同一份程序在不同的配置下会有不通的行为。  
**\#**其次，配置对于程序是只读，程序通过读取配置来改变自己的行为，但是程序不应该去改变配置。  
**\#**常见的配置有：DB Connection Str、Thread Pool Size、Buffer Size、Request Timeout、Feature Switch、Server Urls等  
2、配置伴随应用的整个生命周期  
**\#**配置贯穿于应用的整个生命周期，应用启动时通过读取配置来初始化，在运行时根据配置调行为。  
3、配置可有多种加载方式  
**\#**配置也有很多种加载方式，常见的有程序内部hard code，配置文件，在环境变量，启动参数，基于数据库等  
4、配置需要治理  
**\#**权限控制：  
由于配置能改变的程序的行为，不正确的配置甚至能引起灾难，所以对配置的修改必须有比较完善的权限控制  
**\#**不同的环境、集群配置管理  
同一份程序在不同的环境（开发、测试、生产）、不同的集群（如不同的数据中心）经常需要不同的【配置，所以需要有完善的环境、集群配置管理  
5、框架类组件配置管理  
**\#**还有一类比较特殊的配置 - 框架类组件配置，比如CAT客户端的配置。  
**\#**虽然这类框架类组件是由其他团队开发、维护，但是运行时是在业务实际应用内的，所以本质上可以认为框架类组件也是应用的一部分  
**\#**这类组件对应的配置也需要有比较完善的管理方式。



## 特性 {#四-特性}

1、统一管理不同环境、不同集群的配置  
\*\*\#\*\*Apollo提供了一个统一界面集中式管理不同环境（environment）、不同集群（cluster）、不同命名空间（namespace）的配置。  
**\#**同一份代码部署在不同的集群，可以有不同的配置，比如zk的地址等  
**\#**通过命名空间（namespace）可以很方便的支持多个不同应用共享同一份配置，同时还允许应用对共享的配置进行覆盖  
2、配置修改实时生效（热发布）  
**\#**用户在Apollo修改完配置并发布后，客户端能实时（1秒）接收到最新的配置，并通知到应用程序  
3、版本发布管理  
**\#**所有的配置发布都有版本概念，从而可以方便地支持配置的回滚  
4、灰度发布  
**\#**支持配置的灰度发布，比如点了发布后，只对部分应用实例生效，等观察一段时间没问题后再推给所有应用实例  
5、权限管理、发布审核、操作审计  
**\#**应用和配置的管理都有完善的权限管理机制，对配置的管理还分为了编辑和发布两个环节，从而减少人为的错误。  
**\#**所有的操作都有审计日志，可以方便的追踪问题  
6、客户端配置信息监控  
**\#**可以在界面上方便地看到配置在被哪些实例使用  
7、提供Java和.Net原生客户端  
**\#**提供了Java和.Net的原生客户端，方便应用集成  
**\#**支持Spring Placeholder, Annotation和Spring Boot的ConfigurationProperties，方便应用使用（需要Spring 3.1.1+）  
**\#**同时提供了Http接口，非Java和.Net应用也可以方便的使用  
8、提供开放平台API  
\*\*\#\*\*Apollo自身提供了比较完善的统一配置管理界面，支持多环境、多数据中心配置管理、权限、流程治理等特性。  
**\#**不过Apollo出于通用性考虑，对配置的修改不会做过多限制，只要符合基本的格式就能够保存  
**\#**在我们的调研中发现，对于有些使用方，它们的配置可能会有比较复杂的格式，而且对输入的值也需要进行校验后方可保存，如检查数据库、用户名和密码是否匹配  
**\#**对于这类应用，Apollo支持应用方通过开放接口在Apollo进行配置的修改和发布，并且具备完善的授权和权限控制  
9、部署简单  
**\#**配置中心作为基础服务，可用性要求非常高，这就要求Apollo对外部依赖尽可能地少  
**\#**目前唯一的外部依赖是MySQL，所以部署非常简单，只要安装好Java和MySQL就可以让Apollo跑起来  
\*\*\#\*\*Apollo还提供了打包脚本，一键就可以生成所有需要的安装包，并且支持自定义运行时参数



## **接入Apollo步骤**

#### 1、Appid {#appid}

确保classpath:/META-INF/app.properties文件存在，并且其中内容形如：app.id=YOUR-APP-ID

#### 2、Environment {#environment}

应用在不同的环境可以有不同的配置, Environment可以通过以下3种方式的任意一个配置：

* 2.1 通过Java的System Property env来指定环境 -Denv=YOUR-ENVIRONMENT
* 2.2 通过操作系统的System Environment env来指定环境
* 2.3 通过配置文件来指定env=YOUR-ENVIRONMENT
  对于Mac/Linux，文件位置为/opt/settings/server.properties
  对于Windows，文件位置为C:\opt\settings\server.properties
  目前，env支持以下几个值（大小写不敏感）：
  DEV, FAT, UAT, PRO

#### 3、本地缓存 {#本地缓存}

Apollo客户端会把从服务端获取到的配置在本地文件系统缓存一份，当去服务器读取配置失败时，会使用本地缓存的。

**Mac/Linux: /opt/data/{appId}/config-cache**

**Windows: C:\opt\data{appId}\config-cache**

#### 4、添加依赖 {#添加依赖}

![](/assets/import2.png)

#### 5、指定服务端 {#指定服务端}

通过Java的System Property env来指定

-Ddev\_meta=http://192.168.30.27:8018

#### 6、读取配置 {#读取配置}

通过namespace读取配置，如果不指定则默认拿application

* 6.1 api方式

![](/assets/import3.png)

* 6.2 结合Spring方式

Apollo和Spring也可以很方便地集成，只需要标注@EnableApolloConfig后就可以通过@Value获取配置信息：

![](/assets/import6.png)



