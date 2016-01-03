### SPI 接口规范 
 SPI接口的规范如下：
 
*  供Server实现接口命名必须以“Spi”+版本号为后缀，比如：ResblockSpiV1，供客户端调用的SPI接口以“Spi”为后缀，比如：ResblockSpi。
* RequestMapping 只能添加在方法上，不能添加在接口上，RequestMapping的value以版本号为前缀(/version/path)，method必须提供并且只能限定为一种，比如：  
```@RequestMapping(value = "/v1/resblocks", method = RequestMethod.GET）```
 * 接口方法的名称必须以版本号为后缀，比如：searchV1（），findByIdV1();
 * 接口方法的参数，只能是8种基本类型以及枚举值，可变参数仅限枚举类型，并且方法的参数必须使用```@RequestParam```、```@PathVariable```、```@RequestHeader```标注。
 * 接口方法必须提供完善且符合要求的注释，以便自动生成API网关里的文档。
 * 接口方法默认会进行登录校验，如果无需登录即可访问，请添加标注```@LoginNeedless```
 
下面示例为mini楼盘字典楼盘SPI的声明：

```

package com.lianjia.sh.samples.loupan.spi.v1;
/**
 * 楼盘SPIV1
 * @author huisman
 * @since v1
 * @Copyright (c) 2016, Lianjia Group All Rights Reserved.
 */
public interface ResblockSpiV1 {

  /**
   *  根据城市国标码对楼盘搜索，可根据区域ID、商圈ID分页检索楼盘。
   * 
   * @author huisman
   * @param gbCode 城市对应的国标码
   * @param bizcircleId 商圈ID
   * @param districtId 行政区域ID
   * @param pageSize 分页大小
   * @param pageNo 当前页码
   * @return 
   * @since v1
   * @summary 根据gbCode分页检索楼盘
   * @example /v1/resblocks?gbCode=310000
   * @errorCode
   */
  @LoginNeedless
  @RequestMapping(value = "/v1/resblocks", method = RequestMethod.GET, params = "gbCode")
  Pagination<Resblock> searchV1(@RequestParam("gbCode") int gbCode,
      @RequestParam(value = "districtId", required = false) Integer districtId,
      @RequestParam(value = "bizcircleId", required = false) Integer bizcircleId,
      @RequestParam(value = "pageNo", defaultValue = "1") int pageNo,
      @RequestParam(value = "pageSize", defaultValue = "30") int pageSize);
}
 
```

#### 方法注释规范

```
  /**
  * 方法详细说明
  * @author 作者名或者产品线，邮件等
  * @param 参数名  参数说明
  * @param 参数名  参数说明
  * @param 参数名  参数说明
  * @return 返回结果的说明
  * @since 方法的版本号，比如v1/v2，小写。
  * @summary 功能概括，一般不超过20字
  * @example  访问示例，最好可以返回结果
  * @errorCode 业务抛出的错误码，没有可为空
  */
  
```

上述方法注释的模板可以在IDE中配置，以下是Eclipse Code Comment配置：

```
/**
  * 
  * @author huisman
  * ${tags}
  * @since v1
  * @summary
  * @example
  * @errorCode
  * 
  */

```



### 代码示例
 ![loupan-spi模块](https://raw.githubusercontent.com/bookdao/books/master/springcloud-for-sh-lianjia-se/parts/chapter2/images/spi-code.png)
 loupan-spi模块代码： [GitHub loupan-spi](https://github.com/bookdao/samples/tree/master/springcloud-for-sh-lianjia-se/loupan-spi/src/main/java/com/lianjia/sh/samples/loupan/spi)。
 
还有一点需要注意：供客户端调用的SPI接口要使用```@FeignClient(“server name”)```，server name是接口实现方的模块名，代码示例如下：

```
@FeignClient("loupan-core-server")
public interface BizcircleSpi extends BizcircleSpiV1 {
}

@FeignClient("loupan-statistics-server")
public interface CitySpi extends SomeSpiV1,SomeSpiV2 {
}

@FeignClient("loupan—search-server")
public interface BizcircleSpi extends SomeSpiV1,SomeSpiV2,SomeSpiV3 {
}

```
 
 