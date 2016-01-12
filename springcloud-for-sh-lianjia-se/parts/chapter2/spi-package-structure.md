<!-- toc -->
### SPI包结构
SPI由```model、业务枚举以及接口```组成，但我们必须考虑如何对接口功能进行```版本控制```。

如果我们公开一个方法，那么客户端很可能基于此方法进行业务设计，这就意味着客户端隐式地和我们耦合在了一起。

另外，不同的业务线依赖的SPI的jar版本可能不同，我们将面临多版本SPI并存的情况。你无法强制客户端升级，因为新的SPI的model可能有调整，升级之后，客户端原有的逻辑也必须重构一遍。

所以我们必须对接口方法进行版本控制，而且要做好向后兼容。

考虑到以上情况，我们SPI包结构如下：

``` java
com.lianjia.sh.loupan.spi
     v1
     	  model
     	  
     	  CitySpiV1.class
     v2
     	  model
    
     	  CitySpiV2.class
     	  
     source   

     LoupanBizCode.class
     CitySpi.class
```

#### 包前缀规范   
首先，package前缀的规范: `com.lianjia.sh.项目名.功能模块名.spi`。也就是说，包前缀为SPI模块名。如果功能简单，则可以忽略功能模块。下面，我列举几个正确的包前缀：

*  `com.lianjia.sh.loupan.search.spi`
*  `com.lianjia.sh.loupan.core.spi`
*  `com.lianjia.sh.loupan.statistics.spi`

#### 子包目录
其次，按版本号分成几个sub package，比如，`v1`，`v2`。

v1或v2由model（业务实体）、具体版本的SPI接口V1/V2.class（留给server实现） 组成。

同时，与v1,v2包并列有source包（业务枚举）、`业务码BizCode.class`，以及不带版本号后缀的Spi.class，这是供客户端调用的。

#### 包结构示例
下面示例为loupan-search-spi 模块的包结构：

``` java
com.lianjia.sh.loupan.search.spi
   	 v1
     	  model
     	 	    City.class           —- v1版本的业务实体
     	  	 	Resblock.class       —- v1版本的业务实体
     	  	 	OtherEntity.class    —- v1版本的业务实体
     	 
     	  CitySpiV1.class       —-  服务提供方 server实现
     	  ResblockSpiV1.class   -—  服务提供方 server实现
     	  OtherSpiV1.class      —-  服务提供方 server实现
     v2
     	  model
     	  		Resblock.class       —- v2业务实体，数据结构调整
     	  		OtherEntity.class    —- v2业务实体，新增或者结构调整
      
          ResblockSpiV2.class  —- 服务提供方 server实现
          OtherSpiV2.class     —  服务提供方 server实现
          
    source
     	  ImageSource.class     —- 图片类型
     	  OtherEnumSource.class —- 其他业务枚举
     	  
    LoupanBizCode.class    -- 楼盘REST 业务码
    ResblockSpi.class      —-  客户端调用
    CitySpi.class      	   —-  客户端调用
    OtherSpi.class         —-   客户端嗲用
```

### 接口和方法
SPI 接口分两种，一种是给`客户端调用`的，和v1/v2等包平级，这些接口的后缀为”Spi”，是没有版本号的，比如CitySpi.class。另一种是`server需要实现`的、放在v1/v2包里的SPI接口，这些接口的后缀为：“Spi”+版本，比如`CitySpiV1.class、CitySpiV2.class`。

那么这两类接口有什么关系呢？下面代码演示了他们之间的关联：

```java
   package com.lianjia.sh.loupan.search.spi.v1;
   
   public interface CitySpiV1{
    @RequestMapping(value="/v1/citys",method=RequestMethod.GET)
    List<City> findAllV1();
   }
   
   
   package com.lianjia.sh.loupan.search.spi.v2;
   
   public interface CitySpiV2{
    @RequestMapping(value="/v2/citys",method=RequestMethod.GET)
    List<NewCity> findAllV2();
   }
   
   package com.lianjia.sh.loupan.search.spi;
   @FeignClient(“loupan-search”)
   public interface CitySpi extends CitySpiV1,CitySpiV2{
   }
   
```

在进一步解释这两类接口之前，我们思考下客户端该如何使用我们的SPI呢？

方案A，客户端需要知晓每个版本的功能

``` java
public void doXXX(){
       …some logic
       citySpiV1.xxxx();
       …some logic
       citySpiV2.xxxx();
     }
```

方案B，是将所有版本的API串起来，只对客户端提供一个入口：

``` java
public void doXXX(){
       …some logic
       citySpi.xxxxV1();
       …some logic
       citySpi.xxxxV2();
     }
```

我们采用了方案B，增加了一类供客户端调用的SPI，它仅仅把一组功能所有版本的SPI串了(extends)起来，并未声明什么新方法。

对客户端来说，他现在不用关心某个功能到底在`CitySpiV1`里，还是在`CitySpiV2`里。他只要注入`CitySpi`，就能拿到所有版本支持的功能。

但此种方案也有不足之处：

*  首先为了避免方法名重复，我们约定，**所有接口的方法都要添加版本号后缀**，比如 findCitysV1、findCitysV2、autoSearchV2。  

*  对客户端来说，虽然所有功能都一览无遗，但也对他们造成了干扰。比如,findCitysV1和findCitysV2，到底该用哪个呢？ 添加版本号是为了向后兼容，这对老项目有益处，而新项目，没有历史包袱，直接使用高版本的方法即可，低版本的存在反而是一种干扰。


在实践的过程中，我们可能会进一步调整方案B。


### 业务码

一般REST接口都会声明一系列业务错误码，这便于客户端定位问题。

lorik-spi-view提供了标准业务码:`com.dooioo.se.lorik.spi.view.support.BizCode`

BizCode有个静态方法: `BizCode.toCodes(BizCode...codes)`将业务码转为字符串，可复制字符串到`LorikRest(codes={“”})`里，以生成API文档。

SPI业务码类的命名，推荐为：`XXXBizCode`，前缀一般为SPI模块名。

下面示例为mini楼盘的业务码：

```

/**
  * 楼盘标准业务码
  * @author huisman
  * @since 1.0.0  
  * @Copyright (c) 2016, Lianjia Group All Rights Reserved.
*/
public interface LoupanBizCode {
  
    /**
     *  城市不存在。
     */
    BizCode CITY_NOT_EXISTS=new BizCode(21000, "城市不存在");

    /**
     * 商圈不存在
     */
    BizCode BIZCIRCLE_NOT_EXISTS=new BizCode(22000, "商圈不存在");
    
    /**
     *  楼盘不存在
     */
   BizCode REBLOCK_NOT_EXISTS=new BizCode(23000, "楼盘不存在");
    
    
    /**
     *  行政区域不存在
     */
   BizCode DISTRICT_NOT_EXISTS=new BizCode(24000, "区域不存在");
    
}

```

业务码可在服务实现时配合`com.dooioo.se.lorik.core.web.result.Assert`抛出：

```
  
  @Service
  public class CityService{
  
      public void updateInvalid(int id){
          City city=cityDao.findById(id);
          Assert.found(city !=null,LoupanBizCode.CITY_NOT_EXISTS);
          //other logic
      }
  }
  
  
```

### 业务枚举

业务实现所涉及的业务枚举，推荐声明在SPI包里。



    



