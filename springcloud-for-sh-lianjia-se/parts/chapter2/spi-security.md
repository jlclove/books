<!-- toc -->
对编写SPI的工程师来说，接口的安全性也是必须考量的一点。
如何确保关键业务数据不外泄？为保证接口的安全调用，我应该做什么？
本节主要介绍我们如何保障接口访问的安全性。 
 
#### 生产环境和测试、集成环境隔离
![生产环境和其他环境隔离]({{book.imagePath}}/parts/chapter2/images/enviroment-isolation.png)

也就是说：
* 测试或者集成环境的Eureka Client不会注册到正式环境。
* 测试或集成环境的Eureka Client不会调用正式环境的服务。

#### 微服务仅限生产服务器内部访问
![微服务仅限内部访问]({{book.imagePath}}/parts/chapter2/images/internal-call.png)

微服务仅限生产服务器内部调用，也就意味着：
*  所有非正式环境网段的IP地址发出的TCP请求直接被丢弃，微服务仅限内部调用。
*  仅允许Nginx转发过来的请求（本质上讲，Nginx也是正式环境网段内的）
*  如果终端用户知道了某个生产服务器的IP地址和端口，绕过了Nginx节点，直连该服务器，那么TCP请求直接被防火墙干掉。
*  REST的方式访问微服务，也会首先由Nginx转发请求给API网关，任何试图绕开Nginx直接访问微服务的TCP请求直接被防火墙干掉。

#### 关键业务接口的方法禁止添加@LoginNeedless
如果接口会返回关键业务数据、用户隐私数据或者利益相关的数据，请校验用户是否登录。 默认情况下，所有SPI方法都是登录用户才能请求的。校验用户是否登录，lorik-spi-view已提供透明的支持，SPI编写者无需关心。

但是，**如果SPI编写者主动给接口方法添加```@LoginNeedless```标注，那么lorik-spi-view将不会校验请求是否是登陆用户发出的**。

一般情况下，有些接口没有任何副作用，可以公开访问的话，接口方法添加@LoginNeedless 有助于客户端直接访问。

**所以，各位SPI编写者要清楚那些接口需要校验用户登陆（禁止添加@LoginNeedless）；那些接口可以公开访问，可直接在浏览器里跨域访问（方法需要添加@LoginNeedless）**。

#### 引入lorik-spi-security-plugin
通过以上措施，我们的SPI基本安全了。

但是，为了防止某个机智的工程师登陆生产服务器，通过curl或用其他工具访问关键业务接口，简单来说，为了防止生产服务器内部伪造的请求，我们提供了安全插件：**lorik-spi-security-plugin**，服务提供方（Server）引入该插件即可获得更严格的身份认证。

安全插件的工作原理很简单：
1. 首先所有微服务的调用方（包括API网关）在启动时会随机生成一个安全Token，在向Eureka Server注册时，保存在实例的Metadata里。  
 安全Token的生成策略可调整为动态的，比如每随机30分到3小时更新一下Token。  

2. 所有对微服务的调用都会添加三个自定义头：X-Client-Name、X-Client-Id、X-Client-Security-Token。X-Client-Name为调用方的spring.application.name，亦即向Eureka注册的VIPAddress，X-Client-Id是调用方注册到Eureka时生成的实例ID，X-Client-Security-Token是第一步生成的Token。
3. SPI提供方引入安全插件后，我们会自动拦截以上三个Request Header，并根据X-Client-Name从Eureka Server获取实例的注册信息，然后校验客户端IP、安全Token、实例ID是否合法。

这样，即使是生产环境服务器内部发起的伪造请求，由于无法提供正确的请求头，Http请求将直接被拒绝。

当然，前提是我们的Eureka服务器要做好安全控制，不能泄露客户端生成的安全Token。


#### 开放给外网的API，采用HTTPS+OAuth2
针对开放给外网访问的REST接口，我们采用Https传输接口数据，通过OAuth2认证客户端的身份。








