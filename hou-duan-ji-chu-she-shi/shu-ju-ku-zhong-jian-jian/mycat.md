### 什么是mycat

MyCAT是mysql中间件，前身是阿里大名鼎鼎的Cobar，Cobar在开源了一段时间后，不了了之。于是MyCAT扛起了这面大旗，在大数据时代，其重要性愈发彰显。这篇文章主要是MyCAT的入门部署。

MyCat是一个开源的分布式数据库系统，是一个实现了MySQL协议的服务器，前端用户可以把它看作是一个数据库代理，用MySQL客户端工具和命令行访问，而其后端可以用MySQL原生协议与多个MySQL服务器通信，也可以用JDBC协议与大多数主流数据库服务器通信，其核心功能是分表分库，即将一个大表水平分割为N个小表，存储在后端MySQL服务器里或者其他数据库里。

MyCat发展到目前的版本，已经不是一个单纯的MySQL代理了，它的后端可以支持MySQL、SQL Server、Oracle、DB2、PostgreSQL等主流数据库，也支持MongoDB这种新型NoSQL方式的存储，未来还会支持更多类型的存储。而在最终用户看来，无论是那种存储方式，在MyCat里，都是一个传统的数据库表，支持标准的SQL语句进行数据的操作，这样一来，对前端业务系统来说，可以大幅降低开发难度，提升开发速度。



## 安装 {#安装}

Mycat官网：[http://www.mycat.io/](http://www.mycat.io/)  
可以了解下Mycat的背景和应用情况，这样使用起来比较有信心。

Mycat下载地址：[http://dl.mycat.io/](http://www.mycat.io/)  
官网有个文档，属于详细的介绍，初次入门，看起来比较花时间。

**下载：**  
建议大家选择 1.6-RELEASE 版本，毕竟是比较稳定的版本。

**安装：**  
根据不同的系统选择不同的版本。包括linux、windows、mac,作者考虑还是非常周全的，当然，也有源码版的。（ps:源码版的下载后，只要配置正确，就可以正常运行调试，这个赞一下。）

![](https://ws3.sinaimg.cn/large/006tNc79gy1fjfersexy7j31kw0ac0zb.jpg)

Mycat的安装其实只要解压下载的目录就可以了，非常简单。  
安装完成后，目录如下：

| 目录 | 说明 |
| :--- | :--- |
| bin | mycat命令，启动、重启、停止等 |
| catlet | catlet为Mycat的一个扩展功能 |
| conf | Mycat 配置信息,重点关注 |
| lib | Mycat引用的jar包，Mycat是java开发的 |
| logs | 日志文件，包括Mycat启动的日志和运行的日志。 |

###  配置

Mycat的配置文件都在conf目录里面，这里介绍几个常用的文件：

| 文件 | 说明 |
| :--- | :--- |
| server.xml | Mycat的配置文件，设置账号、参数等 |
| schema.xml | Mycat对应的物理数据库和数据库表的配置 |
| rule.xml | Mycat分片（分库分表）规则 |

Mycat的架构其实很好理解，Mycat是代理，Mycat后面就是物理数据库。和Web服务器的Nginx类似。对于使用者来说，访问的都是Mycat，不会接触到后端的数据库。

**server.xml**

示例

![](/assets/importa6.png)

重点关注下面这段，其他默认即可。

| 参数 | 说明 |
| :--- | :--- |
| user | 用户配置节点 |
| --name | 登录的用户名，也就是连接Mycat的用户名 |
| --password | 登录的密码，也就是连接Mycat的密码 |
| --schemas | 数据库名，这里会和schema.xml中的配置关联，多个用逗号分开，例如需要这个用户需要管理两个数据库db1,db2，则配置db1,dbs |
| --privileges | 配置用户针对表的增删改查的权限，具体见文档吧 |

**schema.xml**

schema.xml是最主要的配置项，首先看我的配置文件。

![](/assets/importa7.png)

| 参数 |
| :--- |


|  | 说明 |
| :--- | :--- |
| schema | 数据库设置，此数据库为逻辑数据库，name与server.xml中schema对应 |
| dataNode | 分片信息，也就是分库相关配置 |
| dataHost | 物理数据库，真正存储数据的数据库 |

每个节点的属性逐一说明：

**schema:**

| 属性 | 说明 |
| :--- | :--- |
| name | 逻辑数据库名，与server.xml中的schema对应 |
| checkSQLschema | 数据库前缀相关设置，建议看文档，这里暂时设为folse |
| sqlMaxLimit | select 时默认的limit，避免查询全表 |

**table:**

| 属性 | 说明 |
| :--- | :--- |
| name | 表名，物理数据库中表名 |
| dataNode | 表存储到哪些节点，多个节点用逗号分隔。节点为下文dataNode设置的name |
| primaryKey | 主键字段名，自动生成主键时需要设置 |
| autoIncrement | 是否自增 |
| rule | 分片规则名，具体规则下文rule详细介绍 |

**dataNode**

| 属性 | 说明 |
| :--- | :--- |
| name | 节点名，与table中dataNode对应 |
| datahost | 物理数据库名，与datahost中name对应 |
| database | 物理数据库中数据库名 |

**dataHost**

| 属性 | 说明 |
| :--- | :--- |
| name | 物理数据库名，与dataNode中dataHost对应 |
| balance | 均衡负载的方式 |
| writeType | 写入方式 |
| dbType | 数据库类型 |
| heartbeat | 心跳检测语句，注意语句结尾的分号要加。 |



## 使用 {#使用}

Mycat的启动也很简单，启动命令在Bin目录：

```
##启动
mycat start


##停止
mycat 
stop


##重启
mycat restart
```

如果在启动时发现异常，在logs目录中查看日志。

* wrapper.log 为程序启动的日志，启动时的问题看这个
* mycat.log 为脚本执行时的日志，SQL脚本执行报错后的具体错误内容,查看这个文件。mycat.log是最新的错误日志，历史日志会根据时间生成目录保存。

mycat启动后，执行命令不成功，可能实际上配置有错误，导致后面的命令没有很好的执行。

Mycat带来的最大好处就是使用是完全不用修改原有代码的，在mycat通过命令启动后，你只需要将数据库连接切换到Mycat的地址就可以了。如下面就可以进行连接了：

```
 mysql -h192
.168.0.1 -P8806 -uroot -p123456
```

连接成功后可以执行sql脚本了。  
所以，可以直接通过sql管理工具（如：navicat、datagrip）连接，执行脚本。我一直用datagrip来进行日常简单的管理，这个很方便。

Mycat还有一个管理的连接，端口号是9906.

```
 mysql -h192
.168.0.1 -P9906 -uroot -p123456
```

连接后可以根据管理命令查看Mycat的运行情况，当然，喜欢UI管理方式的人，可以安装一个Mycat-Web来进行管理，有兴趣自行搜索。

简而言之，开发中使用Mycat和直接使用Mysql机会没有差别。

## 常见问题 {#常见问题}

使用Mycat后总会遇到一些坑，我将自己遇到的一些问题在这里列一下，希望能与大家有共鸣：

* Mycat是不是配置以后，就能完全解决分表分库和读写分离问题？  
  Mycat配合数据库本身的复制功能，可以解决读写分离的问题，但是针对分表分库的问题，不是完美的解决。或者说，至今为止，业界没有完美的解决方案。  
  分表分库写入能完美解决，但是，不能完美解决主要是联表查询的问题，Mycat支持两个表联表的查询，多余两个表的查询不支持。 其实，很多数据库中间件关于分表分库后查询的问题，都是需要自己实现的，而且节本都不支持联表查询，Mycat已经算做地非常先进了。  
  分表分库的后联表查询问题，大家通过合理数据库设计来避免。

* Mycat支持哪些数据库，其他平台如 .net、PHP能用吗？  
  官方说了，支持的数据库包括MySQL、SQL Server、Oracle、DB2、PostgreSQL 等主流数据库，很赞。  
  尽量用Mysql,我试过SQL Server，会有些小问题，因为部分语法有点差异。

* Mycat 非JAVA平台如 .net、PHP能用吗？  
  可以用。这一点MyCat做的也很棒。

### 



