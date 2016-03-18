Spring Jdbc缺省并没有提供类似于MyBatis那样的Sql独立配置功能，而是采用与逻辑代码耦合的方式。如果你希望解耦，那么Shark提供有com.sharksharding.sql.PropertyPlaceholderConfigurer类。

在配置文件中配置PropertyPlaceholderConfigurer，如下所示：
```Xml
<bean class="com.sharksharding.sql.PropertyPlaceholderConfigurer">
    <constructor-arg name="path"
        value="classpath:properties/sql.properties" />
</bean>
```

可以支持从classpath下加载sql配置文件，同时也支持从绝对路径。当成功配置PropertyPlaceholderConfigurer后，具体的使用方式，如下所示：

```Java
final String SQL = property.getSql("insertUserInfo", userInfo.getUserinfo_id());
```

PropertyPlaceholderConfigurer提供的getSql(String key, long routeKey)方法，用于支持从“.properties”文件中获取出指定的一条sql，典型的key-value形式。key建议与持久层方法名 保持一致，而routeKey则为路由条件。