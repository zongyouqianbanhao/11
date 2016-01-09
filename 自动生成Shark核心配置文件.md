Shark的工具包中除了提供有自动生成全局唯一的sequenceId的功能外，还提供有自动生成分库分表配置文件等功能，因 为这很明显可以减少配置出错率。比如我们采用库内分片模式，32个库1024个表，数据源配置文件中，需要配置的数据源信息会有32个，当然这通过手工的 方式也未尝不可，无非就是一个Shark分库分表配置文件加上一个dataSource文件(如果主从的话，还需要一个从库的dataSource文 件)，dataSource文件中需要编写32个数据源信息。但是如果我们使用的是一库一片这种分片方式，使用的库的数量是1024个的时 候，dataSource文件中需要定义的数据源将会是1024个，你能保证配置不会出问题？

既然手动编写配置文件极可能会出现错误，那么究竟应该如何使用Shark提供的自动生成配置文件功能来降低出错率呢？Shark自动生成配置文件，如下所示：
```Java
/**
 * 生成核心配置文件
 * 
 * @author gaoxianglong
 */
public @Test void testCreateCoreXml() {
    CreateCoreXml c_xml = new CreateCoreXml();
    /* 是否控制台输出生成的配置文件 */
    c_xml.setIsShow(true);
    /* 配置分库分片信息 */
    c_xml.setDbSize("64");
    c_xml.setShard("true");
    c_xml.setWrIndex("r0w32");
    c_xml.setShardMode("false");
    c_xml.setConsistent("false");
    c_xml.setDbRuleArray("#userinfo_id|email_hash# % 1024 / 32");
    c_xml.setTbRuleArray("#userinfo_id|email_hash# % 1024 % 32");
    c_xml.setSqlPath("classpath:properties/sqlFile.properties");
    /* 执行配置文件输出 */
    System.out.println(c_xml.createCoreXml(new File("e:/shark-context.xml")) ? "create success" : "create fial");
}
```