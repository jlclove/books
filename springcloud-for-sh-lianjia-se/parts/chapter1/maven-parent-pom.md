开发Maven多模块项目时，可以把各模块的公共配置放在Parent项目的`pom.xml`里。

`Spring Cloud`为了简化项目版本依赖的管理，提供了一系列构建类型为pom的starter项目。我们的项目只需要指定其Parent为`spring-cloud-starter-parent`，就可以继承spring-cloud-starter-parent的依赖管理、常用Maven插件等，这样子模块就可以导入依赖包，而无需关心其版本号，也不必重复配置Maven插件。

**我们采用的Spring Cloud 版本为：Brixton.M4**

``` xml
     <parent>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-parent</artifactId>
		<version>Brixton.M4</version>
	</parent>
```

最终，Maven Parent项目的pom.xml如下：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  
  <!--我们项目的父级为  Spring Cloud ， 版本为：Brixton.M4  -->
  <parent>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-parent</artifactId>
		<version>Brixton.M4</version>
	</parent>
	
  <groupId>com.lianjia.sh.samples.loupan</groupId>
  <artifactId>loupan</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>pom</packaging>
  <name>loupan</name>
  <description>Spring Cloud Sample for SE@sh.lianjia.com</description>
  
  <properties>
		<!--覆盖Spring提供的1.6，指定了项目编译后的class版本 -->
		<java.version>1.8</java.version>
 </properties>
 
   <!-- 为防止某些jar包无法下载，我们指定仓库地址 -->
  <repositories>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>http://repo.spring.io/milestone</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
	</repositories>
  
  <modules>
  	<module>loupan-server</module>
  	<module>loupan-spi</module>
  	<module>loupan-ui</module>
  </modules>
</project>

```


### 后续章节
parent pom.xml配置好之后，终于到实践环节了。

后续章节，先介绍如何编写SPI模块（第二章），接着介绍如何实现SPI（第三章），最后介绍如何调用SPI以开发一个web app（第四章）。