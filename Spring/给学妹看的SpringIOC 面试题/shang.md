## 给学妹看的SpringIOC 面试题（上）

https://mp.weixin.qq.com/s/SH4laewpIsio66MUJFLTyg

前段时间是校招的高峰期啊，很多学弟，学妹们出去面试的时候都会被问到一个问题，谈谈你对Spring的理解？

很多同学都是会说一些IOC，AOP等，但是聊到一些细节IOC里面的细节点，就不知怎么接着和面试官怎么聊了。

所以今天我就跟大家具体详细聊聊SpringIOC 那些事！！！

### 什么是Spring

> spring 首先它是一个框架，在我们的开发工作的环境中，所有的其他的框架基本都依赖Spring，spring起着一个容器的作用，用来承载我们整体的bean对象。它帮我们整理了整个bean的从创建到销毁的管理

### IOC控制反转是啥？

> 类的创建、销毁都由 Spring 来控制，也就是说控制对象生存周期的不再是引用它的对象，而是 Spring来控制整个过程。对于某个具体的对象而言，以前是它控制其他对象，现在是所有对象都被 Spring 控制。

看到这里其实这都是一些简单的理解，以及一些官方的说法，为了真正的搞懂什么是SpringIoc,就上面的这些东西是远远不够的，所以我给大家画了一个流程图，跟着这个流程图，我们一步一步来解析IOC。

只有解析完了流程，我们才能有一个整体的架构的脉络思路，后面我们再聊**DI（依赖注入）\**以及怎么处理的\**缓存依赖**。

> 这里跟大家分享一个知识点，在看一些架构的源码的时候，大家一定要先理清整体架构的脉络，这样才能方便我们理解整个架构，否则就是一面茫然，不知道写的是啥！！！

话不多说了，还是直接来看下这个整体流程图！！！

![图片](shang.assets/1)

从这个图，我们还是从上到下，从左到右的顺序来讲解哈

```java
public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
}
```

我们以开始启动spring容器开始，常见配置bean有 XML 配置文件的形式或者注解形式等还有一些其他的方式。

不管哪种方式，spring考虑到扩展性问题，会通过BeanDefinitionReader，来加载bean的配置信息，然后生成一个BeanDefinition（bean的定义信息，用来存储 bean 的所有属性方法定义）

> BeanDefinitionReader 只是接口约束一些定义信息，常见的实现类 XmlBeanDefinitionReader（xml形式），PropertiesBeanDefinitionReader（Properties配置文件），AbstractBeanDefinitionReader （相关一些环境信息）等

#### BeanFactoryPostProcesser

说完了BeanDefinition那么接下来就是走到**BeanFactoryPostProcessor**

> BeanFactoryPostProcessor 接口是 Spring 初始化 BeanFactory 时对外暴露的扩展点，其实就是在bean的实例化之前，可以获取bean的定义信息，以及修改相关信息。

比如说我们现在常见的注解方式来加载bean信息，里面其实就是也是用的BeanFactoryPostProcessor的子类实现的。

我们常见的 @Service、@Controller、@Repository等注解其实都是组合注解，里面里面都是包含Component注解实现的，如下GIF动图所示：

从这个动图中大家可以发现BeanFactoryPostProcessor有一堆的实现子类，因此当我们有自己的业务逻辑实现的时候也只需要实现BeanFactoryPostProcessor就可以了，然后加上@Component注解就可以了。

#### BeanFactory

BeanFactory，从名字上也很好理解，生产 bean 的工厂，它负责生产和管理各个 bean 实例。同时也是Spring容器暴露在外获取bean的入口

> BeanFactory的生产过程其实是利用反射机制实现的。

![图片](shang.assets/2)

接下来我们再来看一下BeanFactory的继承关系

![图片](shang.assets/3)

这张关系图我们只要了解的几个关键点

> - HierarchicalBeanFactory：提供父容器的访问功能
> - ListableBeanFactory：提供了批量获取Bean的方法
> - AutowireCapableBeanFactory：在BeanFactory基础上实现对已存在实例的管理
> - ConfigurableBeanFactory：单例bean的注册以及生成实例，统计单例bean等信息
> - ConfigurableListableBeanFactory：增加了一些其他功能：类加载器、类型转化、属性编辑器、BeanPostProcessor、bean定义、处理bean依赖关系、 bean如何销毁等等一些还有其他的功能
> - DefaultListableBeanFactory：实现BeanFactory所有功能同时也能注册BeanDefinition

可能有人要问了，ApplicationContext和BeanFactory是不是只是继承关系？

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
        BeanFactory factory = new ClassPathXmlApplicationContext("classpath:applicationContext.xml"); 
```

> BeanFactory是一个底层的IOC容器，而ApplicationContext是在其基础上增加了一些它的特性的同时同时增加了一些其他的整合特性比如：更好的整合SpringAOP、国际化消息、以及事务的发布、资源访问等这些新的特性。
>
> 所以BeanFactory和ApplicationContext不是同一个东西，是两个不同的对象，想要获取BeanFactory可以通过applicationContext.getParentBeanFactory()获取。

所以当通过XML来配置bean的信息的时候我们就可以使用BeanFactory作为容器，因为我们不需要有那么多其他的额外的一些特性。当我们通过注解的形式来注册bean信息的时候，我们就可以使用ApplicationContext来作为容器。当然这个只是作为了解，在我们的业务代码中基本是可以不用关心这一点的。

#### Bean的生命周期

Spring Bean的生命周期在spring的面试题中这其实是非常常见的一道面试题，其实并不用去背那么多流程，在Spring的源码中其实已经写好了bean的完整生命流程，上面的**BeanFactory**中已经表明

![图片](shang.assets/4)

> - BeanNameAware#setBeanName：在创建此bean的bean工厂中设置bean的名称，在普通属性设置之后调用，在InitializinngBean.afterPropertiesSet()方法之前调用
> - BeanClassLoaderAware#setBeanClassLoader：将 bean ClassLoaderr 提供给 bean 实例的回调
> - BeanFactoryAware#setBeanFactory：回调提供了自己的bean实例工厂，在普通属性设置之后，在InitializingBean.afterPropertiesSet()或者自定义初始化方法之前调用
> - org.springframework.context.ResourceLoaderAware#setResourceLoader：在普通bean对象之后调用，在afterPropertiesSet 或者自定义的init-method 之前调用，在 ApplicationContextAware 之前调用。
> - org.springframework.context.ApplicationEventPublisherAware#setApplicationEventPublisher：在普通bean属性之后调用，在初始化调用afterPropertiesSet 或者自定义初始化方法之前调用。在 ApplicationContextAware 之前调用。
> - org.springframework.context.MessageSourceAware#setMessageSource：在普通bean属性之后调用，在初始化调用afterPropertiesSet 或者自定义初始化方法之前调用，在 ApplicationContextAware 之前调用。
> - org.springframework.context.ApplicationContextAware#setApplicationContext：在普通Bean对象生成之后调用，在InitializingBean.afterPropertiesSet之前调用或者用户自定义初始化方法之前。在ResourceLoaderAware.setResourceLoader，ApplicationEventPublisherAware.setApplicationEventPublisher，MessageSourceAware之后调用
> - org.springframework.web.context.ServletContextAware#setServletContext：运行时设置ServletContext，在普通bean初始化后调用
> - org.springframework.beans.factory.config.BeanPostProcessor#postProcessBeforeInitialization：将此BeanPostProcessor 应用于给定的新bean实例
> - InitializingBean#afterPropertiesSet：在设置所有 bean 属性后由包含的 BeanFactory调用
> - org.springframework.beans.factory.support.RootBeanDefinition#getInitMethodName：获取InitMethodName名称，并且运行初始化方法
> - org.springframework.beans.factory.config.BeanPostProcessor#postProcessAfterInitialization
> - DisposableBean#destroy：销毁
> - org.springframework.beans.factory.support.RootBeanDefinition#getDestroyMethodName：返回被销毁的bean名称

这其实就是bean的整个生命周期过程，其实这里面注视大家都是可以自己查看的，每一个方法上面都是很详细注释，我也只是根据注视简单的翻译了一下。

整个过程bean的生命周期可以缩短理解为：

![图片](shang.assets/5)

但是要完全理解Spring，那肯定就要说Spring里面的一个非常重要的方法 **ApplicationContext.refresh()**这其中的包含了13个子方法：

```java
public void refresh() throws BeansException, IllegalStateException {
   //   添加一个synchronized 防止出现refresh还没有完成出现其他的操作（启动，或者销毁） 
   synchronized (this.startupShutdownMonitor) {

      // 1.准备工作
      // 记录下容器的启动时间、
      // 标记“已启动”状态，关闭状态为false、
      // 加载当前系统属性到环境对象中
      // 准备一系列监听器以及事件集合对象
       prepareRefresh();

      // 2. 创建容器对象：DefaultListableBeanFactory，加载XML配置文件的属性到当前的工厂中（默认用命名空间来解析），就是上面说的BeanDefinition（bean的定义信息）这里还没有初始化，只是配置信息都提取出来了，（包含里面的value值其实都只是占位符）
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

      // 3. BeanFactory的准备工作，设置BeanFactory的类加载器，添加几个BeanPostProcessor，手动注册几个特殊的bean等
      prepareBeanFactory(beanFactory);
      try {
         // 4.子类的覆盖方法做额外的处理，就是我们刚开始说的 BeanFactoryPostProcessor ，具体的子类可以在这步的时候添加一些特殊的BeanFactoryPostProcessor完成对beanFactory修改或者扩展。
         // 到这里的时候，所有的Bean都加载、注册完成了，但是都还没有初始化
         postProcessBeanFactory(beanFactory);
         // 5.调用 BeanFactoryPostProcessor 各个实现类的 postProcessBeanFactory(factory) 方法
         invokeBeanFactoryPostProcessors(beanFactory);

         // 6.注册 BeanPostProcessor  处理器 这里只是注册功能，真正的调用的是getBean方法
        registerBeanPostProcessors(beanFactory);

         // 7.初始化当前 ApplicationContext 的 MessageSource，即国际化处理
         initMessageSource();

         // 8.初始化当前 ApplicationContext 的事件广播器，
         initApplicationEventMulticaster();

         // 9.从方法名就可以知道，典型的模板方法(钩子方法)，感兴趣的同学还可以再去复习一下之前写的设计模式中的-模版方法模式
         //  具体的子类可以在这里初始化一些特殊的Bean（在初始化 singleton beans 之前）
         onRefresh();

         // 10.注册事件监听器，监听器需要实现 ApplicationListener 接口。这也不是我们的重点，过
         registerListeners();

         // 11.初始化所有的 singleton beans（lazy-init 的除外），重点关注
         finishBeanFactoryInitialization(beanFactory);

         // 12.广播事件，ApplicationContext 初始化完成
         finishRefresh();
      }
      catch (BeansException ex) {
         if (logger.isWarnEnabled()) {
            logger.warn("Exception encountered during context initialization - " +
                  "cancelling refresh attempt: " + ex);
         }

         // 13.销毁已经初始化的 singleton 的 Beans，以免有些 bean 会一直占用资源
         destroyBeans();
        
         cancelRefresh(ex);
         // 把异常往外抛
         throw ex;
      }
      finally {
         // Reset common introspection caches in Spring's core, since we
         // might not ever need metadata for singleton beans anymore...
         resetCommonCaches();
      }
   }
}
```

这里只是大致的说明一下这里的每个方法的用途，如果还想要了解的更深，就需要大家自己再去看这里面的更深成次的代码了，这个大家可以自己尝试的断点试一下。或者后面再单独给大家写一篇这里面的细节流程。

> 断点看源码不必要每个方法都去看，先了解一个大概，然后再多断点几次，每次断点都相对上一次进入的更深成次一点，满满的你就能全部理解了。这是一个漫长的过程。

### 总结

Spring IOC整个启动过程我们就先讲到这里，由于篇幅问题一下子写的太长怕看起来有点难受，后面再接着跟大家分享怎么处理**循环依赖问题**，以及**DI依赖注入**等源码分析

看到这里给大家整理了几个比较常见的面试来加深一下巩固：

BeanFactory和ApplicationContext的区别？

> BeanFactory是一个底层的IOC容器，而ApplicationContext是在其基础上增加了一些它的特性的同时同时增加了一些其他的整合特性比如：更好的整合SpringAOP、国际化消息、以及事务的发布、资源访问等这些新的特性

BeanFactory 与 FactoryBean的区别?

> BeanFactory 是 IoC 底层容器 ,FactoryBean 是 创建 Bean 的一种方式，帮助实现复杂的初始化逻辑

Spring IoC 容器的启动过程？

> 这个问题只要看懂了第一张流程图，以及最后的ApplicationContext.refresh()方法中的内部13个子方法，再回答这个问题应该问题不大，面试官应该会眼前一亮，ho,有点东西！！！