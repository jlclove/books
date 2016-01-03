### SPI包结构
SPI由model、业务枚举以及接口组成，但我们必须考虑如何对接口功能进行**版本控制**。

如果我们公开一个方法，那么客户端很可能基于此方法进行业务设计，这就意味着客户端隐式地和我们耦合在了一起。

另外，不同的业务线依赖的SPI的jar版本可能不同，我们将面临多版本SPI并存的情况。你无法强制客户端升级，因为新的SPI的model可能有调整，升级之后，客户端原有的逻辑也必须重构一遍。

所以我们必须对接口方法进行版本控制，而且要做好向后兼容。

考虑到以上情况，我们SPI包结构如下：

```
com.lianjia.sh.loupan.spi
   	 v1
     	  model
     	  CitySpiV1.class
     v2
     	  model
     	  CitySpiV2.class
     	  
     source 
     CitySpi.class
```
     
首先，package前缀的规范: **com.lianjia.sh.项目名.功能模块名.spi**。也就是说，包前缀为SPI模块名。如果功能简单，则可以忽略功能模块。下面，我列举几个正确的包前缀：

*  com.lianjia.sh.loupan.search.spi
*  com.lianjia.sh.loupan.core.spi
*  com.lianjia.sh.loupan.statistics.spi

其次，按版本号分成几个sub package，比如，v1，v2，v1或v2由model（业务实体）、具体的接口V1|V2.class（留给server实现） 组成。

同时，与v1,v2包并列有source包（业务枚举），以及不带版本号后缀的Spi.class，这是供客户端调用的。

下面示例为loupan-search-spi 模块的包结构：

```
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
     	  
    ResblockSpi.class  —- 客户端调用
    CitySpi.class      —- 客户端调用
    OtherSpi.class     —  客户端嗲用
```

### 接口和方法命名规范
SPI 接口分两种，一种是给客户端调用的，和v1/v2等包平级，这些接口的后缀为”Spi”，是没有版本号的，比如CitySpi.class。另一种是server需要实现的、放在v1/v2包里的SPI接口，这些接口的后缀为：“Spi”+版本，比如CitySpiV1.class、CitySpiV2.class。

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

```
public void doXXX(){
       …some logic
       citySpiV1.xxxx();
       …some logic
       citySpiV2.xxxx();
     }
```

方案B，是将所有版本的API串起来，只对客户端提供一个入口：

```
public void doXXX(){
       …some logic
       citySpi.xxxxV1();
       …some logic
       citySpi.xxxxV2();
     }
```

我们采用了方案B，增加了一类供客户端调用的SPI，它仅仅把一组功能所有版本的SPI串了(extends)起来，并未声明什么新方法。

对客户端来说，他现在不用关心某个功能到底在CitySpiV1里，还是在CitySpiV2里。他只要注入CitySpi，就能拿到所有版本支持的功能。

但此种方案也有不足之处：

*  首先为了避免方法名重复，我们约定，**所有接口的方法都要添加版本号后缀**，比如 findCitysV1、findCitysV2、autoSearchV2。  

*  对客户端来说，虽然所有功能都一览无遗，但也对他们造成了干扰。比如,findCitysV1和findCitysV2，到底该用哪个呢？ 添加版本号是为了向后兼容，这对老项目有益处，而新项目，没有历史包袱，直接使用高版本的方法即可，低版本的存在反而是一种干扰。


在实践的过程中，我们可能会进一步调整方案B。





    



