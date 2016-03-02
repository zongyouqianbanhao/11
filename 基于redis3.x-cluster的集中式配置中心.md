1、首先需要引入shark-core-v.jar、shark-resource-v.jar等相关构件；

2、然后shark的配置文件定义如下：
```Xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:task="http://www.springframework.org/schema/task" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans
			http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
			http://www.springframework.org/schema/aop 
			http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
			http://www.springframework.org/schema/context 
			http://www.springframework.org/schema/context/spring-context-3.2.xsd 
			http://www.springframework.org/schema/task 
			http://www.springframework.org/schema/task/spring-task-3.2.xsd">
	<task:annotation-driven />
	<aop:aspectj-autoproxy proxy-target-class="true" />
	<context:component-scan base-package="com.gxl.shark">
		<context:include-filter type="annotation"
			expression="org.aspectj.lang.annotation.Aspect" />
	</context:component-scan>
	<context:property-placeholder location="classpath*:*.properties" />
	<bean class="com.gxl.shark.resources.conn.RedisConnectionManager" 
		init-method="init">
		<constructor-arg index="0" value="${redis.key}" />
		<constructor-arg index="1" ref="jedisCluster" />
	</bean>
	<!-- 配置redis连接池 -->
	<bean id="jedisPoolConfig" class="org.apache.commons.pool2.impl.GenericObjectPoolConfig">
		<property name="maxTotal" value="${redis.pool.maxTotal}" />
		<property name="minIdle" value="${redis.pool.minIdle}" />
		<property name="maxIdle" value="${redis.pool.maxIdle}" />
		<property name="maxWaitMillis" value="${redis.pool.maxWait}" />
		<property name="testOnBorrow" value="${redis.pool.testOnBorrow}" />
		<property name="testOnReturn" value="${redis.pool.testOnReturn}" />
	</bean>
	<bean id="jedisCluster" class="redis.clients.jedis.JedisCluster">
		<constructor-arg index="0">
			<list>
				<bean class="redis.clients.jedis.HostAndPort">
					<constructor-arg index="0" value="${redis.host}" />
					<constructor-arg index="1" value="${redis.port}" />
				</bean>
			</list>
		</constructor-arg>
		<constructor-arg index="1" ref="jedisPoolConfig" />
	</bean>
	<bean id="jdbcTemplate" class="com.gxl.shark.core.shard.SharkJdbcTemplate">
		<constructor-arg name="isShard" value="true" />
		<property name="dataSource" ref="dataSourceGroup" />
	</bean>
	<!-- 引用一个缺省数据源 -->
	<bean id="dataSourceGroup" class="com.gxl.shark.core.config.SharkDatasourceGroup">
		<property name="targetDataSources">
			<map key-type="java.lang.Integer">
				<entry key="0" value-ref="dataSource1" />
			</map>
		</property>
	</bean>
	<bean id="dataSource1" class="com.mchange.v2.c3p0.ComboPooledDataSource"
		destroy-method="close">
		<property name="user" value="${name}" />
		<property name="password" value="${password}" />
		<property name="jdbcUrl" value="jdbc:mysql://120.24.75.22:3306/db_0000" />
		<property name="driverClass" value="${driverClass}" />
		<property name="initialPoolSize" value="${initialPoolSize}" />
		<property name="minPoolSize" value="${minPoolSize}" />
		<property name="maxPoolSize" value="${maxPoolSize}" />
		<property name="maxStatements" value="${maxStatements}" />
		<property name="maxIdleTime" value="${maxIdleTime}" />
	</bean>
</beans>
```

大家需要注意，尽管shark所持有相关的数据源以及sharding规则都配置在redis中，但是本地配置文件中仍然需要持有一个缺省数据源，当然这并不影响shark后续从资源中心拉到最新的资源数据进行更替。

上述配置文件中参数${redis.key}中需要制定redis的key，数据结构为“key-value(version,resource)”，value中最重要的就是版本号version，当配置中心发生改变时，需要修改当前版本号，shark的配置中心客户端会根据版本差异判断是否是更新了配置中心的数据。

3、一旦使用zookeeper作为配置中心后，程序中就不要在使用下述代码获取SharkJdbcTemplate，如下所示：
```Java
@Resource
private SharkJdbcTemplate jdbcTemplate;
```

而是需要使用如下形式，确保程序不再持有的是之前引用，如下所示：
```Java
SharkJdbcTemplate jdbcTemlate = GetJdbcTemplate.getSharkJdbcTemplate();
```

在此大家需要注意，数据源务必要配置参数destroy-method="close"，否则监听到配置中心发生变化后，将不会释放之前持有的数据库连接；