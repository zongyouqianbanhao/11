简单来说，数据路由任务无非就是根据Sharding算法对持有的多数据源进行动态切换，这是任何Sharding中间件最核心的 部分。一旦在程序中使用Shark后，应用层将会持有N个数据源，Shark通过路由条件进行运算，然后通过Route技术对数据库和数据表进行读写 操作。在此大家需要注意，Shark内部并没有实现自己的ConnectionPool，这也就意味着，给予了开发人员极大的自由，使之可以随意切换任 意的ConnectionPool产品，比如你觉得C3P0没有BonePC性能高，那么你可以切换为BonePC，或者如果你觉得BonePC不够稳 定，你又可以切换回C3P0。

对于开发人员而言，并不需要关心底层的数据库和表的划分规则，程序中任何的CRUD操作，都像是在操作同一个数据库一样，并且读写效率还不能够比之 前低太多(比如几毫秒或者实际毫秒之内完成)，而Shark就承担着这样的一个任务。Shark所处的领域模型定位，如下所示：
![](http://dl.iteye.com/upload/picture/pic/133859/e114f4fc-3788-3e5d-9103-def32509b1c9.jpg)

Shark的领域模型定位介于持久层和JDBC之间。之前笔者曾经提及过，Shark是站在巨人的肩膀上，这个巨人正是Spring。简单来 说，Shark重写了JdbcTemplate，并使用了Spring提供的AbstractRoutingDataSource作为动态数据源层。因 此从另外一个侧面反应了Shark的源码注定是简单、轻量、易阅读、易维护的，因为Shark的核心只做分库分表。我们知道一般的Shading中间 件，动不动就几万行代码，其中得“猫腻”有很多，不仅数据库连接池要自己写、动态数据源要自己写，再加上一些杂七杂八的功能，比如：通用性支持、多种类型 的RDBMS或者Nosql支持，那么代码自然冗余，可读性极差。而Shark仅仅只会考虑如何通过Sharding规则实现数据路由。Shark的3层架构，如下所示：
![](http://dl.iteye.com/upload/picture/pic/133889/274e2a9c-c0ce-3c25-a17a-f1f914ca4667.jpg)

既然Shark只考虑最核心的功能，同时也就意味着它的性能恒定指标还需要结合其他第三方产品，比如Shark的动态数据源层所使用的 ConnectionPool可以为C3P0，也可以为BonePC。你别指望Shark还能为你处理边边角角的零碎琐事，想要什么效果，自行组合配 置，这就是Shark，一个简单、轻量级的Sharding中间件。Shark的应用总体架构，如下所示：
![](http://dl.iteye.com/upload/picture/pic/133881/a41cb049-d046-3293-af1b-27912c15a6ed.jpg)

