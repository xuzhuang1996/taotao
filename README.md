# taotao
ssm、taotao商城
ssm入门建议：https://www.imooc.com/u/2145618/courses?sort=publish 秒杀工程。这个老师将的非常好。

- 问题1：父项目的pom.xml定义所需依赖的jar包的版本即可，子项目只需要简单引用。但是尝试多次，子项目加载时总是报如上缺少jar包版本信息的错误。  
解决：设置父项目的依赖jar包时外层少加了这个依赖版本管理的标签<dependencyManagement>。也就是parent这个项目的依赖需要加上这个dependencyManagement标签。
- 问题2：逆向工程中配置的数据库信息，一直报错：时区问题。
  解决：https://blog.csdn.net/qq_36350532/article/details/81534812 直接改了MySQL的时区
- 资料：在idea中打包jar：https://www.jianshu.com/p/55c0a0932be1    
- 资料：构建聚合工程：https://blog.csdn.net/kingcat666/article/details/79071802    
- 资料：idea部署Tomcat：https://segmentfault.com/a/1190000003827831  。
- 资料：idea的mybaits插件逆向工程：https://blog.csdn.net/qq_23703157/article/details/78681088 (收费。算了)
