### 什么是mycat

MyCAT是mysql中间件，前身是阿里大名鼎鼎的Cobar，Cobar在开源了一段时间后，不了了之。于是MyCAT扛起了这面大旗，在大数据时代，其重要性愈发彰显。这篇文章主要是MyCAT的入门部署。

MyCat是一个开源的分布式数据库系统，是一个实现了MySQL协议的服务器，前端用户可以把它看作是一个数据库代理，用MySQL客户端工具和命令行访问，而其后端可以用MySQL原生协议与多个MySQL服务器通信，也可以用JDBC协议与大多数主流数据库服务器通信，其核心功能是分表分库，即将一个大表水平分割为N个小表，存储在后端MySQL服务器里或者其他数据库里。

MyCat发展到目前的版本，已经不是一个单纯的MySQL代理了，它的后端可以支持MySQL、SQL Server、Oracle、DB2、PostgreSQL等主流数据库，也支持MongoDB这种新型NoSQL方式的存储，未来还会支持更多类型的存储。而在最终用户看来，无论是那种存储方式，在MyCat里，都是一个传统的数据库表，支持标准的SQL语句进行数据的操作，这样一来，对前端业务系统来说，可以大幅降低开发难度，提升开发速度。



### 使用限制

**1.非分片字段查询**

Mycat中的路由结果是通过**分片字段**和**分片方法**来确定的。例如下图中的一个Mycat分库方案：

* 根据 **tt\_waybill**表的 **id**字段来进行分片
* 分片方法为 **id**值取 **3**的模，根据模值确定在DB1，DB2，DB3中的某个分片

![](/assets/importa5.png)

如果查询条件中有 **id** 字段的情况还好，查询将会落到某个具体的分片。例如：

> [MySQL](http://lib.csdn.net/base/mysql)&gt;select \* from tt\_waybill where **id** = 12330;

此时Mycat会计算路由结果

> 12330 % 3 = 0 –&gt; DB1

并将该请求路由到DB1上去执行。   
  
  
如果查询条件中没有 **分片字段** 条件，例如：

> [mysql](http://lib.csdn.net/base/mysql)&gt;select \* from tt\_waybill where waybill\_no =88661;

此时Mycat无法计算路由，便发送到所有节点上执行：

> DB1 –&gt; select \* from tt\_waybill where waybill\_no =88661;   
> DB2 –&gt; select \* from tt\_waybill where waybill\_no =88661;   
> DB3 –&gt; select \* from tt\_waybill where waybill\_no =88661;

如果该分片字段选择度高，也是业务常用的查询维度，一般只有一个或极少数个DB节点命中（返回结果集）。示例中只有3个DB节点，而实际应用中的DB节点数远超过这个，假如有50个，那么前端的一个查询，落到MySQL

[数据库](http://lib.csdn.net/base/mysql)

上则变成50个查询，会极大消耗Mycat和MySQL数据库资源。

**2.分页排序**

**3.任意表JOIN**

**4.分布式事务**

Mycat并没有根据二阶段提交协议实现 **XA事务**，而是只保证 **prepare** 阶段数据一致性的 **弱XA事务** 



