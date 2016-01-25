<!-- toc -->

### SPI 接口规范 

*  接口类名必须以“Spi”为后缀，比如：`ResblockSpi`，接口不允许有父接口。  

* ```RequestMapping``` 只能添加在方法上，不能添加在接口上。  
  RequestMapping的value以版本号为前缀(“/version/path”)，method必须提供并且只能限定为一种，比如：  
```@RequestMapping(value = "/v1/resblocks", method = RequestMethod.GET）```  
  
* 接口方法的名称必须以版本号为后缀，比如：```searchV1（）```，```findByIdV2()```;  

* 接口方法的参数，只能是8种基本类型以及枚举值，可变参数仅限枚举类型，并且方法的参数必须添加注解```@RequestParam```、```@PathVariable```、```@RequestHeader```。  


* 接口方法必须提供完善且符合要求的注释，以便自动生成API文档。  

* 接口方法默认会进行登录校验，如果无需登录即可访问，请添加注解```@LoginNeedless```  

* SPI接口最好生成Java Doc注释，推荐添加@summary，一般是此SPI的简介。
### 代码示例
下面示例为mini楼盘字典楼盘SPI的声明：

``` java
package com.lianjia.sh.samples.loupan.spi;
//省略spring mvc annotation
import com.dooioo.se.lorik.spi.view.Pagination;
import com.dooioo.se.lorik.spi.view.authorize.LoginNeedless;
import com.dooioo.se.lorik.spi.view.support.LorikRest;
import com.dooioo.se.lorik.spi.view.support.LorikRest.Feature;
import com.lianjia.sh.samples.loupan.spi.model.Resblock;

/**
 * 楼盘SPI,楼盘是从北京链家同步过来的，目前仅支持苏州、上海的楼盘查询。
 * @summary 楼盘
 * @Copyright (c) 2016, Lianjia Group All Rights Reserved.
 */
@FeignClient("loupan-server")
public interface ResblockSpi {

  /**
   * 根据城市国标码对楼盘搜索，可根据区域ID、商圈ID分页检索楼盘。
   * 
   * @author huisman
   * @version v1
   * @param gbCode 城市对应的国标码
   * @param bizcircleId 商圈ID
   * @param districtId 行政区域ID
   * @param pageSize 分页大小
   * @param pageNo 当前页码
   * @return
   * @since 2016-01-01
   * @summary 根据gbCode分页检索楼盘
   */
  @LoginNeedless
  @RequestMapping(value = "/v1/resblocks", method = RequestMethod.GET, params = "gbCode")
  Pagination<Resblock> searchV1(@RequestParam("gbCode") int gbCode,
      @RequestParam(value = "districtId", required = false) Integer districtId,
      @RequestParam(value = "bizcircleId", required = false) Integer bizcircleId,
      @RequestParam(value = "pageNo", defaultValue = "1") int pageNo,
      @RequestParam(value = "pageSize", defaultValue = "30") int pageSize);

  /**
   * 根据城市国标码以及楼盘关键字信息自动提示楼盘信息，最多返回 size（默认20）条数据。
   * 
   * @author huisman
   * @version v1
   * @param keyword 楼盘关键字
   * @param gbCode 城市国标码
   * @param size 返回的结果数，默认20
   * @return 不存在则返回空List
   * @since 2016-01-01
   * @summary 楼盘自动提示
   */
  @LoginNeedless
  @RequestMapping(value = "/v1/resblocks/autoSearch", method = RequestMethod.GET)
  List<Resblock> autoSearchV1(@RequestParam(value = "keyword", defaultValue = "") String keyword,
      @RequestParam(value = "gbCode") int gbCode,
      @RequestParam(value = "size", defaultValue = "20") int size);

  /**
   * 根据楼盘ID查找楼盘
   * @author huisman
   * @version v1
   * @param id 楼盘ID
   * @return
   * @since 2016-01-01
   * @summary 根据ID查找楼盘
   */
  @LoginNeedless
  @LorikRest(value = {Feature.NullTo404}, codes = {"23000:楼盘不存在","22000:商圈不存在"})
  @RequestMapping(value = "/v1/resblocks/{id}", method = RequestMethod.GET)
  Resblock findByIdV1(@PathVariable(value = "id") int id);
}

```

### 注释模板
SPI方法的注释模板可在IDE中配置，以下是Eclipse Code Comment配置：

``` java
/**
  * 
  * @author huisman
  * @version v1
  * ${tags}
  * @since ${date}
  * @summary 
  */
```

SPI类的Eclipse注释模板：

``` java
/**
  *
  * @summary 
  * @since ${date}
  * @Copyright (c) {year}, Lianjia Group All Rights Reserved.
  */

```



### loupan-spi模块  

loupan-spi模块代码： [GitHub loupan-spi](https://github.com/bookdao/samples/tree/master/springcloud-for-sh-lianjia-se/loupan-spi/src/main/java/com/lianjia/sh/samples/loupan/spi)，代码结构如下图所示：
 ![loupan-spi模块]({{book.imagePath}}/parts/chapter2/images/spi-code.png)
 
 
 
还有一点需要注意：SPI接口要添加注解```@FeignClient(“server name”)```，server name是接口实现方的```spring.application.name```，一般是实现方模块名，代码示例如下：

``` java
@FeignClient("loupan-core-server")
public interface BizcircleSpi {
}

@FeignClient("loupan-statistics-server")
public interface CitySpi{
}

@FeignClient("loupan—search-server")
public interface BizcircleSpi{
}

```
 
 