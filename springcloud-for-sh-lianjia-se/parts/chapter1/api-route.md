<!-- toc -->

### 认证
所有接口文档（[http://api.doc.dooioo.org/doc](http://api.doc.dooioo.org/doc)）中出现***需要授权***的Label，接口请求时都必须传递Http Request Header: `X-Token`。

![Login Label]({{book.imagePath}}/parts/chapter1/images/login_label.png)


#### 申请X-Token
API网关负责统一认证，所谓认证，是指调用SSO登录服务验证`X-Token`，认证通过后，API网关自动添加Http Request Header： `X-Login-UserCode`和`X-Login-Company-Id`，将员工登录信息传递给后台服务。

`X-Token`可通过工号和密码调用SSO登录接口获得，接口响应的`loginTicket`即`X-Token`的值，具体接口文档请参考：
[SSO登录服务 - 员工登录 (员工登录)
](http://api.doc.dooioo.org/v1/doc/3606983534/3255124964/1626426016)

注意事项：
* 多次调用SSO登录接口时，如果`Source`相同，则旧的loginTicket会失效，`30分钟`内`旧loginTicket`会提示重复登录或者登录状态异常。
* `loginTicket`目前所有平台（Source)的有效期是两小时，携带`X-Token`访问任何接口都会`自动顺延`loginTicket的过期时间。
* 通常情况下，只需调用一次登录SSO接口获取loginTicket，之后可通过刷新Ticket的方式延长loginTicket的过期时间。

#### Web页面跨域访问授权接口
众所周知，授权接口都需要传递Http Request Header ：`X-Token`，为了支持Web页面跨域访问授权接口，
Ajax请求必须携带登录cookie，代码示例(集成环境)如下：
``` javascript
//需要授权的，JQuery1.5.1+，低版本有bug
$.ajax({
   type:'GET',
    xhrFields: {
      withCredentials: true
   },
   url:'http://api.route.dooioo.org/loupan/management/server/v1/organization/coordinates/paginate?orgId=&pageNo=1&pageSize=20',
   dataType:'json'
   })
   .done(function( data ) {
    if ( console && console.log ) {
      console.log( "Sample of data:", data );
    }
  });
```
Web页面请求授权接口的前提条件：
1. 员工登录了当前页面（生成了登录SSO cookie: `login_ticket` );
2. Ajax请求设置了请求属性： `withCredentials: true`;

#### 网关负责认证，业务负责授权
API网关只负责认证，授权则交给每个服务提供者。    
  
也就是说，后端服务拿到的当前登录人（`X-Login-UserCode`和`X-Login-Company-Id`）API网关可以保证是合法的、不会被客户端伪造，至于员工能否查看某个数据，则交给业务方来处理。  
  
如果业务方编码不当，则可能出现越权访问。

#### API网关常见业务码
通过API网关访问授权接口，`X-Token`非法时，会响应以下业务码：  
![api route bizcode]({{book.imagePath}}/parts/chapter1/images/apiroute_bizcode.png)

### 路由
API是访问机房服务的桥梁，而能否访问到某个服务，则取决于是否有此服务的路由信息。 

路由，可理解为虚拟路径和主机IP的映射规则：
```
   # 如果请求路径前缀是：/ky/old/server ，则路由请求到主机：http://10.8.1.204:1080
  /ky/old/server =>  http://10.8.1.204:1080
  
  # 如果请求路径前缀是：/loupan/server，则路由请求到主机：http://loupan.dooioo.com
  /loupan/server => http://loupan.dooioo.com
```


路由目前分为静态路由和动态路由。

#### 动态路由
所谓动态路由，就是服务会自动发现，并自动建立服务名到IP的映射规则，无需人为处理。  
  
目前，我们使用Eureka来实现服务的自动发现，大家可以访问：[http://discorvery1.se.dooioo.com](http://discovery1.se.dooioo.com)查看所有注册的服务。
  
动态路由将虚拟路径映射为虚拟主机名，请求路由时，根据**虚拟主机名**去Eureka里查询所有节点IP：端口，再负载均衡到某个节点上。

而虚拟主机名通常为 FeignClient的value，比如： `@FeignClient(“loupan-management-server”)`;

动态路由的映射规则示例：
```
   #如果请求路径前缀是：/loupan/management/server ，则路由请求到虚拟主机：loupan-management-server，
   #实际路由时，组件根据虚拟主机名去Eureka里查询所有节点的IP:端口，负载均衡到某一个IP上。
  /loupan/management/server =>  loupan-management-server
```
#### 静态路由
静态路由，是指需要管理员手动维护的映射规则。  
通常是为了方便测试或者将老系统接入API网关。

静态路由虚拟路径的命名规范： /产品线/产品/server，比如：/ky/tel/server，表示客源产品线的电话服务。

静态路由的映射规则（path=> location)示例：
```
   #如果请求路径前缀是：/ky/tel/server ，则路由请求到主机：http://tel.ky.dooioo.com:80
  /ky/tel/server =>  http://tel.ky.dooioo.com
```

##### 添加静态路由
测试和集成环境，开发和测试人员可自由添加静态路由，方便调试。  
接口说明文档请参考：[API网关 - 管理支持 (新增或更新静态路由)
](http://api.doc.dooioo.org/v1/doc/3100800167/194699906/2606921125)

此接口需要Header参数：`X-Role-Id`，测试和集成环境有效值为：admin。
`path`参数即虚拟路径，`location`为服务访问的域名。

生产环境的静态路由，请在上线之前钉钉发给我。

#### 查询可用路由
开发和测试人员可通过接口： [http://api.route.dooioo.com/admin/routes.json](http://api.route.dooioo.com/admin/routes.json) ，查询当前可用路由信息。

集成环境将.com调整为.org，测试环境将.com调整为.net。
### 跨域支持
API网关支持CORS规范，IE10、Chrome、FireFox、Safari浏览器可直接跨域访问接口，不需要通过Jsonp获取数据。

#### Origin的限制
CORS跨域请求都会设置Http请求头：`Origin`，有效值为发起跨域请求的主机域名。
目前我们支持的域名后缀：测试环境=dooioo.net，集成环境=dooioo.org，生产环境=dooioo.com。

### 超时、重试、断路器机制
#### 超时
API网关静态路由和动态路由的Http请求读取超时时间：`7000ms`，连接超时：`5000ms`。
#### 重试
动态路由的Http请求（访问微服务接口），如果连接超时，默认重试一次；  
遇到读取超时，如果请求方法为`GET/DELETE/HEAD`等幂等性方法，会重试一次，其他`POST/PUT`不会重试。

静态路由的Http请求不重试。

#### 断路器
动态路由的Http请求（访问微服务接口）才会应用断路器逻辑。

断路器开启条件： `10秒`内，请求次数超过`30`次，请求失败比例超过：`%65`。
  
粗略地来说，如果某个服务10秒内有超过30个请求，但是失败了：30*0.65=18个请求，则会开启断路。

断路器开启后，`5秒`（时间窗口）内不转发请求给后台服务。

### 测试
测试人员可以通过API文档中心（[http://api.doc.dooioo.org/doc](http://api.doc.dooioo.org/doc)： 服务- 导入到Postman - API网关简单测试，将服务接口导入到Postman，请不要手动输入接口。

下面我演示如何将楼盘字典核心接口导入Postman：  
#### 访问API文档中心
 访问API文档中心（[http://api.doc.dooioo.org/doc](http://api.doc.dooioo.org/doc)，找到楼盘字典核心服务，点击展开 ，选择导出到Postman。  
![apidoc loupan]({{book.imagePath}}/parts/chapter1/images/api_doc_loupan_core.png)

#### 复制API网关简单测试里的链接 
![apidoc import postman]({{book.imagePath}}/parts/chapter1/images/api_doc_import_postman.png)
#### 导入到Postman Chrome App
打开Postman Chrome App，找到左上角的导入，在弹出的对话框中选择”From Link”，将刚才的连接粘贴到输入框，点击”Import“，关闭对话框，可在左边的集合目录里找到刚才导入的微服务。  
![import postman]({{book.imagePath}}/parts/chapter1/images/postman_import.png)


### 实时流量监控

#### API网关实时流量监控
API网关实时流量在线监控：[http://turbine.se.dooioo.com/monitor](http://turbine.se.dooioo.com/monitor)。  
  
集成环境将com更换为org，目前测试环境暂不提供此功能。

监控指标说明：  

![Hystrix Dashboard intro]({{book.imagePath}}/parts/chapter1/images/turbine_dashboard_introduction.png)

#### 其他微服务实时流量监控
如果后端服务或者UI项目启用了断路器，也可通过turbine监控实时流量。  
  
访问地址： [http://turbine.se.dooioo.com/monitor?serviceId=你的spring.application.name](http://turbine.se.dooioo.com/monitor?serviceId=spring.applicaiton.name)  

serviceId默认为API网关。  

示例：   
小区UI开启了断路器，它的`spring.application.name`为“loupan-web”，
我们可通过以下连接查看实时流量: [http://turbine.se.dooioo.com/monitor?serviceId=loupan-web](http://turbine.se.dooioo.com/monitor?serviceId=loupan-web)

#### 实时流量监控的前提
1. 增加Hystrix依赖，开启断路器。 
```xml
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-hystrix</artifactId>
		</dependency>
```
2. 依赖spring-boot-actuator。Spring Boot默认有此依赖，可忽略。
3. feign.hystrix.enabled=true，默认true，除非你配置成false。
4. 没有禁用Management Endpoint，默认不禁用，可忽略。
  
### 调试指南
当某个API网关的请求异常时，如何定位问题？  
  
默认情况下，所有的微服务接口都会自动添加Response Header: `X-Instance-Id`。  
而经过API网关路由的请求会自动添加Response Header: `X-Route-By`。

这两个Header的值即节点IP的加密值，开发人员可通过接口解密：[API网关 - 管理支持 (查询X-Instance-Id对应的IP)
](http://api.doc.dooioo.org/v1/doc/3100800167/194699906/427734797)

通过`X-Instance-Id`、`X-Route-By`开发人员可以判断请求是否到达API网关、是否到达服务接口以及那个节点响应了客户端请求。


下图是使用Postman向我们的API网关`api.route.dooioo.com` 发起的一个请求 

```http
  $.get("http://api.route.dooioo.com/loupan/server/v1/citys")  
```

API网关响应如下：  

![API网关请求头]({{book.imagePath}}/parts/chapter1/images/api-route-header.png)  
  

我们通过接口：[http://api.route.dooioo.com/instance/b7b0fc1f593866af3ac8e2526dd0a880](http://api.route.dooioo.com/instance/b7b0fc1f593866af3ac8e2526dd0a880)，即可查询是那个楼盘服务的节点响应了客户端。

### 注意事项
#### 静态路由可能会添加多次’X-Forwarded-For’ Header。
主要原因是，静态路由通过域名访问后端服务，会多次经过Nginx，这样获取客户端IP时，如果通过 ‘X-Forwarded-For’获取客户端IP时会取到多个IP值。
#### 参数编码
所有接口URI的编码字符集为：`UTF-8`，请求参数的编码字符集为：`UTF-8`。
#### 跨域限制
Web页面跨域访问API网关时，Http请求Header ’Origin‘的值必须是以`dooioo.com`(正式环境)为后缀的域名，否则API网关直接响应403。

#### 并发数
动态路由的每个服务对应一个HttpClient，断路器的并发数为：100，HttpClient单节点最大并发数：150，HttpClient配置如下：
```
spi.httpclient:
    keepAliveInSeconds: 40
    maxRoutePerHost: 150
    maxTotal: 600
```

所有静态路由共用一个HttpClient，单个域名最大并发数：300， HttpClient配置如下：
```
##连接池总连接数1200  
zuul.host.maxTotalConnections: 1200
#每个域名并发连接300
zuul.host.maxPerRouteConnections: 300 
```



