### 微服务
关于微服务的讨论，可以参考： [Martin Fowler - MircroServices](http://martinfowler.com/articles/microservices.html "martinfowler microservices")。

站在工程师的角度看，微服务涉及三种角色：
//todo 流程图  

* Service SPI [^1]   服务契约
* Sevice Provider 服务实现方
* Client Consumer 服务调用方

对于工程师来说，引入Service 和调用方式没有什么变化，Spring项目代码如下：

```java
  @Service
  public class MyService{
  	// CityService 为SPI
    @Autowired
    CityService cityService;
    
    public List<MyObject> findById(int id){
       City city= cityService.findById(id);
       if(city == null){
        return Collections.emptyList();
       }
       // other logic
     }
  }
```
但是在底层，服务间之间调用已由 **同一进程内调用** 透明的转换为 **跨JVM进程间调用**。

之前，当我们某个项目线提供一组公共服务时，通常将服务封装成Jar包发布，但随之而来的，如果Jar包出现Bug、配置文件变更、功能调整、或者安全性问题时，所有依赖方（客户端）必须强制升级，真可谓，牵一发而动全身，我们称这种服务依赖方式为：“**硬引用**“。

微服务是如何避免这种尴尬的服务维护呢？

对客户端来说，如果我要萝卜，服务方给它萝卜即可，至于萝卜是从何而来，客户端是不关心的。反观我们采用“**硬引用**”的方式暴露服务时，将实现方式直接驻留在了客户端，从而变相地和客户端耦合在了一起。

基于微服务的方式开发时，项目线（Service Provider）只向客户端(Consumer)提供 **Service SPI**。SPI只是一系列声明（各种接口和Model），没有任何实现细节，驻留在客户端的只是一个空壳，真身仍在项目线。

如果某服务声明说我有（返回）萝卜，那客户端就能获取萝卜，绝无可能冒出白菜（所谓的契约）。至于萝卜的制造工艺，各项目线可随时变更，只要不违背契约即可（通常服务都有版本号）。

我们称这种服务依赖方式为: ”**软引用**“，类似文件的快捷方式。

微服务还有其他诸多特点或挑战，但在此书中不会过多讨论。

### 微服务、Spring Cloud、Netflix OSS[^2]

上文说到，微服务的方法调用是一种跨JVM进程的调用。也就意味着，微服务离不开分布式系统之间的交互。

在一个分布式系统的环境中，我们通常会部署多个节点（实例），而且很有可能动态增加一些节点来应对紧急网络流量。传统的基于Nginx的静态路由（由人工手动配置），显然是不合适的。

一次典型的针对微服务的方法调用，其背后的基本步骤如下：  

	 1. 服务发现 - 根据所调用的服务查找Service Provider的所有可用节点；
	 2. 服务路由 - 从众多节点根据特定的规则选择一个节点。
	 3. RPC请求 - 向选定的节点发起远程网络调用(RPC)，解析返回数据。
 
由于网络调用的复杂性，我们还必须处理RPC[^3]请求异常时可能导致的雪崩效应[^4]，所以流量整形[^5]和监控是必要的。

另外，由于部署多个节点，如何管理应用程序的环境配置也是必须考虑的。

针对以上这些基本需求，Spring团队整合了一种解决方案：Spring Cloud + Netflix OSS。

####服务发现 - Eureka
[Netflix Eureka 1.X.X](https://github.com/Netflix/eureka/wiki "Eureka Wiki") 使用了一种简单机制实现了节点的自动注册和发现。

Eureka 有两种角色：Eureka Client 和 Eureka Server。通常我们的Spring Cloud 程序（服务）都是作为Eureka Client启动的。 

Eureka Client启动时，会根据配置的 Eureka Server的Url，主动向Eureka Server汇报自己的以下信息：

*  **虚拟主机地址（VIPAddress）**   
	服务的标识，极其重要，只有根据VIPAddress才能在Eureka Server中查找服务的节点。
*  **主机名或IP地址**   
默认是主机名，但通常都配置成节点的IP地址。
*  **端口**   
节点对外提供服务的端口
*  **节点实例ID**    
全局唯一，如果重复，会覆盖之前注册的相同实例ID的节点。
*  status url   
程序运行状态的监控url.
*  health url   
程序健康状况的监控url
*  metadata 和其他   
其他的元数据信息，可被其他服务获取。

与此同时，Eureka Client会开启一些定时任务，按固定频率向Eureka Server轮询其他客户端的注册信息（注册信息会缓存在本地一段时间），当然也少不了发送心跳包。

Eureka Server(1.X.X)目前不支持向所有节点广播节点变更，节点的自动发现主要依赖Eureka Client定时Poll数据。

默认情况下，Eureka Client的实例ID为主机名，这样同一个台机器只能运行一个Client。但通过Spring，我们可以让Eureka Client启动时生成随机的Instance Id。

SE团队集目前部署了两个Eureka Server节点（集成环境）：
[http://discovery1.se.dooioo.org](http://discovery1.se.dooioo.org) 和[http://discovery2.se.dooioo.org](http://discovery2.se.dooioo.org)，正式环境将顶级域名.org 改为.com，访问即可查看所有已注册的Eureka Client。

#### 服务路由 - Ribbon
当Eureka Client通过特定服务的**VIPAddress**获取到众多节点时，如何路由到一个合适的节点？这时轮到Netflix开源组件 [Ribbon](https://github.com/Netflix/ribbon/wiki "Netflix Ribbon Wiki") 出场了。

Ribbon 有以下特性：

* 插件式的路由规则，内置Round Robin 和 Response time weighted 。既可以向节点随机分发请求，也可以根据响应时间来分发。另外，我们可以方便地扩展一些符合自己应用场景的路由规则。
* 集成Eureka服务发现（Ribbon-Eureka模块）。
* 弹性容错。Ribbon通过`IPing`接口可以动态感知节点是否存活，以过滤一些失去响应的节点。它可以进一步基于断路器[^6]模式过滤节点。关于断路器模式，请参考 [Circuit Breaker](http://martinfowler.com/bliki/CircuitBreaker.html)。
* 支持分布式云环境。假如，我们将服务部署到阿里云不同的数据中心，比如，北京，杭州，上海，节点路由时，可以优先选择位于同一数据中心的节点，也可以主动避开网络拥堵的数据中心。

默认情况下，Ribbon 随机转发请求到各个节点。

#### RPC调用 - Feign
当Ribbon选定节点后，接下来客户端便要发起RPC调用了。

常见的RPC调用，有二进制流序列化+TCP协议 或 二进制流序列化+Http协议(比如Hessian)，序列化/反序列化协议既可以是原生JAVA，也可以是其他序列化协议（比如，[Kryo](https://github.com/EsotericSoftware/kryo/wiki) 和 [Fst](https://github.com/RuedigerMoeller/fast-serialization/wiki) ,Thrift,ProtoBuf)；有文本序列化+ Http协议，使用JSON或XML反序列化。

最终，我们的RPC调用决定采用Http协议+JSON文本序列化的方式，这样即可以直接将服务作为Rest接口暴露出去，也更容易对众多服务进行自动化测试。

而完成这一RPC调用的组件，就是 [Netflix Feign](https://github.com/Netflix/feign)。

Feign 根据 Service SPI 接口的标注信息，构造符合Http协议的请求参数。你可以把Feign看成一个HttpClient。比如，我们以后的Service SPI类似以下代码：

```java
@FeignClient("city")
//@FeignClient(url="http://localhost:8080")
public interface CityService{
    @RequestMapping(value="/v1/city/{id}",method=RequestMethod.GET)
    City findById(@PathVariable(value="id")int id );
 }
```

Feign 有一个接口 `feign.Contract`，用于完成Http请求协议的构造。Spring提供了 `SpringMvcContract`以支持Spring MVC标注的解析。

上文中CityService的`findById`将会被Feign理解为：向 url = http://city/v1/city/{id}的主机发起一个Http **GET**请求，id为模板参数，运行时替换。

大家也看到了，url的主机为”city“，也就是`@FeignClient("city")`中的”city“,”city“被称为**虚拟主机地址**，是服务的标识。一个服务通常是一个Eureka Client，服务会部署多个节点，那么根据虚拟主机地址，我们就能获取服务的对应节点了。

此时，远程调用时需先向Eureka Server查询该虚拟主机对应的所有节点，再由Ribbon选定一个合适的节点。

但我们也可以直接指定请求的url：`@FeignClient(url="http://localhost:8080")`，此时会被Feign理解为：向 url = http://localhost:8080/v1/city/{id}的主机发起一个Http **GET**请求。此时已无需Ribbon路由，通常在测试时才会手动指定节点。

Url主机地址被动态解析并替换，同时根据方法实参将Url或请求数据中的模板参数替换为实际数据，开始发送请求，最后对响应信息反序列化。


#### 流量整形以及断路器

在我们开始编码之前，强调几点：

* 面向契约  
 自古以来，Java中就流传着一种传说：面向接口、面向契约编程。但在快速迭代中，接口反而成了累赘。这次微服务实践，服务将全部面向契约，但服务实现方不做约束。
 
* 无状态性  
未付
* 代码的自我表达能力
* 必要的注释以及版本意识
 
  
 
[^1]: SPI = Service Provider Interface
[^2]: OSS = Open Source Software
[^3]: RPC = Remote Procedure Call
[^4]: “雪崩效应”是指信息在沿供应链传递中其波动会被依次放大的现象，这种现象导致信息在传递过程之中有如滚雪球一般越滚越大。
[^5]: 流量整形(traffic shaping)典型作用是限制流出某一网络的某一连接的流量与突发，使这类报文以比较均匀的速度向外发送。
[^6]: 断路器（Circuit Breaker）是指能够关合、承载和开断正常回路条件下的电流并能关合、在规定的时间内承载和开断异常回路条件下的电流的开关装置。
