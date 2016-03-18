1、源码的下载与编译

由于shark目前并没有提交到maven的中央仓库，因此开发人员只能够从https://github.com/gaoxianglong/shark.git 中下载shark的源码，使用命令package -Dmaven.test.skip=true -Pdev将其编译为jar构件，项目中我们在大部分情况下只需用到shark-core包，如果希望使用基于zookeeper或者redis的集中式资源配置中心，那么还需要在工程中引用shark-resource包。

2、配置sharding规则
```Xml
<import resource="datasource1-context.xml" />
<aop:aspectj-autoproxy proxy-target-class="true" />
<!-- 自动扫描 -->
<context:component-scan base-package="com.sharksharding">
	<context:include-filter type="annotation"
		expression="org.aspectj.lang.annotation.Aspect" />
</context:component-scan>
<!-- 片名连续的库内分片配置 -->
<bean id="jdbcTemplate" class="com.sharksharding.core.shard.SharkJdbcTemplate">
	<constructor-arg name="isShard" value="true" />
	<property name="dataSource" ref="dataSourceGroup" />
	<property name="wr_index" value="r0w0" />
	<!-- 分片模式,false为库内分片模式,true为一库一表分片模式 -->
	<property name="shardMode" value="false" />
	<!-- 片名是否连续,true为片名连续,false为非片名连续 -->
	<property name="consistent" value="true" />
	<property name="dbRuleArray" value="#userinfo_test_id|email_hash# % 4 / 2" />
	<property name="tbRuleArray" value="#userinfo_test_id|email_hash# % 4 % 2" />
	<property name="tbSuffix" value="_0000" />
</bean>
<bean id="dataSourceGroup" class="com.sharksharding.core.config.SharkDatasourceGroup">
	<property name="targetDataSources">
		<map key-type="java.lang.Integer">
			<entry key="0" value-ref="dataSource1" />
			<entry key="1" value-ref="dataSource2" />
		</map>
	</property>
</bean>
<bean class="com.gxl.shark.sql.PropertyPlaceholderConfigurer">
	<constructor-arg name="path" value="classpath:sql.properties" />
</bean>
```

关于其他sharding规则、以及参数含义请见wiki。

3、配置数据源
```Xml
<!-- 加载本地数据源配置信息 -->
<context:property-placeholder location="classpath*:druid-jdbc.properties" />
<bean id="dataSource1" class="com.alibaba.druid.pool.DruidDataSource"
	init-method="init" destroy-method="close">
	<property name="name" value="dataSource1" />
	<property name="username" value="${um.username}" />
	<property name="password" value="${um.password}" />
	<property name="url" value="jdbc:mysql://ip:3306/db_0000" />
	<property name="initialSize" value="${um.initialSize}" />
	<property name="minIdle" value="${um.minIdle}" />
	<property name="maxActive" value="${um.maxActive}" />
	<!-- 是否缓存preparedStatement，也就是PSCache，在mysql下建议关闭，分库分表较多的数据库，建议配置为false -->
	<property name="poolPreparedStatements" value="${um.poolPreparedStatements}" />
	<property name="maxOpenPreparedStatements" value="${um.maxOpenPreparedStatements}" />
	<!-- 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能 -->
	<property name="testOnBorrow" value="${um.testOnBorrow}" />
	<!-- 归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能 -->
	<property name="testOnReturn" value="${um.testOnReturn}" />
	<!-- 建议配置为true，不影响性能 -->
	<property name="testWhileIdle" value="${um.testWhileIdle}" />
	<property name="filters" value="${um.filters}" />
	<property name="connectionProperties" value="${um.connectionProperties}" />
	<!-- 合并多个DruidDataSource的监控数据 -->
	<property name="useGlobalDataSourceStat" value="${um.useGlobalDataSourceStat}" />
	<!-- 保存DruidDataSource的监控记录到日志中 -->
	<property name="timeBetweenLogStatsMillis" value="${um.timeBetweenLogStatsMillis}" />
</bean>
<bean id="dataSource2" class="com.alibaba.druid.pool.DruidDataSource"
	init-method="init" destroy-method="close">
	<property name="name" value="dataSource2" />
	<property name="username" value="${um.username}" />
	<property name="password" value="${um.password}" />
	<property name="url" value="jdbc:mysql://ip:3306/db_0001" />
	<property name="initialSize" value="${um.initialSize}" />
	<property name="minIdle" value="${um.minIdle}" />
	<property name="maxActive" value="${um.maxActive}" />
	<property name="poolPreparedStatements" value="${um.poolPreparedStatements}" />
	<property name="maxOpenPreparedStatements" value="${um.maxOpenPreparedStatements}" />
	<property name="testOnBorrow" value="${um.testOnBorrow}" />
	<property name="testOnReturn" value="${um.testOnReturn}" />
	<property name="testWhileIdle" value="${um.testWhileIdle}" />
	<property name="filters" value="${um.filters}" />
	<property name="connectionProperties" value="${um.connectionProperties}" />
	<property name="useGlobalDataSourceStat" value="${um.useGlobalDataSourceStat}" />
	<property name="timeBetweenLogStatsMillis" value="${um.timeBetweenLogStatsMillis}" />
</bean>
```

关于其他sharding规则、以及参数含义请见wiki。

4、使用SharkJdbcTemplate
```Java
@Resource
private SharkJdbcTemplate jdbcTemplate;
```

注意：

除了批量读/写的batch()方法外，SharkJdbcTemplate支持Spring JdbcTemplate的所有方法。