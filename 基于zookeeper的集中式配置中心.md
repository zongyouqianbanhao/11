1、首先需要引入shark-core-v.jar、shark-resource-v.jar等相关构件；

2、然后shark的配置文件定义如下：
```Xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
			http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
			http://www.springframework.org/schema/aop 
			http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
			http://www.springframework.org/schema/context 
			http://www.springframework.org/schema/context/spring-context-3.2.xsd">
	<aop:aspectj-autoproxy proxy-target-class="true" />
	<context:component-scan base-package="com.gxl.shark">
		<context:include-filter type="annotation"
			expression="org.aspectj.lang.annotation.Aspect" />
	</context:component-scan>
	<context:property-placeholder location="classpath*:*.properties" />
	<bean class="com.gxl.shark.resources.conn.ZookeeperConnectionManager"
		init-method="init">
		<constructor-arg index="0" value="${address}" />
		<constructor-arg index="1" value="${session.timeout}" />
		<constructor-arg index="2" value="${nodepath}" />
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

大家需要注意，尽管shark所持有相关的数据源以及sharding规则都配置在zookeeper中，但是本地配置文件中仍然需要持有一个缺省数据源，当然这并不影响shark后续从资源中心拉到最新的资源数据进行更替。

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