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
  
- 问题8：同时运行多个Tomcat出错。   
  解决：首先是端口，jmx等。其次，deployment下面的部署地址要改，在idea中，如果application context的地址如果是/。说明部署到Tomcat的webapps下的root目录下。因此多个项目同时启动Tomcat时，必须有各自的名字。
  
- 问题9：中途加了pojo，结果编译不了。CTRL+shift+F9  
 
- 问题10：不同工程的相互引用。一定要注意包名的定义，如sso工程下的包，至少也要来一个sso.dao不能直接就dao，比如sso引用了manger下的maper工程，引用包的时候，除了在pom.xml中添加依赖，在引用包需要注意区分自己的dao与mapper的dao包。
  
  ----------------------
  
- 资料：在idea中打包jar：https://www.jianshu.com/p/55c0a0932be1    
- 资料：构建聚合工程：https://blog.csdn.net/kingcat666/article/details/79071802    
- 资料：idea部署Tomcat：https://segmentfault.com/a/1190000003827831  。
- 资料：jsonp的初见：https://blog.csdn.net/hansexploration/article/details/80314948
  
   
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

第五步。图片服务器。我全程在Windows上处理，没有Linux。首先是配置Nginx（安装完后需要在配置文件加入一个服务，并自定义其根目录），接着在Windows开启FTP服务，指明其服务的目录为该根目录，该服务用于文件传输，将文件传输到Nginx服务器的文件目录下（如果在Linux下，老师的vsftp是一个安全、高速、稳定的FTP服务器，就不用开启，直接安装）。电脑必须有密码，不然在代码中无法登陆这个站点。   
总的来说，相当于2台设备。一台客户端a，一台服务器b。其中b有图片服务器Nginx，a为了上传图片到b的这个nginx服务器，使用ftp服务，因此b需要开启ftp服务供a访问。现在a开始上传文件，文件通过ftp服务上传到ftp指定的位置，而这个位置与nginx服务器存放文件的位置一样（不然要nginx干嘛呢）,以供下次访问图片时Nginx来进行调度。

第六步。规格参数的存储。在xu_tmall里，是用的方案一的存储方式。而方案二，是将整个规格的内容写成text来存储。取出来后转成json格式来用。我的理解：在新建商品分类的时候，生成tb_item_param规格参数模板表。新增商品的时候，根据tb_item_param填写参数值，生成商品的规格参数表。



后面的：   
1.首页左侧有一个商品分类。本来是ajax向protal请求，然后protal调用rest的服务，但是ajax直接向rest工程请求数据，返回的是json数据，而且此时是在protal工程中发布的网页，因此客户端现在请求的数据，是通过rest工程中的服务获取的，出现跨域问题，因此需要jsonp的方式来获取json数据。   
2.之后是首页轮播图。浏览器请求protal，然后protal调用rest的服务，这个跨域进行调用controller，选择httpclient来模拟客户端，来请求服务。就是现在是2个工程，一个工程想掉另一个的内容，就是通过httpclient来模拟调用。（1.2为啥这么调服务就看文档就行了）。     
3.solr的使用：总结：绝对不要到博客里找解决方案，在版本不同的前提下。我用的solr7.6，不用在Tomcat下部署，直接按照文档就能使用。同时中文分析器，网上百度查找你就玩了，Google搜索对应版本的中文分析器不然报错解决不了。   
4.solr工程：首先需要查询所有MySQL数据库中的内容，导入solr进行建立索引库，因此需要对多表查询数据，所以自己写DAO，包括接口以及mapper.xml，来动态生成DAO。接着，现在需要导入solr建立索引库，因此将DAO查询到的数据进行遍历插入索引库（这里需要用solrj，视频中的类过时了，在spring-solr.xml配置HttpsolrClient对象，在DAO查到数据的时候注入solrClient对象，进行插入操作）

5.**redis集群**。首先虚拟机不要下载vmare了，就用window10自带的。其次，按照redis文档中说的配置好conf文件，但是如果protected-method不改为no的话，显示连接不上，为了连接成功，且不改变bind的ip属性值（每次改6个conf文件不好玩），我改了保护模式protected-method，（应该可以通过设置密码的方式，但文档没找到），并且创建集群的时候我直接在redis的服务器下用的ip地址，而不是127.0.0.1，不然jedis连不上。所以重新构建集群使用：./redis-trib.rb create --replicas 1 192.168.0.104:7001 192.168.0.104:7002  192.168.0.104:7003  192.168.0.104:7004  192.168.0.104:7005  192.168.0.104:7006。另外让集群停止的方式，暂时没找到命令，选择停止redis服务后，将启动服务时生成的文件全删掉，只留下配置文件conf。重新启动。
