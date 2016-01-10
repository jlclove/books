### 从0开始编写一个SPI

本节我来演示如何声明SPI，演示用的SPI接口功能为：
* 判断城市是否存在
* 查询所有城市
* 根据gbCode查找特定城市
* 新增一个城市

Service SPI ，我们可以将之看做Class类型为接口的Service。如果不是编写微服务的SPI，我们会实现如下：

```java 
  public interface ICityService{
     /**
     * 根据gbCode（国标码）检测城市是否存在
     * @param gbCode 国标码
     * @return 如果城市存在，返回true，否则 false
     */
     boolean exists(int gbCode);
     
     /**
     * 查找当前状态正常的所有城市，不包括已删除和暂停状态的
     * @return never null，如果无数据，则返回空List
     */
     List<City> findAll();
     /**
     * 根据gbCode（国标码）获取城市。
     * 中国大陆的每个城市都有一个国标码（GB/T 2260-2002），比如北京:
     * 110000，上海：310000，推荐外键使用国标码来标识一个城市，而不要使用
     * City#getId()。
     * @param gbCode 国标码
     * @return 如果城市不存在，返回null，否则返回对应的城市
     */
     City findByGbCode(int gbCode);
     
    /**
     * 新增城市
     * @param cityName 城市名
     * @param gbCode 国标码 
     * @param latitude 维度
     * @param longitude 经度
     * @param cuser 创建人工号
     * @return 返回新增成功的城市，包含ID。
     */
     City add(String cityName,int gbCode,double latitude,double longitude,int cuser);
  }
```

当基于微服务开发时，因为这是我们的第一个版本，按照我们的规范，类名应该改为：CitySpiV1，方法名添加后缀V1，方法注释添加 @author、@version、@since、@summary（请留意doc tag的顺序）:

```java
public interface CitySpiV1{
     /**
     * 根据gbCode（国标码）检测城市是否存在
     * @author huisman
     * @version v1
     * @param gbCode 国标码
     * @return 如果城市存在，返回true，否则 false
     * @since 2016-01-01
     * @summary 根据国标码检测城市是否存在
     */
     boolean existsV1(int gbCode);
     
     /**
     * 查找当前状态正常的所有城市，不包括已删除和暂停状态的
     * @author huisman
     * @version v1
     * @return never null，如果无数据，则返回空List
     * @since 2016-01-01
     * @summary 查找所有城市
     */
     List<City> findAllV1();
     /**
     * 根据gbCode（国标码）获取城市。
     * 中国大陆的每个城市都有一个国标码（GB/T 2260-2002），比如北京:
     * 110000，上海：310000，推荐外键使用国标码来标识一个城市，而不要使用
     * City#getId()。
     * @author huisman
     * @version v1
     * @param gbCode 国标码
     * @return 如果城市不存在，返回null，否则返回对应的城市
     * @since 2016-01-01
     * @summary 根据国标码获取城市
     */
     City findByGbCodeV1(int gbCode);
     
    /**
     * 新增城市
     * @author huisman
     * @version v1
     * @param cityName 城市名
     * @param gbCode 国标码 
     * @param latitude 维度
     * @param longitude 经度
     * @param cuser 创建人工号
     * @return 返回新增成功的城市，包含ID。
     * @since 2016-01-01
     * @summary 新增城市
     */
     City addV1(String cityName,int gbCode,double latitude,double longitude,int cuser);
  }

```
如果我们的RPC调用是TCP+二进制流或者HTTP+二进制流，那么SPI声明就完成了。

但我们的SPI调用方使用Netflix Feign、以HTTP+JSON的方式发起RPC调用，所以必须告诉Feign如何发起HTTP请求，包括HTTP方法、HTTP请求路径、请求参数等。

简单点说，SPI调用方就是一个Http Client，它调用SPI接口，背后其实是对提供方（Server）发起HTTP请求以获取数据。 所以 CitySpiV1的每个方法必须提供必要的元数据信息以便Feign客户端（SPI调用方）能够构造正确的Http请求。

Feign定义了接口：```Contract```，用于解析SPI接口方法的元数据（方法或类上的Annotation），从而构造合适的HTTP请求。  

Spring Cloud的```SpringMvcContract```实现了```Contract```，支持使用Spring MVC标注 @RequestMapping、@PathVariable、@RequestParam、@RequestHeader 给方法添加元数据。

但是请注意，Spring Cloud目前仅支持 @RequestMapping、@PathVariable、@RequestParam、@RequestHeader，而且用法上有一点限制，具体来说：
* @RequestMapping必须指定method，并且只能指定一个Http Method。  
```java
 // 报错
  @RequestMapping(value="/v1/citys/{gbCode}")
  public City findByGbCodeV1(@PathVariable("gbCode") int gbCode);
  
  // 报错
  @RequestMapping(value="/v1/citys/{gbCode}",
  method={RequestMethod.GET,RequestMethod.POST}
  public City findByGbCodeV1(@PathVariable("gbCode") int gbCode);
  
  // 正确
  @RequestMapping(value="/v1/citys/{gbCode}",method=RequestMethod.GET)
  public City findByGbCodeV1(@PathVariable("gbCode") int gbCode);
```
  
* @PathVariable、@RequestParam、@RequestHeader，必须指定value值    

```java
 // 报错
  @RequestMapping(value="/v1/citys/{gbCode}"
  public City findByGbCodeV1(@PathVariable int gbCode);
  
  // 正确
  @RequestMapping(value="/v1/citys/{gbCode}"
  public City findByGbCodeV1(@PathVariable(value="gbCode") int gbCode);
```


为了配合Neflix Feign以HTTP+JSON这种方式进行RPC调用，我们必须给方法添加合适的Spring MVC标注。Request Mapping的url最好符合REST规范，大家可以参考第一章：
[扩展：REST API最佳实践](../restful-api-v1.4.html)。

更新后的代码如下：

```java
public interface CitySpiV1{
     /**
     * 根据gbCode（国标码）检测城市是否存在
     * @author huisman
     * @version v1
     * @param gbCode 国标码
     * @return 如果城市存在，返回true，否则 false
     * @since 2016-01-01
     * @summary 根据国标码检测城市是否存在
     */
     @RequestMapping(value="/v1/citys/{gbCode}/exists",method=RequestMethod.GET)
     boolean existsV1(@PathVariable(value="gbCode")int gbCode);
     
     /**
     * 查找当前状态正常的所有城市，不包括已删除和暂停状态的
     * @author huisman
     * @version v1
     * @return never null，如果无数据，则返回空List
     * @since 2016-01-01
     * @summary 查找所有城市
     */
     @RequestMapping(value="/v1/citys",method=RequestMethod.GET)
     List<City> findAllV1();
     /**
     * 根据gbCode（国标码）获取城市。
     * 中国大陆的每个城市都有一个国标码（GB/T 2260-2002），比如北京:
     * 110000，上海：310000，推荐外键使用国标码来标识一个城市，而不要使用
     * City#getId()。
     * @author huisman
     * @version v1
     * @param gbCode 国标码
     * @return 如果城市不存在，返回null，否则返回对应的城市
     * @since 2016-01-01
     * @summary 根据国标码获取城市
     */
     @RequestMapping(value="/v1/citys/{gbCode}",method=RequestMethod.GET)
     City findByGbCodeV1(@PathVariable(value="gbCode")int gbCode);
     
    /**
     * 新增城市
     * @author huisman
     * @version v1
     * @param cityName 城市名
     * @param gbCode 国标码 
     * @param latitude 维度
     * @param longitude 经度
     * @param cuser 创建人工号
     * @return 返回新增成功的城市，包含ID。
     * @since 2016-01-01
     * @summary 新增城市
     */
     @RequestMapping(value="/v1/citys/add",method=RequestMethod.POST)
     City addV1(@RequestParam(value="cityName")String cityName,
     @RequestParam(value="gbCode")int gbCode,
@RequestParam(value="latitude",defaultValue="0",required=false)double latitude,  
@RequestParam(value="longitude",defaultValue="0",required=false)double longitude,
     @RequestParam(value="cuser")int cuser);
  }

```

最后，我们检查各个接口，看看那些方法需要登录校验，那些方法可以公开访问：
* existsV1，findAllV1，findByGbCodeV1可以公开访问（REST 或者FeignClient），因为不是敏感数据，所以无需登录校验，我们手动添加@LoginNeedless；由于findAllV1，findByGbCodeV1支持REST访问，而响应可能为null，我们添加标注@LorikRest，启用特性Feature.NullTo404（REST访问时返回null则响应404）。  
另外，existsV1方法实现时有业务码抛出，所以用LorikRest的属性codes枚举出来以便生成API文档。  
如果当前接口不支持REST方式访问，可不加此标注。  

* addV1肯定需要登录校验的，一般记录创建人就是当前系统登录的用户。  
所以我们要把 @RequestParam(value="cuser")换成 @RequestHeader(StandardHttpHeaders.X_Login_UserCode)int cuser)。  
因为我们将登录用户的信息自动添加在Request Header里，目前只添加了StandardHttpHeaders.X_Login_CompanyId（登录员工的公司ID）和StandardHttpHeaders.X_Login_UserCode（登录员工工号），服务提供方如果要用，SPI必须使用@RequestHeader声明。

最终的CitySpiV1代码如下：

```java
public interface CitySpiV1{
     /**
     * 根据gbCode（国标码）检测城市是否存在
     * @author huisman
     * @version v1
     * @param gbCode 国标码
     * @return 如果城市存在，返回true，否则 false
     * @since 2016-01-01
     * @summary 根据国标码检测城市是否存在
     */
     @LoginNeedless
     @LorikRest(codes={"20100:城市已停用","20101:城市已删除"})
     @RequestMapping(value="/v1/citys/{gbCode}/exists",method=RequestMethod.GET)
     boolean existsV1(@PathVariable(value="gbCode")int gbCode);
     
     /**
     * 查找当前状态正常的所有城市，不包括已删除和暂停状态的
     * @author huisman
     * @version v1
     * @return never null，如果无数据，则返回空List
     * @since 2016-01-01
     * @summary 查找所有城市
     */
     @LoginNeedless
     @LorikRest(value=Feature.NullTo404)
     @RequestMapping(value="/v1/citys",method=RequestMethod.GET)
     List<City> findAllV1();
     /**
     * 根据gbCode（国标码）获取城市。
     * 中国大陆的每个城市都有一个国标码（GB/T 2260-2002），比如北京:
     * 110000，上海：310000，推荐外键使用国标码来标识一个城市，而不要使用
     * City#getId()。
     * @author huisman
     * @version v1
     * @param gbCode 国标码
     * @return 如果城市不存在，返回null，否则返回对应的城市
     * @since 2016-01-01
     * @summary 根据国标码获取城市
     */
     @LoginNeedless
     @LorikRest(value=Feature.NullTo404)
     @RequestMapping(value="/v1/citys/{gbCode}",method=RequestMethod.GET)
     City findByGbCodeV1(@PathVariable(value="gbCode")int gbCode);
     
    /**
     * 新增城市
     * @author huisman
     * @version v1
     * @param cityName 城市名
     * @param gbCode 国标码 
     * @param latitude 维度
     * @param longitude 经度
     * @param cuser 创建人工号
     * @return 返回新增成功的城市，包含ID。
     * @since 2016-01-01
     * @summary 新增城市
     */
     @RequestMapping(value="/v1/citys/add",method=RequestMethod.POST)
     City addV1(@RequestParam(value="cityName")String cityName,
     @RequestParam(value="gbCode")int gbCode,
@RequestParam(value="latitude",defaultValue="0",required=false)double latitude,
@RequestParam(value="longitude",defaultValue="0",required=false)double longitude,
@RequestHeader(StandardHttpHeaders.X_Login_UserCode)int cuser);
  }

```

最后的最后，对所有版本的SPI，别忘了给客户端提供统一访问入口: 不带版本号后缀的SPI

```java
@FeignClient("loupan-server")
public interface CitySpi extends CitySpiV1{
}

```




