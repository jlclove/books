### 编码之前
在编码之前，请同学们务必阅读之前的章节。
另外，有几个编码要点，在此强调一下：

* 面向契约  
自古以来，Java中就流传着一种传说：面向接口、面向契约编程。但在快速迭代中，接口反而成了累赘。这次微服务实践，Service SPI将全部面向契约，但服务实现方不做约束。
* 应用程序无状态  
编写分布式程序的最佳实践就是节点无状态，我们可以简单的将无状态性理解为Http请求的幂等性：一次和多次请求任何一个节点都有相同的副作用。

简单来说，应用程序节点之间无依赖，我们不需要同一个用户的请求只能落在一个节点上（IP Hash)。

那么，遇到共享资源（数据）时，我们如何解决？

我之前做的一个项目，电话转接号的申请接口就是一个共享资源的例子：

*  电话转接有两到三个节点，尚未分配的所有转接号就是一个共享资源池（同一个数据库）。
*  一个转接号只能分配给一个人。
*  请求可能同时重复发送（可能落在同一节点，也可能落在任一节点），无论请求发送多少次，此人已分配的转接号应该不变（不能重复申请,但每隔三天此人的转接号就要过期）。

当时设计的目标TPS=300/s,平均响应时间应在300ms以下,具体实现分以下几步：

*  节点一启动，首先主动向数据库申请4000个转接号（更新转接号状态为已占用）放在程序内存中，比如 currentPool=`ConcurrentLinkedQueue`，有一个`AtomicInteger`类型的计数器，统计集合中已分配了多少转接号，如果达到某个阈值，再异步向数据库批量申请4000个放在另一个 preparedPool=` ConcurrentLinkedQueue`中，当currentPool分配完或不足时，此时再加锁，执行逻辑：`currentPool=preparedPool,preparedPool=null;`。  
   这样便将数据库事务转换成JAVA并发了，高并发时也能稳定服务。  

* 节点收到员工申请转接号的请求时，先向memcache中添加一个标识位（`memcache#add(key,value,10s)`),类似互斥锁。操作成功之后，开始向currentPool申请转接号，然后缓存到memcache中。  

* 如果某个节点向memcache添加互斥锁时失败，那说明有节点已在处理了，那它应该等待200毫秒，直接从缓存中获取已分配的转接号。一般可以重试两三次，我们允许此种情况下可能出现的失败。  
  基本这种情况是极其少见的，一般是压力测试或被攻击时才频繁。
  
 可以看到，我使用了memcache来承担全局变量的角色，也存在某种程度的风险。
 在平时的业务实现中，应该尽量在设计上避免此类共享资源的场景。
 
除了上述的共享资源，还有一种场景我们也应该避免：分布式事务。处理跨系统之间业务逻辑，尽量走异步队列，如果某次业务逻辑处理失败，尽量有重试机制，以保证业务数据最终一致性。
 
* 代码的自我表达能力  
代码的自我表达能力是一种最佳编码实践。有时候，仅仅是实现方式的稍微调整，比如以下代码是向远程服务器发起Http调用：
 
```java
	RemoteRequest  
	.to(“http://api.route.dooioo.org/loupan/server/v1/city/{id}”)
    .withParam("id",3).get(City.class);
```
比如，有时候使用静态内部类（接口），也可使代码更可读：

```java
  public interace InvocationHandler{
       Object invoke(some param);
       
       static final class Factory{
         InvocationHandler   create(some param){
         }
       }
  }
  //调用时
   new InvocationHandler.Factory();
  
```
代码编写和写作是类似的，我们既要追求代码简洁，又要追求表达力。

在这方面，我们还有许多改进和学习的地方，欢迎讨论。

* 完善的注释以及版本意识  
本次微服务实践，Service SPI的编写者必须提供完善的注释。
除了方便客户端更友好的接入外，我们会自动根据java doc 生成 API 网关的文档。
java doc的说明文档请[参考这里](http://www.oracle.com/technetwork/java/javase/documentation/index-137868.html#tag)，大家可以了解下tags的顺序及其他说明。

下面我演示一下标准SPI接口注释的写法：
<pre><code class=“java”>
@FeignClient("loupan-server")
public interface DistrictSpi{
	/**
	 * 根据区域ID查找区域，方法的详细说明
	 * 说明，说明。
	 * @author huisman
	 * @param  districtId 区域ID
	 * @param  userCode 工号
	 * @since v1
	 * @summary 根据ID查找区域 
	 * @example /v1/district/500002032
 	 * @errorCode 
 	 * &nbsp;&nbsp;&nbsp;&nbsp;20010=区域已删除
	 * &nbsp;&nbsp;&nbsp;&nbsp;20020=区域巴拉巴拉
	 */
	@RequestMapping(value = "/v1/district/{districtId}", method = RequestMethod.GET)
	District  findDistrictV1(@PathVariable(value = "districtId") long districtId,@RequestParam(value=“userCode”,required=false,defaultValue=“8080”)Integer userCode);
}
</code></pre>

上述java doc在API Gateway 将被展示为：

|  字段  | 说明|
| :------------ | :-----------| 
| Request Path  | GET /v1/district/{districtId}?userCode={userCode}  |
| 接口功能  | 根据ID查找区域          |
| 接口说明  | 根据区域ID查找区域，方法的详细说明<br>说明，说明。      |
| 版本号  | v1          |
| 路径参数 | @PathVariable districtId，区域ID, 类型:Long，必填       |
| 请求参数 | @RequestParam userCode，工号，类型:Integer，可选，默认值8080      |
| 错误码|20010=区域已删除<br>20020=区域巴拉巴拉   |
| 返回值| http://api.route.dooioo.org/loupan/server/v1/district/500002032 |
| author|huisman|

我们新增加了三个java doc tag: @summary ,@errorCode,@example。

*  @summary 将会被展示为接口方法的功能说明（一般不超过20字）
*  @errorCode 是接口方法可能抛出的业务错误码，如果没有，则可以忽略，此时API文档会显示无。
* @example ，示例请求路径，用于展示接口方法返回值的json结构。  
通常API文档会在页面发送Ajax请求该路径,默认不需要指定主机地址，API文档会自动添加上虚拟主机地址，比如集成环境为：http://api.route.dooioo.org/虚拟主机地址/@example。  
上述示例接口的返回值则会请求：http://api.route.dooioo.org/loupan-server/v1/district/500002032，并将响应结果展示在页面上。  
 也可以写死固定的Url，则会直接ajax请求该url。

其他tag说明如下：

@author tag既可以是负责人名，也可以是邮件地址或者产品线团队。<br>
@since 我们用于显示API版本。<br>
@param 我们用于显示方法参数的说明。<br>
Spring MVC的标注将会被正确解释为符合Http 语义的说明，比如路径参数，请求参数。<br>
方法doc的描述（First Sentence）则显示为接口说明。

各个doc tags 的顺序建议和示例保持一致。


 
 
       