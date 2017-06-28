问题一：”mvn install”发现未找到相应版本的jar包，查看远程仓库，存在，本地repository在版本号下无jar包。
分析：使用maven工具有一个很重要的配置文件settings.xml，默认在maven安装路径（M2_HOME）的下的conf文件夹下，这是全局的配置文件。此外，不同还可以在~/.m2/文件夹下，设置user级（{user.home}）的配置文件。这个配置文件包括指定本地仓库路径、服务器信息等。user级的配置文件优先级高于全局配置。
原因及解决：user级的配置文件默认设置在”~/.m2/settings.xml”文件中，需要将配有内部仓库地址的setting文件cp到该目录下，由于粗心，误将文件命名未setting.xml，导致maven找不到user级的配置文件，而使用全局配置。
 
问题二：maven的间接依赖，依赖传递导致的相同包版本冲突问题

如图，项目中同时load两个版本的spring相关包，此时如何排包？
方法一：
使用IDEA的maven projects视图，查看依赖关系，找到需要排除的包，右键exclude，IDEA会自动修改你的pom文件，排除该包，如下：

具体步骤可参考：http://www.myexception.cn/open-source/1497379.html
方法二：
maven的依赖仲裁原则：1. 最短路径优先；2. 路径相同，则先声明者优先；3. 覆写优先，子POM内声明的优先于父POM中的依赖
由于我们遇到的问题场景，属于第二种，路径相同的情况，3.1.1版本路径：pms->mtthrift->mns-invoker->spring-context；3.1.2版本路径：pms->pms-controller->pms-platform->spring-context。可以通过把所需版本的依赖移到依赖树的顶层，即在父pom文件引入依赖，使其路径最短。
方法三：
在父pom文件中，使用版本管理元素dependencyManagement，指定版本号，这样所有子模块添加依赖时，无需再指定version，maven会向父pom的dependencyManagement需找声明的版本。dependencyManagement里只是声明依赖，并不实现引入，因此子项目需要显式的声明需要用的依赖。