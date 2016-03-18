一旦使用Shark分库分表后，原本单库中使用的序列自增Id将无法再继续使用，那么这应该怎么办呢？其实解决这个问题并不困 难，目前有2种方案，第一种是所有的应用统一调用一个集中式的Id生成器，另外一种则是每个应用集成一个Id生成器。无论选择哪一种方案，Id生成器所持 有的DB都只有一个，也就是说，通过一个通用DB去管理和生成全局以为的sequenceId。

目前市面上几乎所有的Sharding中间件都没有提供sequenceId的支持，而Shark的工具包中却为大家提供了支持。Shark选 择的是每个应用都集成一个Id生成器，而没有采用集中式Id生成器，因为在分布式场景下，多一个外围系统依赖就意味着多一分风险。那么究竟应该如何使用 Shark提供的Id生成器呢？首先我们需要单独准备一个全局DB出来，然后使用Shark提供的建表语句，如下所示：
```Sql
#sequenceid的sql
CREATE TABLE shark_sequenceid(
	s_id INT NOT NULL AUTO_INCREMENT COMMENT '主键',
	s_type INT NOT NULL COMMENT '类型',
	s_useData BIGINT NOT NULL COMMENT '申请占位数量',
	PRIMARY KEY (s_id)
)ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE utf8mb4_bin;
```

当成功建立好生成sequenceId所需要的数据库表后，接下来要做的事情就是进行调用，生成全局唯一的sequenceId，如下所示：
```Java
/**
 * 获取SequenceId
 * 
 * @author gaoxianglong
 */
public @Test void getSequenceId() {
    /* 初始化数据源信息 */
    DbConnectionManager.init("account", "pwd", "url", "driver");
    System.out.println(SequenceIDManger.getSequenceId(1, 1, 5000));
}
```

上述程序示例中，首先需要调用com.sharksharding.shark.util.sequence.DbConnectionManager类的 init()方法对数据源进行初始化，然后调用DbConnectionManager类的 getSequenceId()方法即可成功获取全局唯一的sequenceId。在此大家需要注意，Shark生成的sequenceId是一个 17-19位之间的整型，在getSequenceId()方法中，第一个参数为IDC机房编码，第二个参数为类型操作码，而最后一个参数非常关键，就是 需要向DB中申请的内存占位数量(自增码)。

简单来说，相信大家都知道，既然业务库都分库分表了，就是为了缓解数据库读写瓶颈，当并发较高时，一个生成sequenceId的通用数据库能扛得 住吗？扛得住！关键就是因为内存占位。其实原理很简单，当第一次应用从Id生成器中去拿sequenceId时，Id生成器会锁表并将数据库字段 s_useData更新为5000，那么第二次应用从Id生成器中去拿sequenceId时，将不会与DB建议物理会话链接，而是直接在内存中去消耗着 5000内存占位数，直至消耗殆尽时，才会重新去数据库中申请下一次的内存占位。

那么如果并发访问会不会有问题呢？其实保证全局唯一性有3点，第一是程序中会加锁，其次数据库会for update，最后每一个操作码都是唯一的，都会管理自己旗下的内存占位数(通过Max()函数比较，累加下一个内存占位量，比如5000)。或许你会在 想，如何提升Id生成器的性能，尽可能的避免与数据库建立物理会话，没错，这么想是对的，每次假设从数据库申请的占位数量是50000，那么性能肯定比只 申请5000好，但是这也有弊端，一旦程序出现down机，内存中的内存数量就会丢失，只能重新申请，这会造成资源浪费。