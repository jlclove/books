## Spring Java Config
%E5%BE%B3%E4%BD%91 、%E5%BE%B7%E4%BD%91

Spring 3.0之后，提供了基于Java代码的方式配置Bean (以下统称Java Config)。

Java Config支持:
 * 类上添加注解`org.springframework.context.annotation.Configuration`，`@Configuration`标注指明此类主要用于配置Spring Bean，就好比 `@Service`标注某个类是服务。 
 * 方法上添加注解`org.springframework.context.annotation.Bean`，`@Bean`表明此方法将被Spring用于实例化一个Bean，可以看成，我们手动new了一个对象，然后自动安装到Spring 容器里。

以下代码：

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
  public DataSource dataSource(){
     return DataSourceBuilder.create().build();
  }
```

|  属性  | 说明 | 默认值 |
| :------------ | :-----------| :----------- | 
|name| Bean的名称，我们可以指定一个或多个（别名） | 方法名|
| initMethod| Bean实例化时的回调方法，等同于：<br>`Bean bean=new Bean(…); bean.init();`| 无|
| destroyMethod| Bean销毁时的回调方法 |public的close/shutdown方法，设置为空字符串(“”)可禁用默认值|
| autowire| Spring IOC自动注入的方式:ByName/ByType|Autowire.No|
|@Scope| Bean的作用域，Spring内置支持：prototype、singleton |singleton|
| @DependsOn | Bean依赖的Bean的名称，可有多个值|无|
| @Primary| 如果自动注入时有多个候选Bean（有歧义时），会优先选择有@Primary注解的     |无|
| @Lazy| 指明此Bean在其他Bean第一次访问时初始化，所谓延迟加载     |Spring BeanFactory启动时实例化所有单例Bean|

#### 依赖注入






