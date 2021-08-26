## 给学妹看的SpringIOC 面试题（下）

https://mp.weixin.qq.com/s/4z9V4-k2Whqiz69gXy5iMQ

之前上篇跟学弟学妹讲了一下SpringIOC的启动流程，今天接着给学妹聊聊**DI—Dependency Injection**(依赖注入)

### [给学妹看的SpringIOC 面试题（上）](https://mp.weixin.qq.com/s?__biz=MzAwNDA2OTM1Ng==&mid=2453153025&idx=1&sn=4094d005d59d78a96bc7d9d0a4f58bf0&chksm=8cfd0782bb8a8e94f6d87e9cf2c4b07def2e23b0e5b05ce07a5d4ac85cc43c9403d33d8fbee0&token=946915165&lang=zh_CN&scene=21#wechat_redirect)

### 什么是**依赖注入？**

> 依赖注入(DI)是一个过程，通过该过程，对象只能通过构造函数参数，工厂方法的参数或在构造或创建对象实例后在对象实例上设置的属性来定义其依赖关系(即，与它们一起工作的其他对象)。从工厂方法返回。
>
> 然后，容器在创建 bean 时注入那些依赖项。从根本上讲，此过程是通过使用类的直接构造或服务定位器模式来自己控制其依赖关系的实例化或位置的 Bean 本身的逆过程(因此称为 Control Inversion)。
>
> 使用 DI 原理，代码更简洁，当为对象提供依赖项时，去耦会更有效。该对象不查找其依赖项，也不知道依赖项的位置或类。结果，您的类变得更易于测试，尤其是当依赖项依赖于接口或抽象 Base Class 时，它们允许在单元测试中使用存根或模拟实现。
>
> -----------以上解释来源Spring官方文档

说白了依赖注入只是把bean添加到IOC容器的一种方式。

从依赖注入的方式来说整体可以分为两大类来处理，一种是手动方式，一种是自动方式。

> - 手动方式：
>
> - - XML 资源配置元信息（比较常见）
>   - Java 注解配置元信息 （比较常见）
>   - API 配置元信息（不太常用）
>
> - 自动方式：
>
> - - Autowiring

依赖注入的方式有上面的两种，但是也可按注入的类型来区分：

> - Setter注入
> - 构造器注入
> - 接口注入
> - 方法注入

聊到依赖注入那么首先需要先聊聊 **Autowiring Modes**自动绑定模式

Spring的官方文档中对Autowiring Modes解释是：

> Spring 容器可以自动装配协作 bean 之间的关系。通过检查 ApplicationContext 的内容，您可以让 Spring 自动为您的 bean 解析协作者（其他 bean）

同时也提出了4种自动装配模式

> - no：(默认)无自动装配。Bean 引用必须由`ref`元素定义。对于大型部署，建议不要更改默认设置，因为明确指定协作者可以提供更好的控制和清晰度。在某种程度上，它记录了系统的结构。
> - byName：按属性名称自动布线。Spring 寻找与需要自动装配的属性同名的 bean。例如，如果一个 bean 定义被设置为按名称自动装配，并且包含一个`master`属性(即，它具有`setMaster(..)`方法)，那么 Spring 将查找一个名为`master`的 bean 定义并使用它来设置属性。
> - byType：如果容器中恰好存在一个该属性类型的 bean，则使该属性自动装配。如果存在多个错误，则会引发致命异常，这表明您可能不对该 bean 使用`byType`自动装配。如果没有匹配的 bean，则什么也不会发生(未设置该属性)。
> - constructor：类似于`byType`，但适用于构造函数参数。如果容器中不存在构造函数参数类型的一个 bean，则将引发致命错误。

虽然官方文档提出了Autowiring自动绑定方式，但是在我们的真实的业务场景中，相对来说是用的比较少的，因为它有一定的局限性，而且Spring官方文档中也列出了其中的不足点。

##### 自动装配的局限性和缺点(官方文档链接)

> - `property`和`constructor-arg`设置中的显式依赖项始终会覆盖自动装配。您不能自动连接简单属性，例如基元，`Strings`和`Classes`(以及此类简单属性的数组)。此限制是设计使然  PS:针对这种情况可以通过另外的一种方式@value等进行转化来处理这个场景。
> - 自动装配不如显式接线精确。尽管如前所述，Spring 还是小心避免在可能产生意外结果的模棱两可的情况下进行猜测。SpringManagement 的对象之间的关系不再明确记录。
> - 容器内的多个 bean 定义可能与要自动装配的 setter 方法或构造函数参数指定的类型匹配。对于数组，集合或`Map`实例，这不一定是问题。但是，对于需要单个值的依赖项，不会任意解决此歧义。如果没有唯一的 bean 定义可用，则引发异常。

说完这么多文档的基础知识，那么接下来就是开始demo测试环节，来加深理解一下上面的说的那么多到底是个啥。

#### Setter

先从注入的类型先分析怎么样的一种方式叫`Setter`方式注入

```java
/构建一个测试Service
public class SetterServiceInjection {
    public void testMethod(String param) {
        System.out.println(param);
    }
}

public class SetterServiceInjectionTest {
    private SetterServiceInjection setterServiceInjection;

    // Setter方式注入
    public void setSetterServiceInjection(SetterServiceInjection setterServiceInjection) {
        this.setterServiceInjection = setterServiceInjection;
    }

    public void testMethod(){
        setterServiceInjection.testMethod("Setter方式注入");
    }

  
  // 测试启动demo
    public static void main(String[] args) {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
        //获取IOC容器中的bean
        SetterServiceInjectionTest serviceInjectionTest = (SetterServiceInjectionTest) applicationContext.getBean("setterServiceInjectionTest");
        serviceInjectionTest.testMethod();
       // 结果打印：
       // Setter方式注入
       
    }
}
```

xml文件配置

```xml
<bean id="setterServiceInjection" class="com.ao.bing.demo.spring.ioc.SetterServiceInjection"/>

<!--Setter方式-->
<bean id="setterServiceInjectionTest" class="com.ao.bing.demo.spring.ioc.SetterServiceInjectionTest">
 <property name="setterServiceInjection" ref="setterServiceInjection"/>
</bean>
```

上面是很常见的一种注入方式，而且这种方式常见于去写一些配置文件、插件二方包、或者注入数据源信息等。

当然Setter不是仅仅只是这一种使用方式，还可以注入对象，或者说注入一些集合信息等等。

#### 构造器注入

在代码的实现上面构造器和Setter方式是很相似的。还是按照上面的代码改造一下如下所示

```java
rivate final SetterServiceInjection setterServiceInjection;

    // Setter方式注入
//    public void setSetterServiceInjection(SetterServiceInjection setterServiceInjection) {
//        this.setterServiceInjection = setterServiceInjection;
//    }

    public void testMethod(){
        setterServiceInjection.testMethod("构造器方式注入");
    }

    //构造器注入
    public SetterServiceInjectionTest(SetterServiceInjection setterServiceInjection){
        this.setterServiceInjection = setterServiceInjection;
    }
```

```xml
 <context:component-scan base-package="com.ao.bing.demo"/>

    <bean id="setterServiceInjection" class="com.ao.bing.demo.spring.ioc.SetterServiceInjection"/>

    <!--Setter方式-->
<!--    <bean id="setterServiceInjectionTest" class="com.ao.bing.demo.spring.ioc.SetterServiceInjectionTest">-->
<!--        <property name="setterServiceInjection" ref="setterServiceInjection"/>-->
<!--    </bean>-->

    <bean id="setterServiceInjectionTest" class="com.ao.bing.demo.spring.ioc.SetterServiceInjectionTest">
        <constructor-arg index="0" ref="setterServiceInjection"/>
    </bean>
```

既然两个代码这么相似，为什么Spring官方还需要推荐使用这种方式呢？和`Setter`方式区别又是啥？

> - 推荐原因：从定义的属性来说添加了`final`修饰说明我们注入的**依赖不能再变动**。其次从XML的配置bean的属性来说，当需要实例化setterServiceInjectionTest这个类的时候已经实现了有参构造函数，那么就不会再使用默认的构造函数，同时针对传入的参数需要确保有这种类型的值，否则就会报错，所以这样就保证了**依赖不会为空**最后因为构造器传入的参数是确定有值的，那就意味着构造属性是已经**完全初始化的状态**，所以这也就避免了后面需要分析的循环依赖的问题。
>
> - 区别
>
> - - 在Setter注入,可以将依赖项部分注入,构造方法注入不能部分注入
>   - 使用setter注入不能保证类的所有的属性都注入进来。
>   - 在类对象相互依赖的时候可以通过Setter方式解决循环依赖问题。

#### 接口回调注入

提供`Spring`中获取容器本身的一些功能资源，就是通过实现一系列Spring Aware接口来实现具体的功能。

> - BeanFactoryAware：获取 IoC 容器 - BeanFactory
> - ApplicationContextAware：获取 Spring 应用上下文 - ApplicationContext 对象
> - EnvironmentAware：获取 Environment 对象
> - ResourceLoaderAware：获取资源加载器 对象 - ResourceLoader
> - BeanClassLoaderAware：获取加载当前 Bean Class 的 ClassLoader
> - BeanNameAware：获取当前 Bean 的名称
> - MessageSourceAware：获取 MessageSource 对象，用于 Spring 国际化
> - ApplicationEventPublisherAware：获取 ApplicationEventPublishAware 对象，用于 Spring 事件
> - EmbeddedValueResolverAware：获取 StringValueResolver 对象，用于占位符处理

上面的接口回调实现方式也比较简单，基本所有的bean都能实现Aware接口，但是实现Aware接口也有一定的局限性，不能进行扩展只能是进行内嵌，所以理解这就是一种内建的回调方式。

以`ApplicationContextAware`实现代码为例如下图所示

```java
@Component
public class SetterServiceInjectionTest implements ApplicationContextAware {

//    @Autowired
//    private SetterServiceInjection setterServiceInjection;

    private ApplicationContext applicationContext;

    public void testMethod() {
        SetterServiceInjection setterServiceInjection = (SetterServiceInjection) applicationContext.getBean("setterServiceInjection");
        setterServiceInjection.testMethod("接口回调");
    }

    public static void main(String[] args) {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
        //获取IOC容器中的bean
        SetterServiceInjectionTest serviceInjectionTest = (SetterServiceInjectionTest) applicationContext.getBean("setterServiceInjectionTest");
        serviceInjectionTest.testMethod();
    }

    // 获取上下文
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

#### 方法注入

方法注入实现方式可以分为四种：

> - @Autowired：是Spring自带的注解,依照类型进行装配。
> - @Bean：产生一个Bean对象，然后这个Bean对象交给Spring管理。
> - @Resource：@Resource`是JavaEE的标准，Spring对它是兼容性的支持，依照名称进行装配。
> - @Inject（不常见）：jsr330中的规范。

以常见的Autowired为例

```java
  @Autowired 
    private SetterServiceInjection setterServiceInjection;

    public void testMethod(){
        setterServiceInjection.testMethod("方法注入");
    }
    public static void main(String[] args) {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
        //获取IOC容器中的bean
        SetterServiceInjectionTest serviceInjectionTest = (SetterServiceInjectionTest) applicationContext.getBean("setterServiceInjectionTest");
        serviceInjectionTest.testMethod();
    }
```

从上面的代码中并不需要再写一些构造方法，也不用配置相关XML文件只要简单的加上`@Autowired`一个注解就能完成bean的相互关联。

所以方法注入可以理解不用关心方法名称也不用关心方法类型，只要方法上面在参数里面有相关的依赖类型同时加上`@Autowired`或者  `@Resource` 就能相关联上。

#### 类型选择

上面介绍了这么多类型，那么应该怎么合理的选择哪个依赖的注入类型呢？

> - 构造器注入：强制依赖类型，低依赖。
> - Setter 方法注入：非很强的强制依赖类型（无依赖顺序），多依赖。
> - 方法注入：常用于声明类
> - 接口回调注入：业务中常用于写一些主键啥的。

合理的选择注入类型能减少业务开发环境中的很多的问题。

在真实的业务场景中还会遇到另外的一个问题，就是多个类型相同的bean注册到Spring容器中，那么仅仅使用上面的几种方式Spring框架则会抛出`NoUniqueBeanDefinitionException`异常，所以为了解决上述的问题Spring提出了一个新的注解**@Qualifier**来指定哪一个bean或者实现bean的逻辑分组，其用法也相对来说比较加单

```java
public class QualifierDemo {

    @Autowired
    private List<Demo> demos; // 1 ,2,3,4 全部都有

    @Autowired
    @Qualifier 
    private List<Demo> demosQualifier; // 只有 3，4

    @Autowired
    @Qualifier("demo2")
    private Demo demo1; // 只有2

    @Bean
    public Demo demo1() {
        return new Demo(1);
    }
    @Bean
    public Demo demo2() {
        return new Demo(2);
    }
    @Bean
    @Qualifier // 进行逻辑分组
    public Demo demo3() {
        return new Demo(3);
    }
    @Bean
    @Qualifier // 进行逻辑分组
    public Demo demo4() {
        return new Demo(4);
    }
    @Data
    public class Demo {
        private Integer id;
        public Demo (Integer id){
            this.id =id;
        }
    }
}
```

通过上面的代码就能很明确的知道没有使用Qualifier注解的默认就是加载了所有的，使用了Qualifier注解的demosQualifier的里面只有 demo3 和 demo4两个，同样也可以指定使用那么bean如demo1所示。

当然这里只介绍了Qualifier的简单实用，在Spring的官方文档中还有一种用法就是实现Qualifier扩展用法，自定义注解，了解Spring Cloud 的同学可以去看看@LoadBalanced这个注解。用法如下

```java
@Target({ElementType.FIELD, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Qualifier
public @interface DemoGroup {
}
```

Spring依赖注入差不多就跟大家聊完了，当然后一些其他的一些比较少见的就不跟大家细聊了，比如说**延迟依赖注入**感兴趣的可以小伙伴可以再去看下，推荐是使用`ObjectProvider`方式来处理。

### 总结

Spring的依赖注入用一句话来说解耦对象之间的依赖关系，通过xml方式或者注解的方式来灵活管理依赖。

看这中框架性的东西推荐大家可以去看看官方文档，如果看不懂的英文的可以去找找中文翻译过的，来加深自己的理解。(中文官方文档链接)。

接下来剖析一下Spring中的3层缓存怎么去解决的循环依赖。

为了加深理解还给大家整理了一下几个面试题。

构造器注入和 Setter 注入有啥区别?更推荐什么方式？

> 答案已经在文中构造器的解释中给说出来了

怎么解决多个类型相同的bean注册到Spring容器的使用问题？

> 可以使用Qualifier注解来实现

参考文档：中文官方文档、《小马哥核心编程》。

最近在搞的面试版PDF真的觉得还挺有意思的，等搞出来了，应该可以让大家面试前突击突击，对了面试视频筹划中了，这次准备用不同的风格演绎，下个月肯定能出来。