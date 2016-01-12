###pom.xml
SPI模块只需引入三个jar：

*  `spring-webmvc`   
	仅为了代码编译通过，scope为provided。
*  `spring-cloud-netflix-core`   
	仅为了代码编译通过，scope为provided
*  `lorik-spi-view`   
 我们内部对spi层的支持，包括登录校验、分页、Model的视图支持、支持Spring MVC 标注（@RequestParam，@Pathvariable,@RequestHeader)的继承等。  
必须显式依赖特定版本号，**目前版本号为{{ book.lorikSpiViewVersion }}**。

pom.xml如下：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.lianjia.sh.samples.loupan</groupId>
    <artifactId>loupan</artifactId>
    <version>0.0.1-SNAPSHOT</version>
  </parent>
  
  <artifactId>loupan-spi</artifactId>
  <packaging>jar</packaging>
  <version>1.0.0</version>
  
  <dependencies>
      <dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<scope>provided</scope>
		</dependency>
		<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-netflix-core</artifactId>
				<scope>provided</scope>
		</dependency>
		<dependency>
         	<groupId>com.dooioo.se.lorik</groupId>
		    <artifactId>lorik-spi-view</artifactId>
		    <version>version</version>
        </dependency>
   </dependencies>
</project>
```