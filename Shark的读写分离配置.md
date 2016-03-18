com.sharksharding.core.shard.SharkJdbcTemplate是Shark提供的一个Jdbc模板，它继承自Spring的JdbcTemplate。简单来说，SharkJdbcTemplate几乎支持JdbcTemplate的所有原生方法(除批量操作外)。对于开发人员而言，只需要将Spring的JdbcTemplate替换为Shark的SharkJdbcTemplate即可，除此之外，业务逻辑代码中不再有任何的侵入。

一般来说，数据库的主从配置，既可以一主一从，也可以一主多从，但目前Shark仅支持一主一从。Shark一主一从读写分离配置，如下所示：
```Xml
<import resource="datasource1-context.xml" />
<aop:aspectj-autoproxy proxy-target-class="true" />
<!-- 自动扫描 -->
<context:component-scan base-package="com">
	<context:include-filter type="annotation"
		expression="org.aspectj.lang.annotation.Aspect" />
</context:component-scan>
<!-- 读写分离配置 -->
<bean id="jdbcTemplate" class="com.sharksharding.core.shard.SharkJdbcTemplate">
	<constructor-arg name="isShard" value="false" />
	<property name="dataSource" ref="dataSourceGroup" />
	<property name="wr_index" value="r1w0" />
</bean>
<bean id="dataSourceGroup" class="com.sharksharding.core.config.SharkDatasourceGroup">
	<property name="targetDataSources">
		<map key-type="java.lang.Integer">
			<entry key="0" value-ref="dataSource1" />
			<entry key="1" value-ref="dataSource2" />
		</map>
	</property>
</bean>
<bean class="com.sharksharding.sql.PropertyPlaceholderConfigurer">
	<constructor-arg name="path" value="classpath:sql.properties" />
</bean>
```

上述程序实例中，SharkDatasourceGroup就是一个用于管理多数据源的Group，它继承自Spring提供的AbstractRoutingDataSource，并充当了动态数据源层的角色，由此基础之上实现DBRoute。我们可以看见，在标签中，key属 性指定了数据源索引，而value-ref属性指定了数据源，通过这种简单的键值对关系就可以明确的切换到具体的目标数据源上。

在SharkJdbcTemplate中，isShard属性实际上就是一个Sharding开关，缺省为false，也就意味着缺省是没有开启分库分表的，那么在不Sharding的情况下，我们依然可以使用Kratos来完成读写分离操作。在wr_index属性中定义了读写分离的起始索引， 也就是说，我们有多少个主库，就一定需要有多少个从库，比如主库有1个，从库也应该是1个，因此SharkDatasourceGroup中持有的数据源个数就应该一共是2个，索引从0-1，如果主库的索引为0，那么从库的索引就应该为1，也就是“r1w0”。当配置完成后，一旦Shark监测到执行的操作为写操作时，就会自动切换为主库的数据源，而当操作为读操作的时候，就会自动切换为从库的数据源，从而实现一主一从的读写分离操作。