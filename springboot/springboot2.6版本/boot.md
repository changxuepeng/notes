# Spring Boot 2.6 正式发布：循环依赖默认禁止、增加SameSite属性...

https://mp.weixin.qq.com/s/M4V_si8LPqScrMcY5iZfZw

Spring官方正式发布了Spring Boot今年最后一个特性版本：**2.6.0**

同时，也宣布了**2.4.x**版本的终结。

那么这个新版本又带来了哪些新特性呢？下面就一起跟着DD来看看吧！

### 重要特性

#### 1. Servlet应用支持在 Cookie 中配置 SameSite 属性

该属性可通过server.session.cookie.same-site属性来配置，共有三个可选值：

- Strict 严格模式，必须同站请求才能发送 cookie
- Lax 宽松模式，安全的跨站请求可以发送 cookie
- None 禁止 SameSite 限制，必须配合 Secure 一起使用

![图片](boot.assets/640)

#### 2. 支持为主应用端口和管理端口配置健康组

这在 Kubernetes 等云服务环境中很有用。在这种环境下，出于安全目的，为执行器端点使用单独的管理端口是很常见的。拥有单独的端口可能会导致不可靠的健康检查，因为即使健康检查成功，主应用程序也可能无法正常工作。

以往传统的配置会将所有Actuator端点都放在一个单独的端口上，并将用于检测应用状态的健康组放在主端口的附加路径下。

#### 3. 增强/info端点，加入Java Runtime信息

增强后的例子：

```
{
  "java": {
    "vendor": "BellSoft",
    "version": "17",
    "runtime": {
      "name": "OpenJDK Runtime Environment",
      "version": "17+35-LTS"
    },
    "jvm": {
      "name": "OpenJDK 64-Bit Server VM",
      "vendor": "BellSoft",
      "version": "17+35-LTS"
    }
  }
}
```

该信息可以通过这个属性开启或关闭：

```
management.info.java.enabled=true
```

#### 4. 支持使用WebTestClient来测试Spring MVC

开发人员可以使用 WebTestClient 在模拟环境中测试 WebFlux 应用程序，或针对实时服务器测试任何 Spring Web 应用程序。 

这次增强后，开发者可以在Mock环境中使用 @AutoConfigureMockMvc 注释的类，就可以轻松注入 WebTestClient。 这样编写测试就比之前容易多了。

#### 5. 增加spring-rabbit-stream的自动化配置

这次更新添加了 Spring AMQP 的新 spring-rabbit-stream 模块的自动配置。 

当spring.rabbitmq.listener.type属性设置为stream时， StreamListenerContainer 是自动配置的。 

spring.rabbitmq.stream.*属性可用于配置对broker的访问，spring.rabbitmq.listener.stream.native-listener 可用于启用native listener

#### 6. 支持/env端点和configprops配置属性的自定义脱敏

虽然 Spring Boot 之前已经可以处理 /env 和 /configprops 端点中存在的敏感值，只需要可以通过配置属性来控制即可。但还有一种情况，用户可能希望根据属性源自哪个 PropertySource 来应用清理。

例如，Spring Cloud Vault 使用 Vault 来存储加密值并将它们加载到 Spring 环境中。由于所有值都是加密的，因此将整个属性源中的每个键的值脱敏是有意义的。可以通过添加类型为 SanitizingFunction 的 @Bean 来配置此类自定义脱敏规则。

### 其他变更

#### 1. Reactive Session 个性化 

当前版本可以动态配置 reactive session 的有效期

```
server.reactive.session.timeout=30
```

#### 2. Redis 链接自动配置链接池

当应用依赖中包含 commons-pool2.jar 会自动配置 redis 链接池 （Jedis Lettuce 都支持）。如果你想关闭则通过如下属性：

```
spring.redis.jedis.pool.enabled=false

spring.redis.lettuce.pool.enabled=false
```

#### 3. 构建信息个性化

- 通过 spring-boot-maven-plugin 支持自动生成此次构建信息的 **build-info.properties**

```
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <configuration>
           <excludeInfoProperties>
            <excludeInfoProperty>version</excludeInfoProperty>
         </excludeInfoProperties>
      </configuration>
    </plugin>
```

#### 4. Metrics新增指标

#### 应用启动的两个新指标：

```
application.started.time: 启动应用程序所需的时间

application.ready.time:  启动应用到对外提供服务所需时间
```

#### 磁盘空间的两个指标：

```
disk.free： 磁盘空闲空间

disk.total： 磁盘总空间
```

#### 5. Docker镜像的构建

增强docker-maven-plugin插件的功能：

- 为自定义镜像设置tags标签
- 网络配置参数，可用于Cloud Native Buildpacks的构建过程
- 支持使用 buildCache 和 launchCache 配置参数自定义用于缓存层的名称，这些层由构建包提供给构建的镜像

### 6. 移除 2.4 版本中的过期属性

由于2.4版本完成历史使命，因此有大量过期属性被移除，最近要升级的小伙伴一定要关注一下这部分内容，因为你原来的配置会失效！

关于Spring MVC 和 servlet 部分属性：

| 旧属性（已删除）            | 新属性                                 |
| :-------------------------- | :------------------------------------- |
| spring.web.locale           | spring.mvc.locale                      |
| spring.web.locale-resolver  | spring.mvc.locale-resolver             |
| spring.web.resources.*      | spring.resources.*                     |
| management.server.base-path | management.server.servlet.context-path |

关于Elasticsearch属性的变更：

![图片](boot.assets/640-16373716184742)

![图片](boot.assets/640-16373716204534)

因为内容较多，这里就不完全贴出来了，有兴趣的可以看看文末参考资料中的官方信息。

### 7. 默认情况完全禁止Bean的循环引用

还记得前几天我发布的这篇：[**为什么IDEA不推荐你使用@Autowired ？**](http://mp.weixin.qq.com/s?__biz=MzAxODcyNjEzNQ==&mid=2247545326&idx=1&sn=39b7c00bfb808051f0ad1b2566fc6ca3&chksm=9bd39c76aca41560e1abeb8da1965cdcefec139128c7c368144e9f8ea60398cfbce000223b6a&scene=21#wechat_redirect)

对于鼓励大家用构造器的方式，还受到了一些网友的嘲讽。那么在2.6.0之后，如果小伙伴依然觉得循环依赖无所谓，还坚持要用下面的这种模式：

![图片](boot.assets/640-16373716347586)

那么，你将收获下面这样的报错：

```
┌─────┐
|  a (field private com.example.demo.B com.example.demo.A.b)
↑     ↓
|  b (field private com.example.demo.A com.example.demo.B.a)
└─────┘


Action:

Relying upon circular references is discouraged and they are prohibited by default. Update your application to remove the dependency cycle between beans. As a last resort, it may be possible to break the cycle automatically by setting spring.main.allow-circular-references to true.
```

其实，Spring官方这样做，也是为了鼓励大家养成不要有循环依赖的好习惯。

但对于屎山项目，可能这样的要求对于开发者会很痛苦。所以，你也可以通过下面的配置，放开不允许循环依赖的要求：

```
spring.main.allow-circular-references=true
```

#### 8. SpringMVC 默认路径匹配策略

Spring MVC 处理程序映射匹配请求路径的默认策略已从 **AntPathMatcher** 更改为**PathPatternParser**。

Actuator端点现在也使用基于 PathPattern 的 URL 匹配。需要注意的是，Actuator端点的路径匹配策略无法通过配置属性进行配置。

如果需要将默认切换回 AntPathMatcher，可以将 spring.mvc.pathmatch.matching-strategy 设置为 ant-path-matcher，比如下面这样：

```
spring.mvc.pathmatch.matching-strategy=ant-path-matcher
```

好了，关于Spring Boot 2.6的版本解析到这里结束了。

最后，再推荐一下我一直在连载的免费教程：http://blog.didispace.com/spring-boot-learning-2x/。

跟很多其他教程不同。这个教程不光兼顾了1.x和2.x版本。同时，对于每次的更新，都会选择一些相关内容修补Tips，所以对各种不同阶段的读者长期都会有一些收获。如果你觉得不错，记得转发支持一下！

### 参考资料

- https://spring.io/blog/2021/11/19/spring-boot-2-6-is-now-available
- https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.6.0-Configuration-Changelog
- https://www.oschina.net/news/169783/spring-boot-2-6-0-released