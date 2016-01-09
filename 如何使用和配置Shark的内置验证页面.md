Shark提供了一个内置验证页面用于对执行后的sql进行验证。

内置验证页面(QueryViewServlet)是一个标准的javax.servlet.http.HttpServlet，需要配置在你web应用中的WEB-INF/web.xml中，如下所示：
```Xml
<!-- Shark的sql内置验证页面 -->
<servlet>
	<servlet-name>queryViewServlet</servlet-name>
	<servlet-class>com.gxl.shark.util.web.http.QueryViewServlet</servlet-class>
</servlet>
<servlet-mapping>
	<servlet-name>queryViewServlet</servlet-name>
	<url-pattern>/shark/*</url-pattern>
</servlet-mapping>
```

根据web.xml配置中的url-pattern来访问内置验证页面，如果是上面的配置，内置验证页面的首页是/shark/index.html。
内置验证页面如下所示:
![](http://dl.iteye.com/upload/picture/pic/134661/a1104bf1-d769-3eaf-a90a-415f4cdc4291.png)
![](http://dl.iteye.com/upload/picture/pic/134663/8968a56a-facd-376b-99f4-9eb949c5ca63.png)

注意：考虑到测试环境、生产环境的安全性，内置验证页面缺省是只支持query操作，而其他写入操作将不支持。