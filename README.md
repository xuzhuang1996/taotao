# taotao
ssm、taotao商城
ssm入门建议：https://www.imooc.com/u/2145618/courses?sort=publish 秒杀工程。这个老师讲的非常好。




- 问题1：父项目的pom.xml定义所需依赖的jar包的版本即可，子项目只需要简单引用。但是尝试多次，子项目加载时总是报如上缺少jar包版本信息的错误。  
解决：设置父项目的依赖jar包时外层少加了这个依赖版本管理的标签<dependencyManagement>。也就是parent这个项目的依赖需要加上这个dependencyManagement标签。
- 问题2：逆向工程中配置的数据库信息，一直报错：时区问题。
  解决：https://blog.csdn.net/qq_36350532/article/details/81534812   
- 问题3：不能直接新建Java类文件。   
  解决：idea中Java类只能在指定类型的文件夹下新建。打开File下的Project Structure，需要指定该文件夹为source类型。
- 问题4：log4j:WARN Please initialize the log4j system properly.报错。   
  解决：https://stackoverflow.com/questions/9691456/log4j-spring-mvc-no-appenders-could-be-found-for-logger 没有给日志配置属性。
- 问题5：org.apache.jasper.servlet.TldScanner.scanJars At least one JAR was scanned for TLDs yet contained no TLDs报错。   
  解决：在Tomcat安装目录下的配置文件logging.properties文件加入org.apache.jasper.level = FINEST  
- 问题6：项目启动出错：org.apache.catalina.core.StandardContext.startInternal One or more listeners failed to start。  
  解决：https://stackoverflow.com/questions/48639816/tomcat-one-or-more-listeners-failed-to-start 这样就可以看到错误日志了。   
- 问题7：nested exception is java.lang.NoClassDefFoundError: com/fasterxml/jackson/databind/exc/InvalidDefinitionException   
  解决：spring版本问题，我做的时候是spring5.0以上.视频中是5.0以下。我将jackson.version版本改成2.9.4就可以了，之前的2.4.2不行。报错。
  
  ----------------------
  
- 资料：在idea中打包jar：https://www.jianshu.com/p/55c0a0932be1    
- 资料：构建聚合工程：https://blog.csdn.net/kingcat666/article/details/79071802    
- 资料：idea部署Tomcat：https://segmentfault.com/a/1190000003827831  。
  
   
第一步：逆向工程：首先新建maven工程，在pom.xml中添加mybatis-generator-core、mysql-connector-java依赖。接着在generatorConfig.xml中配置数据库信息，表信息。我选择一步到位，直接生成在..\taotaomanager\taotao-manager-pojo\src\main\java\位置。最后需要强调的是：需要在mapper这个工程添加includes等节点，不添加节点mybatis的mapper.xml文件都会被漏掉。pojo不用是因为它没有xml.          
第二步：ssm整合。spring容器包括DAO、service为父容器，而springMVC容器装的是controller为子容器。分开扫描保证相应包加载在相应容器中。      
- DAO层。使用mybatis框架。与数据库进行直接联系。
- Service层。加载service包。本来事务可以在里面配，选择在单独在一个xml配置。
- 表现层，就是springMVC.xml。

第三步。开始编码。页面切换控制器;showIndex函数返回“index”。根据springMVC中的配置。进入index.jsp页面
````
    <bean
            class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/" />
        <property name="suffix" value=".jsp" />
    </bean>
````

第四步，在秒杀项目中，控制层与前台交互，需要新建数据结构来统一。而这个交互的数据结构都在用，因此在common工程中新建添加pojo包。保存交互用的数据结构。在service层写数据库交互的逻辑，在控制层调用这些逻辑，并进行简单请求响应的分支处理。   
首先是getItemList这个方法。**流程**：简单说，浏览器输出地址前台请求，点击那个查询商品，而attributes:{'url':'item-list'}，也就是请求item-list，然后dispatcher传给pageController中{page}，返回item—list，到视图解析器找item-list的页面。于是找到了web-inf下的item—list.jsp页面，而该jsp的参数为data-options="url:'/item/list'，进行请求/item/list。   
接着dispatcher根据地址，分发到控制器，pageController由于只处理/{}的情况，因此不处理，而另一个控制器ItemController中正好可以处理这个URL，进入getItemList方法。然后返回EUDataGridResult。由jq的easyui获取处理，之前jsp中是通过这种方式拿值${sk.startTime}。现在就是easyui处理，暂时不研究这个(IDEA启动Tomcat老自动打开的那个页面，在配置下的server下的URL更改，而决定具体部署为那个网址，在Deployment下面的ApplicationContext)。
