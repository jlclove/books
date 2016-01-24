## Spring Java Config

Spring Framework 3.0版本之后，提供了基于Java代码的方式配置Bean (以下统称Java Config)。

Java Config支持:
 * 类上添加标注`org.springframework.context.annotation.Configuration`，`@Configuration`标注指明此类主要用于配置Spring Bean，就好比 `@Service`标注某个类是服务。 
 * 方法上添加标注`org.springframework.context.annotation.Bean`，`@Bean`表明此方法将被Spring用于实例化一个Bean，可以看成，我们手动new了一个对象，然后自动安装到Spring 容器里。

比如以下代码：

``` java
@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}

```

等同于以前在applicationContext.xml中声明：

``` xml
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>

```

简单来说，以前在Spring applicationContext.xml里配置的任何语义都可以使用Java代码来表达。

接下来我一一列举，一些常见的写法：

#### 完整示例
``` java
@Bean(
	autowire=Autowire.BY_NAME,
	destroyMethod="close",
	initMethod="init",
	name={"dataSource"}
    )
 //@Scope(value="prototype")
  @Scope(value="singleton",proxyMode=ScopedProxyMode.TARGET_CLASS)
  @DependsOn(value={"otherBeanName"})
  @Primary
  @Lazy
  public DataSource data(){
     return DataSourceBuilder.create().build();
  }
```

* name Bean的名称，默认是方法名，我们可以指定一个或多个（别名）。
* initMethod 指定初始化时回调方法。 
		类似： Bean bean=new Bean;  
			  bean.init();
* destroyMethod 指定Bean销毁时回调方法。
* autowire Spring自动注入的方式，默认Autowire.No，取决容器实现。
* @DependsOn 依赖的Bean的name
* @Primary 如果自动注入时有多个候选Bean（有歧义时），会优先选择@Primary标注的
* @Lazy 延迟加载


