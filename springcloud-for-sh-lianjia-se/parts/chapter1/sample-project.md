### 项目目标：Mini楼盘字典
基于Spring Cloud 实现一个简化的楼盘字典。

### 环境以及约定
本书涉及的代码基于Eclipse 4.5编写 、JDK8 编译、Maven(3.X.X)构建。

servlet-api 版本为3.0以上（Tomcat 7.x以上已支持）。

Spring Cloud 版本为：Brixton.M4，我们内部支持Jar包lorik-core版本号为：1.1.0，lorik-spi-view版本号： 2.1.0。

样本项目位置: https://github.com/bookdao/samples.git [[GitHub BookDao](https://github.com/bookdao/samples.git "books sample")]的**springcloud-for-sh-lianjia-se**目录下。

代码格式（Code Style)我们推荐 [Google Code Style](https://github.com/google/styleguide/ "Google Code Style")，基于Eclipse和Intellij IDE开发的同学请下载各自的配置。


基于Spring Cloud 开发和传统的Spring Framework(MVC)最直观上的不同，是不需要xml配置文件，包括web.xml也不必有。所有Bean的配置全在代码里，包括Servlet、Filter、Servlet Listener。
