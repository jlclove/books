<!-- toc -->
### SPI包结构
SPI由```model、业务码、枚举以及接口```组成，但我们必须考虑如何对接口功能进行```版本控制```。

如果我们公开一个方法，那么客户端很可能基于此方法进行业务设计，这就意味着客户端隐式地和我们耦合在了一起。

另外，不同的业务线依赖的SPI的jar版本可能不同，我们将面临多版本SPI并存的情况。你无法强制客户端升级，因为新的SPI的model可能有调整，升级之后，客户端原有的逻辑也必须重构一遍。

所以我们必须对接口方法进行版本控制，而且要做好向后兼容。

考虑到以上情况，我们SPI包结构如下：

``` java
com.lianjia.sh.loupan.spi
     source //业务枚举包
         ImageType.class //业务枚举
	 model  //model包
	 	  City.class
	 	  Company.class
	   
     LoupanBizCode.class  //SPI的业务码
     CitySpi.class      //SPI接口 
     CompanySpi.class  //SPI接口
```

#### 包前缀规范   
首先，package前缀的规范: `com.lianjia.sh.项目名.功能模块名.spi`。也可以说，包前缀为SPI模块名。如果功能简单，则可以忽略功能模块。下面，我列举几个正确的包前缀：

*  `com.lianjia.sh.loupan.search.spi`
*  `com.lianjia.sh.loupan.core.spi`
*  `com.lianjia.sh.loupan.statistics.spi`

#### 子包目录
子包目录有model、枚举包、其他等。

所有的SPI接口和业务码属于`com.lianjia.sh.项目名.功能模块名.spi`包，和model等子包层级并列。

#### 包结构示例
下面示例为loupan-search-spi 模块的包结构：

``` java
com.lianjia.sh.loupan.search.spi
    source
     	  ImageSource.class     —- 图片类型
     	  OtherEnumSource.class —- 其他业务枚举
    model
     	  City.class           —- 业务实体
     	  Resblock.class       —- 业务实体
     	  OtherEntity.class    —- 业务实体 
     	  	 	
    LoupanBizCode.class    -- 楼盘字典业务码
    ResblockSpi.class      —- 公开出去的SPI
    CitySpi.class      	   —- 公开出去的SPI
    OtherSpi.class         —- 其他SPI接口
```

### 接口方法

为了对SPI接口的方法进行版本控制，我们约定：**所有接口的方法都要添加版本号后缀**，比如 findCitysV1、findCitysV2、autoSearchV2。 

### 业务码

一般REST接口都会声明一系列业务错误码，这便于客户端定位问题。

lorik-spi-view提供了标准业务码:`com.dooioo.se.lorik.spi.view.support.BizCode`，推荐集中声明在接口中（**如果业务码字段不是public&& static ，自动生成文档时将忽略**）。

注意：code值在`900000-1000000`之间的业务码是API网关及通用框架抛出的错误。

BizCode有个静态方法: `BizCode.toCodes(BizCode...codes)`将多个业务码合并成以逗号隔开的字符串，可复制到`LorikRest(codes={code1,code2,…})`里，以声明此方法可能抛出的业务码值，此处的codes为int型数组，是BizCode的code值，也可以声明为int型静态常量，示例如下：

``` java
 //code常量
 public interface Codes{
        int CITY_NOT_EXISTS=210000;
         …..
 }
 
//业务码集中声明
public interface XXXBizCode{
	BizCode CITY_NOT_EXISTS=new BizCode(Codes.CITY_NOT_EXISTS, "城市不存在");
		…
}
     
//SPI的方法，一个微服务可能有很多业务码，但我们可声明当前方法抛出的业务码
@LorikRest(value={Feature.NullTo404},
	codes={Codes.CITY_NOT_EXISTS})
    @RequestMapping(value=“/v1/citys/{gbCode}”,method=RequstMethod.GET)
    public City findByIdV1(@PathVariable(value=“gbCode”)int gbCode);
```

SPI业务码类的命名，推荐为：`XXXBizCode`，前缀一般为SPI模块名。

下面示例为mini楼盘的业务码：

``` java

package com.lianjia.sh.samples.loupan.spi;

import com.dooioo.se.lorik.spi.view.support.BizCode;

/**
  * 楼盘标准业务码
  * @author huisman 
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

``` java
  
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



    



