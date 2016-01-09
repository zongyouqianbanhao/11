```Java
/**
 * 生成druid数据源配置文件
 * 
 * @author gaoxianglong
 */
public @Test void testCreateDruidXml() {
    CreateDruidXml c_xml = new CreateDruidXml();
    /* 是否控制台输出生成的配置文件 */
    c_xml.setIsShow(true);
    /* 数据源索引起始 */
    c_xml.setDataSourceIndex(1);
    /* 配置分库分片信息 */
    c_xml.setDbSize("16");
    /* false为懒加载模式，反之启动时开始初始化数据源 */
    c_xml.setInit_method(true);
    c_xml.setTbSuffix("_0000");
    /* 生成数据源信息 */
    c_xml.setUsername("${username}");
    c_xml.setPassword("${password}");
    c_xml.setUrl("jdbc:mysql://ip1:3306/db");
    c_xml.setInitialSize("${initialSize}");
    c_xml.setMinIdle("${minIdle}");
    c_xml.setMaxActive("${maxActive}");
    c_xml.setPoolPreparedStatements("${poolPreparedStatements}");
    c_xml.setMaxOpenPreparedStatements("${maxOpenPreparedStatements}");
    c_xml.setTestOnBorrow("${testOnBorrow}");
    c_xml.setTestOnReturn("${testOnReturn}");
    c_xml.setTestWhileIdle("${testWhileIdle}");
    c_xml.setFilters("${filters}");
    c_xml.setConnectionProperties("${connectionProperties}");
    c_xml.setUseGlobalDataSourceStat("${useGlobalDataSourceStat}");
    c_xml.setTimeBetweenLogStatsMillis("${timeBetweenLogStatsMillis}");
    /* 执行配置文件输出 */
    Assert.assertTrue(c_xml.createDatasourceXml(new File("e:/dataSource-context.xml")));
}
```
```Java
/**
 * 生成druid数据源配置文件
 * 
 * @author gaoxianglong
 */
public @Test void testCreateDruidMSXml() {
    CreateDruidXml c_xml = new CreateDruidXml();
    /* 是否控制台输出生成的配置文件 */
    c_xml.setIsShow(true);
    /* 数据源索引起始 */
    c_xml.setDataSourceIndex(1);
    /* 配置分库分片信息 */
    c_xml.setDbSize("16");
    /* false为懒加载模式，反之启动时开始初始化数据源 */
    c_xml.setInit_method(true);
    c_xml.setTbSuffix("_0000");
    /* 生成master数据源信息 */
    c_xml.setUsername("${username}");
    c_xml.setPassword("${password}");
    c_xml.setUrl("jdbc:mysql://ip1:3306/db");
    c_xml.setInitialSize("${initialSize}");
    c_xml.setMinIdle("${minIdle}");
    c_xml.setMaxActive("${maxActive}");
    c_xml.setPoolPreparedStatements("${poolPreparedStatements}");
    c_xml.setMaxOpenPreparedStatements("${maxOpenPreparedStatements}");
    c_xml.setTestOnBorrow("${testOnBorrow}");
    c_xml.setTestOnReturn("${testOnReturn}");
    c_xml.setTestWhileIdle("${testWhileIdle}");
    c_xml.setFilters("${filters}");
    c_xml.setConnectionProperties("${connectionProperties}");
    c_xml.setUseGlobalDataSourceStat("${useGlobalDataSourceStat}");
    c_xml.setTimeBetweenLogStatsMillis("${timeBetweenLogStatsMillis}");
    /* 执行配置文件输出 */
    Assert.assertTrue(c_xml.createDatasourceXml(new File("e:/masterDataSource-context.xml")));
    /* 生成slave数据源信息 */
    c_xml.setDataSourceIndex(17);
    c_xml.setDbSize("16");
    c_xml.setUsername("${username}");
    c_xml.setPassword("${password}");
    c_xml.setUrl("jdbc:mysql://ip2:3306/db");
    /* 执行配置文件输出 */
    Assert.assertTrue(c_xml.createDatasourceXml(new File("e:/slaveDataSource-context.xml")));
}
```