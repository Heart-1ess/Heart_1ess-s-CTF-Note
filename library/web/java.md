* # Java Web
* ### Java Web目录结构

```
WebContent                          (站点根目录)
|---META-INF                        (META-INF文件夹)
|    |---MANIFEST.MF                (MANIFEST.MF配置清单文件)
|---WEB-INF                         (WEB-INF文件夹)
|    |---web.xml                    (站点配置web.xml)
|    |---lib                        (第三方库文件夹)
|    |    |---*.jar                 (程序需要的jar包)
|    |---classes                    (class文件目录)
|        |---...*.class             (class文件)
|---<userdir>                       (自定义的目录)
|    |---*.jsp,*.js,*.css           (自定义的资源文件)
|---<userfiles>                     (自定义的资源文件)
```

* ### 敏感目录

```
/WEB-INF/web.xml：Web应用程序配置文件，描述了 servlet 和其他的应用组件配置及命名规则
/WEB-INF/classes/：含了站点所有用的 class 文件，包括 servlet class 和非servlet class，他们不能包含在 .jar文件中
/WEB-INF/lib/：存放web应用需要的各种JAR文件，放置仅在这个应用中要求使用的jar文件,如数据库驱动jar文件
/WEB-INF/src/：源码目录，按照包名结构放置各个java文件
/WEB-INF/database.properties：数据库配置文件
```

* ### Servlet相关

```
<servlet-class>  
这个就是指向我们要注册的servlet 的类地址, 要带包路径

<servlet-mapping>  
是用来配置我们注册的组件的访问路径,里面包括两个节点
一个是   <servlet-name>  这个要与前面写的servlet那么一致
另一个是  <url-pattern>  配置这个组件的访问路径

<servlet-name> 这个是我们要注册servlet的名字,一般跟Servlet类名有关


举个例子
<servlet>
    <servlet-name>LoginServlet</servlet-name>
    <servlet-class>com.breeze.servlet.LoginServlet</servlet-class>
  </servlet>
```

直接访问失败后可以将GET请求换为POST请求尝试



