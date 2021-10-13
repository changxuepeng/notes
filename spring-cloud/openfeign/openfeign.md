# openFeign夺命连环9问

https://mp.weixin.qq.com/s/YotFLqsDjuPSwgjDU7suMQ

## 1、前言

前面介绍了Spring Cloud 中的灵魂摆渡者`Nacos`，和它的前辈们相比不仅仅功能强大，而且部署非常简单。

今天介绍一款服务调用的组件：`OpenFeign`，同样是一款超越先辈（`Ribbon`、`Feign`）的狠角色。

文章目录如下：

![图片](openfeign.assets/640)

## 2、Feign是什么？

Feign也是一个狠角色，Feign旨在使得Java Http客户端变得更容易。

Feign集成了Ribbon、RestTemplate实现了负载均衡的执行Http调用，只不过对原有的方式（Ribbon+RestTemplate）进行了封装，开发者不必手动使用RestTemplate调服务，而是定义一个接口，在这个接口中标注一个注解即可完成服务调用，这样更加符合面向接口编程的宗旨，简化了开发。

![图片](openfeign.assets/640-16340872747452)

但遗憾的是Feign现在停止迭代了，当然现在也是有不少企业在用。

有想要学习Feign的读者可以上spring Cloud官网学习，陈某这里也不再详细介绍了，不是今天的重点。

## 3、openFeign是什么？

前面介绍过停止迭代的Feign，简单点来说：OpenFeign是springcloud在Feign的基础上支持了SpringMVC的注解，如`@RequestMapping`等等。OpenFeign的`@FeignClient`可以解析SpringMVC的`@RequestMapping`注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。

> 官网地址：https://docs.spring.io/spring-cloud-openfeign/docs/2.2.10.BUILD-SNAPSHOT/reference/html

## 4、Feign和openFeign有什么区别？

| Feign                                                        | openFiegn                                                    |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Feign是SpringCloud组件中一个轻量级RESTful的HTTP服务客户端，Feign内置了Ribbon，用来做客户端负载均衡，去调用服务注册中心的服务。Feign的使用方式是：使用Feign的注解定义接口，调用这个接口，就可以调用服务注册中心的服务 | OpenFeign 是SpringCloud在Feign的基础上支持了SpringMVC的注解，如@RequestMapping等。OpenFeign 的@FeignClient可以解析SpringMVC的@RequestMapping注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。 |

## 5、环境准备

本篇文章Spring Cloud版本、JDK环境、项目环境均和上一篇Nacos的环境相同：[五十五张图告诉你微服务的灵魂摆渡者Nacos究竟有多强？](https://mp.weixin.qq.com/s?__biz=MzU3MDAzNDg1MA==&mid=2247493854&idx=1&sn=4b3fb7f7e17a76000733899f511ef915&scene=21#wechat_redirect)。

注册中心就不再使用`Eureka`了，直接使用`Nacos`作为注册和配置中心，有不会的可以查看Nacos文章。

本篇文章搭建的项目结构如下图：

![图片](openfeign.assets/640-16340873312464)

> 注册中心使用**Nacos**，创建个微服务，分别为服务提供者**Produce**，服务消费者**Consumer**。

## 6、创建服务提供者

既然是微服务之间的相互调用，那么一定会有服务提供者了，创建`openFeign-provider9005`，注册进入Nacos中，配置如下：

```yaml
server:
  port: 9005
spring:
  application:
    ## 指定服务名称，在nacos中的名字
    name: openFeign-provider
  cloud:
    nacos:
      discovery:
        # nacos的服务地址，nacos-server中IP地址:端口号
        server-addr: 127.0.0.1:8848
management:
  endpoints:
    web:
      exposure:
        ## yml文件中存在特殊字符，必须用单引号包含，否则启动报错
        include: '*'
```

**注意**：此处的`spring.application.name`指定的名称将会在openFeign接口调用中使用。

> 项目源码都会上传，关于如何注册进入Nacos，添加什么依赖源码都会有，结合陈某上篇Nacos文章，这都不是难事！

## 7、创建服务消费者

新建一个模块`openFeign-consumer9006`作为消费者服务，步骤如下。

### 1、添加依赖

除了Nacos的注册中心的依赖，还要添加openFeign的依赖，如下：

```
<dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

### 2、添加注解@EnableFeignClients开启openFeign功能

老套路了，在Spring boot 主启动类上添加一个注解`@EnableFeignClients`，开启openFeign功能，如下：

```
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class OpenFeignConsumer9006Application
{
    public static void main(String[] args) {
        SpringApplication.run(OpenFeignConsumer9006Application.class, args);
    }
}
```

### 3、新建openFeign接口

新建一个openFeign接口，使用`@FeignClient`注解标注，如下：

```
@FeignClient(value = "openFeign-provider")
public interface OpenFeignService {
}
```

> **注意**：该注解`@FeignClient`中的`value`属性指定了服务提供者在nacos注册中心的**服务名**。

### 4、新建一个Controller调试

新建一个controller用来调试接口，直接调用openFeign的接口，如下：

```
@RestController
@RequestMapping("/openfeign")
public class OpenFeignController {
    
}
```

好了，至此一个openFeign的微服务就搭建好了，并未实现具体的功能，下面一点点实现。

## 8、openFeign如何传参？

开发中接口传参的方式有很多，但是在openFeign中的传参是有一定规则的，下面详细介绍。

### 1、传递JSON数据

这个也是接口开发中常用的传参规则，在Spring Boot 中通过`@RequestBody`标识入参。

provider接口中JSON传参方法如下：

```
@RestController
@RequestMapping("/openfeign/provider")
public class OpenFeignProviderController {
    @PostMapping("/order2")
    public Order createOrder2(@RequestBody Order order){
        return order;
    }
}
```

consumer中openFeign接口中传参代码如下：

```
@FeignClient(value = "openFeign-provider")
public interface OpenFeignService {
    /**
     * 参数默认是@RequestBody标注的，这里的@RequestBody可以不填
     * 方法名称任意
     */
    @PostMapping("/openfeign/provider/order2")
    Order createOrder2(@RequestBody Order order);
}
```

注意：`openFeign`默认的传参方式就是JSON传参（`@RequestBody`），因此定义接口的时候可以不用`@RequestBody`注解标注，不过为了规范，一般都填上。

### 2、POJO表单传参

这种传参方式也是比较常用，参数使用POJO对象接收。

provider服务提供者代码如下：

```
@RestController
@RequestMapping("/openfeign/provider")
public class OpenFeignProviderController {
    @PostMapping("/order1")
    public Order createOrder1(Order order){
        return order;
    }
}
```

consumer消费者openFeign代码如下：

```
@FeignClient(value = "openFeign-provider")
public interface OpenFeignService {
    /**
     * 参数默认是@RequestBody标注的，如果通过POJO表单传参的，使用@SpringQueryMap标注
     */
    @PostMapping("/openfeign/provider/order1")
    Order createOrder1(@SpringQueryMap Order order);
}
```

网上很多人疑惑POJO表单方式如何传参，官方文档明确给出了解决方案，如下：

![图片](openfeign.assets/640-16340874480356)

openFeign提供了一个注解`@SpringQueryMap`完美解决POJO表单传参。

### 3、URL中携带参数

此种方式针对restful方式中的GET请求，也是比较常用请求方式。

provider服务提供者代码如下：

```
@RestController
@RequestMapping("/openfeign/provider")
public class OpenFeignProviderController {

    @GetMapping("/test/{id}")
    public String test(@PathVariable("id")Integer id){
        return "accept one msg id="+id;
}
```

consumer消费者openFeign接口如下：

```
@FeignClient(value = "openFeign-provider")
public interface OpenFeignService {

    @GetMapping("/openfeign/provider/test/{id}")
    String get(@PathVariable("id")Integer id);
}
```

使用注解`@PathVariable`接收url中的占位符，这种方式很好理解。

### 4、普通表单参数

此种方式传参不建议使用，但是也有很多开发在用。

provider服务提供者代码如下：

```
@RestController
@RequestMapping("/openfeign/provider")
public class OpenFeignProviderController {
    @PostMapping("/test2")
    public String test2(String id,String name){
        return MessageFormat.format("accept on msg id={0}，name={1}",id,name);
    }
}
```

consumer消费者openFeign接口传参如下：

```
@FeignClient(value = "openFeign-provider")
public interface OpenFeignService {
    /**
     * 必须要@RequestParam注解标注，且value属性必须填上参数名
     * 方法参数名可以任意，但是@RequestParam注解中的value属性必须和provider中的参数名相同
     */
    @PostMapping("/openfeign/provider/test2")
    String test(@RequestParam("id") String arg1,@RequestParam("name") String arg2);
}
```

### 5、总结

传参的方式有很多，比如文件传参.....陈某这里只是列举了四种常见得传参方式。

## 9、超时如何处理？

想要理解超时处理，先看一个例子：我将provider服务接口睡眠3秒钟，接口如下：

```
@PostMapping("/test2")
public String test2(String id,String name) throws InterruptedException {
        Thread.sleep(3000);
        return MessageFormat.format("accept on msg id={0}，name={1}",id,name);
}
```

此时，我们调用consumer的openFeign接口返回结果如下图：

![图片](openfeign.assets/640-16340874750508)

很明显的看出程序异常了，返回了接口调用超时。what？why？...........

openFeign其实是有默认的超时时间的，默认分别是连接超时时间`10秒`、读超时时间`60秒`，源码在`feign.Request.Options#Options()`这个方法中，如下图：

![图片](openfeign.assets/640-163408749427310)

那么问题来了：**为什么我只设置了睡眠3秒就报超时呢？**

其实openFeign集成了Ribbon，Ribbon的默认超时连接时间、读超时时间都是是1秒，源码在`org.springframework.cloud.openfeign.ribbon.FeignLoadBalancer#execute()`方法中，如下图：

![图片](openfeign.assets/640-163408749607212)

**源码大致意思**：如果openFeign没有设置对应得超时时间，那么将会采用Ribbon的默认超时时间。

理解了超时设置的原理，由之产生两种方案也是很明了了，如下：

- 设置openFeign的超时时间
- 设置Ribbon的超时时间

### 1、设置Ribbon的超时时间（不推荐）

设置很简单，在配置文件中添加如下设置：

```
ribbon:
  # 值的是建立链接所用的时间，适用于网络状况正常的情况下， 两端链接所用的时间
  ReadTimeout: 5000
  # 指的是建立链接后从服务器读取可用资源所用的时间
  ConectTimeout: 5000
```

### 2、设置openFeign的超时时间（推荐）

openFeign设置超时时间非常简单，只需要在配置文件中配置，如下：

```
feign:
  client:
    config:
      ## default 设置的全局超时时间，指定服务名称可以设置单个服务的超时时间
      default:
        connectTimeout: 5000
        readTimeout: 5000
```

> default设置的是全局超时时间，对所有的openFeign接口服务都生效

但是正常的业务逻辑中可能涉及到多个openFeign接口的调用，如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/19cc2hfD2rCXDTpOZ5roJBuKpxeWlRAcqWa2OxorzhrUyLsw9NDYkGDeCiaR777KjxfqzrQroqLKNFibIiaGvqkIg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

上图中的伪代码如下：

```
public T invoke(){
    //1. 调用serviceA
    serviceA();
    
    //2. 调用serviceB
    serviceB();
    
    //3. 调用serviceC
    serviceC();
}
```

那么上面配置的全局超时时间能不能通过呢？很显然是`serviceA`、`serviceB`能够成功调用，但是`serviceC`并不能成功执行，肯定报超时。

此时我们可以给`serviceC`这个服务单独配置一个超时时间，配置如下：

```yaml
feign:
  client:
    config:
      ## default 设置的全局超时时间，指定服务名称可以设置单个服务的超时时间
      default:
        connectTimeout: 5000
        readTimeout: 5000
      ## 为serviceC这个服务单独配置超时时间
      serviceC:
        connectTimeout: 30000
        readTimeout: 30000
```

> **注意**：单个配置的超时时间将会覆盖全局配置。

## 10、如何开启日志增强？

openFeign虽然提供了日志增强功能，但是默认是不显示任何日志的，不过开发者在调试阶段可以自己配置日志的级别。

openFeign的日志级别如下：

- **NONE**：默认的，不显示任何日志;
- **BASIC**：仅记录请求方法、URL、响应状态码及执行时间;
- **HEADERS**：除了BASIC中定义的信息之外，还有请求和响应的头信息;
- **FULL**：除了HEADERS中定义的信息之外，还有请求和响应的正文及元数据。

配置起来也很简单，步骤如下：

### 1、配置类中配置日志级别

需要自定义一个配置类，在其中设置日志级别，如下：

![图片](openfeign.assets/640-163408763212814)

> **注意**：这里的logger是feign包里的。

### 2、yaml文件中设置接口日志级别

只需要在配置文件中调整指定包或者openFeign的接口日志级别，如下：

```
logging:
  level:
    cn.myjszl.service: debug
```

这里的`cn.myjszl.service`是openFeign接口所在的包名，当然你也可以配置一个特定的openFeign接口。

### 3、演示效果

上述步骤将日志设置成了`FULL`，此时发出请求，日志效果如下图：

![图片](openfeign.assets/640-163408767280016)

日志中详细的打印出了请求头、请求体的内容。

## 11、如何替换默认的httpclient？

Feign在默认情况下使用的是JDK原生的**URLConnection**发送HTTP请求，没有连接池，但是对每个地址会保持一个长连接，即利用HTTP的persistence connection。

在生产环境中，通常不使用默认的http client，通常有如下两种选择：

- 使用**ApacheHttpClient**
- 使用**OkHttp**

至于哪个更好，其实各有千秋，我比较倾向于ApacheHttpClient，毕竟老牌子了，稳定性不在话下。

那么如何替换掉呢？其实很简单，下面演示使用ApacheHttpClient替换。

### 1、添加ApacheHttpClient依赖

在openFeign接口服务的pom文件添加如下依赖：

```xml
<!--     使用Apache HttpClient替换Feign原生httpclient-->
    <dependency>
      <groupId>org.apache.httpcomponents</groupId>
      <artifactId>httpclient</artifactId>
    </dependency>
    
    <dependency>
      <groupId>io.github.openfeign</groupId>
      <artifactId>feign-httpclient</artifactId>
    </dependency>
```

为什么要添加上面的依赖呢？从源码中不难看出，请看`org.springframework.cloud.openfeign.FeignAutoConfiguration.HttpClientFeignConfiguration`这个类，代码如下：

![图片](openfeign.assets/640-163408769440118)

上述红色框中的生成条件，其中的`@ConditionalOnClass(ApacheHttpClient.class)`，必须要有`ApacheHttpClient`这个类才会生效，并且`feign.httpclient.enabled`这个配置要设置为`true`。

### 2、配置文件中开启

在配置文件中要配置开启，代码如下：

```
feign:
  client:
    httpclient:
      # 开启 Http Client
      enabled: true
```

### 3、如何验证已经替换成功？

其实很简单，在`feign.SynchronousMethodHandler#executeAndDecode()`这个方法中可以清楚的看出调用哪个client，如下图：

![图片](openfeign.assets/640-163408770895020)

上图中可以看到最终调用的是`ApacheHttpClient`。

### 4、总结

上述步骤仅仅演示一种替换方案，剩下的一种不再演示了，原理相同。

## 12、如何通讯优化？

在讲如何优化之前先来看一下**GZIP** 压缩算法，概念如下：

> gzip是一种数据格式，采用用deflate算法压缩数据；gzip是一种流行的数据压缩算法，应用十分广泛，尤其是在Linux平台。

**当GZIP压缩到一个纯文本数据时，效果是非常明显的，大约可以减少70％以上的数据大小。**

网络数据经过压缩后实际上降低了网络传输的字节数，最明显的好处就是可以加快网页加载的速度。网页加载速度加快的好处不言而喻，除了节省流量，改善用户的浏览体验外，另一个潜在的好处是GZIP与搜索引擎的抓取工具有着更好的关系。例如 Google就可以通过直接读取GZIP文件来比普通手工抓取更快地检索网页。

GZIP压缩传输的原理如下图：

![图片](openfeign.assets/640-163408772626622)

按照上图拆解出的步骤如下：

- 客户端向服务器请求头中带有：`Accept-Encoding:gzip,deflate` 字段，向服务器表示，客户端支持的压缩格式（gzip或者deflate)，如果不发送该消息头，服务器是不会压缩的。
- 服务端在收到请求之后，如果发现请求头中含有`Accept-Encoding`字段，并且支持该类型的压缩，就对响应报文压缩之后返回给客户端，并且携带`Content-Encoding:gzip`消息头，表示响应报文是根据该格式压缩过的。
- 客户端接收到响应之后，先判断是否有Content-Encoding消息头，如果有，按该格式解压报文。否则按正常报文处理。

openFeign支持**请求/响应**开启GZIP压缩，整体的流程如下图：

![图片](openfeign.assets/640-163408772884124)

上图中涉及到GZIP传输的只有两块，分别是**Application client -> Application Service**、 **Application Service->Application client**。

**注意**：openFeign支持的GZIP仅仅是在openFeign接口的请求和响应，即是openFeign消费者调用服务提供者的接口。

openFeign开启GZIP步骤也是很简单，只需要在配置文件中开启如下配置：

```
feign:
  ## 开启压缩
  compression:
    request:
      enabled: true
      ## 开启压缩的阈值，单位字节，默认2048，即是2k，这里为了演示效果设置成10字节
      min-request-size: 10
      mime-types: text/xml,application/xml,application/json
    response:
      enabled: true
```

上述配置完成之后，发出请求，可以清楚看到请求头中已经携带了GZIP压缩，如下图：

![图片](openfeign.assets/640-163408773801926)

## 13、如何熔断降级？

常见的熔断降级框架有`Hystrix`、`Sentinel`，openFeign默认支持的就是`Hystrix`，这个在官方文档上就有体现，毕竟是一奶同胞嘛，哈哈...........

但是阿里的Sentinel无论是功能特性、简单易上手等各方面都完全秒杀Hystrix，因此此章节就使用**openFeign+Sentinel**进行整合实现服务降级。

> **说明**：此处并不着重介绍Sentinel，陈某打算放在下一篇文章详细介绍Sentinel的强大之处。

### 1、添加Sentinel依赖

在`openFeign-consumer9006`消费者的pom文件添加sentinel依赖（由于使用了聚合模块，不指定版本号），如下：

```
<dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

### 2、配置文件中开启sentinel熔断降级

要想openFeign使用sentinel的降级功能，还需要在配置文件中开启，添加如下配置：

```
feign:
  sentinel:
    enabled: true
```

### 3、添加降级回调类

这个类一定要和openFeign接口实现同一个类，如下图：

![图片](openfeign.assets/640-163408776813128)

`OpenFeignFallbackService`这个是降级回调的类，一旦`OpenFeignService`中对应得接口出现了异常则会调用这个类中对应得方法进行降级处理。

### 4、添加fallback属性

在`@FeignClient`中添加`fallback`属性，属性值是降级回调的类，如下：

```java
@FeignClient(value = "openFeign-provider",fallback = OpenFeignFallbackService.class)
public interface OpenFeignService {}
```

### 5、演示

经过如上4个步骤，openFeign的熔断降级已经设置完成了，此时演示下效果。

通过postman调用`http://localhost:9006/openfeign/order3`这个接口，正常逻辑返回如下图：

![图片](openfeign.assets/640-163408784802430)

现在手动造个异常，在服务提供的接口中抛出异常，如下图：

![图片](openfeign.assets/640-163408785014532)

此时重新调用`http://localhost:9006/openfeign/order3`，返回如下图：

![图片](openfeign.assets/640-163408786353734)

哦豁，可以很清楚的看到服务已经成功降级调用，哦了，功能完成。

> **注意**：实际开发中返回结果应该根据架构统一定制，陈某这里只是为了演示方便，不要借鉴，哈哈。。。