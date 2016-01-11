<!-- toc -->
### 项目结构
切换成微服务之后，和我们之前的开发有何不同呢？
![代码架构对照图]({{book.imagePath}}/parts/chapter1/images/code-arch-comp.png)

直观上最大的不同是项目由原来的单个应用划分成了三个项目（模块）：

* dao层蜕变成了（数据）服务提供方，只有服务提供方负责数据的访问和持久化。
* service层蜕变成了SPI，声明了一系列接口，定义了标准契约（model)。
* controller和视图层蜕变成了客户端。客户端有两种方式访问数据，一种是直接依赖SPI 发布的jar，比如 loupan-spi-1.0.1.jar，另一种方式是通过API网关，直接http访问服务，这种通常为web ajax或移动app。

对于服务方提供方，一个服务包含两个模块：Service SPI 和Service SPI的实现。服务提供方应该提供标准服务，保持独立，面向所有客户端。

即使是自己产品线的UI项目，也仅仅只是一个客户端而已，而不能针对内部项目提供特殊（苟且）逻辑。我们的目标是，第三方团队也能基于我们的服务快速开发出新产品。

### 服务拆分
一个业务可能拆分多个服务，至于服务的粒度，各业务线自己拆分。

一个参考原则是高内聚，即服务可独立升级、随意部署，不会过多依赖其他模块或服务（基础设施，比如缓存、消息队列，不在此列），高关联度的业务代码组织在一起，形成一个服务。

另一个参考原则是业务功能的分级，那些业务是关键服务，那些业务是
辅助业务，以此隔离成几个服务。



我们的mini楼盘字典，只演示了一个服务，该项目使用Maven多模块的方式组织代码，如下所示：

![楼盘字典模块图]({{imagePath}}/parts/chapter1/images/project-modules.png)

### 模块命名约定
我们将服务提供方称为server，客户端如果是web项目，则称为ui，否则称为client，服务声明则称为spi。

模块命名规则为： **项目名**-**功能模块名**-**角色（spi|server|ui|client）**。
如果项目功能简单，也可以忽略功能模块名。

拿mini楼盘字典来举例，我们的项目名叫loupan，因为功能简单，没有把项目进一步划分为多组功能模块（多个服务），仅提供一个服务。单个服务对应微服务的三个层级，各模块命名如下：

* loupan-spi  服务声明
* loupan-server 服务实现
* loupan-ui  web界面，交互等

如果业务复杂，也可以进一步按功能划分，比如我们可以把楼盘划分为以下三个服务，对应的就有三个server：
  
服务声明（SPI)

* loupan-search-spi        楼盘搜索服务
* loupan-core-spi          楼盘核心服务
* loupan-statistics-spi    楼盘统计服务

服务实现(Service Provider)

* loupan-search-server     搜索服务实现
* loupan-core-server       核心业务实现
* loupan-statistics-server 统计业务实现

客户端（Client/Web UI)

* loupan-ui    
	web界面，交互等，可以调用楼盘自身的服务（loupan-search-spi、loupan-core-spi、loupan-statistics-spi），也可能调用其他项目线的服务，比如oms-core-spi，xingcheng-core-spi。
* loupan-management-ui 系统运行一段时间后，新加了楼盘的运营、管理等web功能。





