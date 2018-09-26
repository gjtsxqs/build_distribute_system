### Sharding-JDBC出身

Sharding-JDBC是当当应用框架ddframe中，从关系型数据库模块dd-rdb中分离出来的数据库水平分片框架，实现透明化数据库分库分表访问。Sharding-JDBC是继dubbox和elastic-job之后，ddframe系列开源的第3个项目。 

Sharding-JDBC直接封装JDBC协议，可以理解为增强版的JDBC驱动，旧代码迁移成本几乎为零。 

Sharding-JDBC定位为轻量级java框架，使用客户端直连数据库，以jar包形式提供服务，无proxy代理层，无需额外部署，无其他依赖，DBA也无需改变原有的运维方式。

### Sharding-JDBC 适用于哪些场景，不适用于哪些场景？是否有性能评估？

对于关系型数据库数据量很大的情况，需要进行水平拆库和拆表，这种场景很适合使用 Sharding-JDBC。

举例说明：假设有一亿数据的用户库，放在 MySQL 数据库里查询性能会比较低，而采用水平拆库，将其分为 10 个库，根据用户的 ID 模 10，这样数据就能比较平均的分在 10 个库中，每个库只有 1000w 记录，查询性能会大大提升。分片策略类型非常多，大致分为 Hash + Mod、Range、Tag 等。

Sharding-JDBC 还提供了读写分离的能力，用于减轻写库的压力。

此外，Sharding-JDBC 可以用在 JPA 场景中，如 JPA、Hibernate、Mybatis，Spring JDBC Template 等任何 Java 的 ORM 框架。

Java 的 ORM 框架也都是采用 JDBC 与数据库交互。这也是我们选择在 JDBC 层，而非选择一个 ORM 框架进行开发的原因。我们希望 Sharding-JDBC 可以尽量的兼容所有的 Java 数据库访问层，并且无缝的接入业务应用。

不合适的场景主要是两方面：

1. 不适合 OLAP 的场景。虽然 Sharding-JDBC 也能做聚合分组查询，但大量的 OLAP 场景，仍然会比较慢，而且复杂的 SQL（如子查询等）目前还没有支持。这种查询不太适合大数据和高并发的互联网 online 数据库，建议使用合理的 OLTP 查询。
2. 不适合事务强一致的要求。目前 Sharding-JDBC 的事务支持两种，一种是弱 XA，另一种是柔性事务（BASE）。因为 XA 的两阶段或三阶段提交其性能较低，因此互联网公司基本不会采用。而无论是弱 XA 还是柔性事务，都无法保证事务在任意时间段完全保证一致，其中柔性事务能保证数据的最终一致性，但达到最终一致性的时间仍然不可控。因此对于对跨库事务强一致要求很高的场景，需要从设计方面去考虑数据库 schema 的合理性。

对于 JTA 事务，目前 Shariding-JDBC 没有实现 JTA 的标准。而且由于在互联网场景下使用 JTA 比较少见，因此暂时不支持 JIA 事务。

在 osgit 上有性能测试文档。单库的场景下，由于需要进行 SQL 解析以及路由，器性能损失是 0.02%。双库的场景下，采用了分布式的方式存取数据，性能提升越 94%。

### Sharding-JDBC 与 Mycat区别和比较

从设计理念上看确实有一定的相似性。主要流程都是SQL 解析 -&gt; SQL 改写 -&gt; SQL 路由 -&gt; SQL 执行 -&gt; 结果归并。但架构设计上是不同的。Mycat 是基于 Proxy，它复写了 MySQL 协议，将 Mycat Server 伪装成一个 MySQL 数据库，而 Sharding-JDBC 是基于 JDBC 接口的扩展，是以 jar 包的形式提供轻量级服务的。

SQL 解析这块，现在的 Shariding-JDBC 和 Mycat 也比较相似，都是使用 Druid 作为 SQL 解析的基础类库。但 Sharding-JDBC 正在重写 SQL 解析这块，是去掉 Duird 的完全自研版本。不可否认 Druid 是一个优秀的连接池，而且 SQL 解析这块做得也很强，但它毕竟不是一个专门为了 Sharding 而做的 SQL 解析器，它的大致解析流程是 Lexer -&gt; Parser -&gt; AST -&gt; Vistor，使用者需要实现它的 Vistor 接口，将自己的业务逻辑在 Vistor 中实现，因此需要通过 Vistor 再生成 SharidingContext，而抽象语法树 AST，也需要对 SQL 完全理解。Sharding-JDBC 自研的 SQL 解析器，对于 Sharding 不相关的关键词采用跳过的方法，整体解析流程简化为 Lexer -&gt; Parser -&gt; SharidingContext，在性能以及实现复杂度上都有所突破。



#### 分库分表使用 like 查询，是否能查询出来？性能如何？会去查询所有的库和表吗？

分库分表使用 like 查询是有限制的。目前 Shariding-JDBC 不支持 like 语句中包含分片键，但不包含分片键的 like 语句可以正确执行。至于 like 性能问题，是与数据库相关的，Shariding-JDBC 仅仅是解析 SQL 以及路由至正确的数据源而已。是否会查询所有的库和表是根据分片键决定的，如果 SQL 中不包括分片键，就会查询所有库和表，这个和是否有 like 没有关系。

#### Sharding-JDBC 如何强制查询走主库？

大致使用方式如下：

```
HintManager hintManager = HintManager.getInstance();
hintManager.setMasterRouteOnly();

// 继续JDBC操作
```

#### 

### 



