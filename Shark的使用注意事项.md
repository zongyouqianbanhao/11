- 不支持强一致性的分布式事务，但可以在业务层依赖MQ、异步操作的方式实现事物，保证最终一致性即可；
- 不支持多表查询，所有多表查询sql，务必全部打散为单条sql逐条执行；
- 不建议使用一些数据库统计函数、Order by语句等；
- sql语句的第一个参数务必是路由条件；
- 不支持sql语句中出现数据库别名；
- 路由条件必须是整数类型；
- 在连续分片模式下，子表后缀为符号"_"+4位整型，比如“tb_0001”——"tb_1024"；

## SQL编写注意：
耦合在业务代码中的sql语句：insert into userinfo(userinfo_id,username) values(?,?)，这种写法是不支持的，约定写法必须为：insert into userinfo(userinfo_id,username) values("+ uid +",?)，也就是说，第一个参数不允许是占位符，而必须是实际的值，其实第一个参数就是路由条件。

在某些情况下，我们可能不太希望将sql耦合在我们的业务代码中，这种情况下，Kratos提供有类似于Mybatis的做法，将sql定义在配置 文件中。Kratos采用的做法很简单，sql信息定义在properties文件中，采用key-value的方式，key建议定义为持久层的方法名 称。sql写法为：addUser=insert into userinfo(uid,name) values(?,?)，配置方式，如下所示：
```Xml
<bean class="com.sharksharding.sql.PropertyPlaceholderConfigurer">
    <constructor-arg name="path" value="classpath:sql.properties" />
</bean>
```

除了可以加载classpath下的sql配置文件外，还允许加载文件路径下的sql配置文件。使用方式如下所示：
```Java
@Resource
private PropertyPlaceholderConfigurer property;

@Override
public void addUser(long routeKey) {
    final String SQL = property.getSql("addUser", routeKey);
    kJdbcTemplate.update(SQL);
}
```

上述程序示例中，getSql()方法的第一个参数就是定义在properties中的key，而routeKey就是路由条件。 
