### 项目结构
在切换成微服务架构之后，和我们之前开发有何不同呢？
![代码架构对照图](https://raw.githubusercontent.com/bookdao/books/master/springcloud-for-sh-lianjia-se/parts/chapter1/images/code-arch-comparison.png)


对于服务方提供方，一个服务基本分为两个模块：Service SPI 和Service SPI 实现。

但是请注意，一个业务可能划分多个服务，至于服务的粒度，决定权在各业务线。

一个参考原则是高内聚，即服务可独立升级、随意部署，不会过多依赖其他模块或服务（基础设施，比如缓存、消息队列，不在此列），高关联度的业务代码组织在一起，形成一个服务。

另一个参考原则是业务功能的分级，那些业务是关键服务，那些业务是
辅助业务，以此隔离成几个服务。

我们的mini楼盘字典，只演示了一个服务，该项目使用Maven多模块的方式组织代码，如下所示：

![楼盘字典模块图](https://raw.githubusercontent.com/bookdao/books/master/springcloud-for-sh-lianjia-se/parts/chapter1/images/project-modules.png)

##### 模块命名约定
那mini
       






