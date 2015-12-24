###读者受众
上海链家SE团队有实际Spring项目开发经验的工程师<br>
下面我们简要介绍下[Spring Boot](http://projects.spring.io/spring-boot/ "Spring Boot Project") 和 [Spring Cloud](http://http://projects.spring.io/spring-cloud/ "Spring Cloud Project")。
###Spring Boot
Spring Boot 使得创建独立的、基于Spring Framework的生产级应用更加容易。在此之前，基于Spring的Web应用（大多数Spring WebMVC应用）必须依赖一个外部的Servlet容器、部署之后才能运行，如下：

* JVM
* Servlet容器
* ROOT.war

现在，基于Spring Boot，可以轻松的将Spring Web项目构建为普通的jar，然后直接运行。我们仅依赖JVM，如下：

* JVM
* youapp.jar

在分布式、微服务的架构中，这种构建方式非常灵活并易部署。无论采用何种部署方式（docker or jenkins or 其他 CI 工具)、多种运行环境（开发、测试、生产），都将以一种简单、一致的方式运行我们的Web程序。

除了构建方式之外，Spring Boot 还提供了众多开发支持（Spring Team总会为你做更多）以简化项目开发。

###Spring Boot 特性
* 创建独立运行的、基于Spring的应用（Just Run !)
* 内嵌的Servlet容器支持（Tomcat,Jetty,Undertow等，无需部署War包）
* 提供一些固定的Parent POMs来简化你的Maven配置（you'll see.)
* 尽可能的自动配置Spring(有时省心，有时苦恼）
* 无需XML配置，没有代码生成。
* 提供了一些生产环境即用的特性，比如，metrics,heath check 和外部配置支持。
* .....

### Spring Cloud 
Spring Cloud 基于 Spring Boot，封装了一系列Netflix OSS 组件，提供了典型分布式场景中开箱即用的特性:   

* 分布式版本配置 ( **Spring Cloud Config**)
* 服务注册与发现 ( **Netflix Eureka**)
* 请求路由 ( **Netflix Zuul**)
* 服务间调用 ( **Netflix Feign**)
* 负载均衡 ( **Netflix Ribbon**)
* 断路器模式 ( **Netflix Hystrix**)   		
* 分布式消息 ( **Spring Cloud Bus**)

在平时开发中，我们并不会和这些基础设施打交道，相反，更多的是基于熟悉的Spring Framework(MVC) 开发。
对于Spring Cloud的诸多概念，我会在涉及到的部分简要介绍一下。







