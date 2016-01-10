
对编写SPI的工程师来说，接口的安全性也是必须考量的一点。
如何确保关键业务数据不外泄？为保证接口的安全调用，我应该做什么？
本节主要介绍我们如何保障接口访问的安全性。 
 
##### 生产环境和测试、集成环境隔离
![生产环境和其他环境隔离]({{book.imagePath}}/parts/chapter2/images/enviroment-isolation.png)

也就是说：
* 测试或者集成环境的Eureka Client不会注册到正式环境。
* 测试或集成环境的Eureka Client不会调用正式环境的服务。

##### 生产环境仅限服务器内部访问


 如何确保接口的安全性？我们目前方案如下：

*  隔离，生产环境只能服务器内部访问。  
*   SPI接口所有方法默认需要登录，除非使用@LoginNeedless显式指明无需登录校验。
*  lorik-spi-security-plugin将对客户端进行更严格的身份认证，服务提供方只需引入依赖即可。  
*  客户端通过FeignClient访问远程服务是信任的。
*  通过API网关(api.route.dooioo.org)代理的Http请求是信任的。
*  其他访问请求直接被拒绝。








