# taotao
ssm、taotao商城
ssm入门建议：https://www.imooc.com/u/2145618/courses?sort=publish 秒杀工程。这个老师将的非常好。
问题1：父项目的pom.xml定义所需依赖的jar包的版本即可，子项目只需要简单引用。但是尝试多次，子项目加载时总是报如上缺少jar包版本信息的错误。  
解决：设置父项目的依赖jar包时外层少加了这个依赖版本管理的标签<dependencyManagement>。也就是parent这个项目的依赖需要加上这个。
