<!-- toc -->
## SPI自动生成API文档
### 插件配置
API文档是根据Java源代码的注释自动生成，Maven项目（SPI模块）仅需配置apidoc插件：
```xml
		<plugin>
				<groupId>com.dooioo.se.lorik</groupId>
				<artifactId>maven-apidoc-plugin</artifactId>
				<version>1.0.2</version>
				<extensions>true</extensions>
				<configuration>
					<options>-appName "${project.name}"</options>
				</configuration>
		</plugin>
```

该插件需配置参数：
  -appName “微服务的中文名称”
  
然后运行Maven 命令： `mvn apidoc` 即可。

源代码的解析是由lorik-apidoclet-1.0.1.jar负责的，解析后的数据自动导入到：[http://api.doc.dooioo.org](http://api.doc.dooioo.org)。

### 注意事项
如果遇到Maven Apidoc插件无法下载时，请修改你本地的${user.home}/.m2/setting.xml，添加插件仓库：

``` xml
<pluginRepositories>
            <pluginRepository>
              <id>central</id>
              <name>Maven2</name>
                <!— 公司私服地址 —>
              <url>http://nexus.dooioo.org/nexus/content/groups/public</url>
              <layout>default</layout>
              <snapshots>
                <enabled>true</enabled>
              </snapshots>
              <releases>
                <updatePolicy>never</updatePolicy>
              </releases>
            </pluginRepository>
     </pluginRepositories>
```

### 注释要求
源代码的注释必须符合我们的要求：
``` java
/**
 * 客户端调用的房屋登盘申请SPI
 * @summary 房屋登盘申请
 * @Copyright (c) 2016, Lianjia Group All Rights Reserved.
 */
@FeignClient(name = "loupan-management-server")
public interface HouseRegisterApplySpi {
  /**
   * 经纪人申请房屋登盘，必须指定楼盘名称、栋坐名称、房屋名称。如果是自动提
   * 示所选择的，则可提供楼盘ID、栋坐ID
   * @author huisman
   * @version v1
   * @param resblockId 楼盘ID
   * @param unitId 单元ID
   * @param resblockName 楼盘名称
   * @param unitName 单元名称
   * @param houseName 房屋名称
   * @param houseAddress 房屋地址，如果有楼盘ID ，那么可能为空
   * @param applyComment 申请备注
   * @param attachPaths 附件路径，多个路径以逗号隔开
   * @param attachFileNames 附件名称，多个路径以逗号隔开，先后顺序和attachPaths相同
   * @param cuser 创建人，当前登录用户
   * @return 申请成功，返回申请结构，包含ID
   * @summary 经纪人申请房屋登盘
   */
  @LorikRest(codes = {20111})
  @RequestMapping(value = "/v1/houses/register/apply", method = RequestMethod.POST)
  HouseRegisterApply applyV1(@RequestParam(value = "resblockId", required = false) Long resblockId,
      @RequestParam(value = "unitId", required = false) Long unitId,
      @RequestParam(value = "resblockName") String resblockName,
      @RequestParam(value = "unitName") String unitName,
      @RequestParam(value = "houseName") String houseName,
      @RequestParam(value = "houseAddress", required = false) String houseAddress,
      @RequestParam(value = "applyComment", required = false) String applyComment,
      @RequestParam(value = "attachPaths", required = false) String attachPaths,
      @RequestParam(value = "attachFileNames", required = false) String attachFileNames,
      @RequestHeader(StandardHttpHeaders.X_Login_UserCode) int cuser);
}
```

### 文档截图
上文示例生成的文档如下图所示：
#### 目录导航
![文档截图-目录]({{book.imagePath}}/parts/chapter2/images/spi-summary-page-left.png)

#### 方法展示
![文档截图-方法展示]({{book.imagePath}}/parts/chapter2/images/spi-method-page-header.png)


#### 参数部分
![文档截图-方法参数]({{book.imagePath}}/parts/chapter2/images/spi-method-page-body.png)


## 普通SpringMVC项目（非Spring Cloud）
我们的文档自动生成也支持普通的SpringMVC项目，所有@Controller和@RestController标注的类的源代码都会被解析。

### 额外配置
由于普通的SpringMVC项目没有虚拟主机地址（一般为英文，可以使用’-’分隔），所以插件需额外配置一个参数：-app “you-app”，这个值一般是配置在API网关的静态路由的path。
``` xml

	<plugin>
		 <groupId>com.dooioo.se.lorik</groupId>
		 <artifactId>maven-apidoc-plugin</artifactId>
		 <version>1.0.0</version>
		 <extensions>true</extensions>
		 <configuration>
			<options>-app "you-app” -appName "${project.name}"</options>
		</configuration>
	</plugin>
```