# 一个SpringMVC接口能返回JSON又能返回XML? 安排！

https://mp.weixin.qq.com/s/nyIEQ3CtlsjAFMPkv5Xqhw

## 原理

在HTTP协议里，当客户端发起一个HTTP请求时，可以携带一个请求头`Accept`来告诉服务端，客户端可以接受哪些响应类型（**MIME**），可以是一个也可以是多个。现在前后端分离普遍使用这种：

```
Accept:application/json
```

对于Spring MVC框架来说接受到对应的`Accept`会根据一定的策略找到对应的`HttpMessageConverter`来处理响应数据的格式。因此我们只需要找到一个动态指定`Accept`的方法就行了。

## 内容协商

听起来就很好理解，需要什么内容大家协商，共同解决问题。Spring MVC提供了一种被称作**内容协商**的机制，客户端在请求时声明需要的**MIME**类型，服务端只需要配置一些策略就是实现一个接口返回不同**MIME**类型的数据格式，想要JSON返回JSON,想要XML返回XML。

> Spring MVC版本基于**Spring MVC 5.3.9**。

### 服务端配置内容协商

内容协商的配置由Spring MVC中的`ContentNegotiationManager`负责，我们可以通过`ContentNegotiationConfigurer`配置它。

首先要在Spring MVC项目中加入Jackson的XML处理库：

```xml
        <dependency>
            <groupId>com.fasterxml.jackson.dataformat</groupId>
            <artifactId>jackson-dataformat-xml</artifactId>
        </dependency>
```

然后配置`WebMvcConfigurer`中的内容协商配置：

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
        configurer.favorParameter(true)
                // 客户端请求url需要携带一个query参数 默认名称是 format
                .parameterName("format")
                // 如果不声明 该query参数 返回的是json  如果你想指定默认返回类型就需要声明
                // .defaultContentType(MediaType.APPLICATION_XML)
                .mediaType("xml", MediaType.APPLICATION_XML)
                .mediaType("json", MediaType.APPLICATION_JSON);
    }
}
```

这样声明以后客户端请求接口中要携带一个query参数（参数名称默认为`format`，你可以修改它）来指定`MIME`的代号。根据上面的配置，如果你需要返回JSON:

```
https://yourapi?format=json
```

> ❝
>
> 你也可以不携带`format`参数，因为默认就是JSON，修改默认的MIME类型需要调用`defaultContentType`。

如果你需要返回XML:

```
https://yourapi?format=xml
```

服务端的接口也需要简单的改造：

```java
    @GetMapping(value = "/get",produces = {"application/json","application/xml"})
    public Map<String, String> doGet(@RequestParam String foo, String bar) {

        Map<String, String> map = new HashMap<>();
        map.put("foo", foo);
        map.put("bar", bar);
        return map;
    }
```

需要根据配置在`@RequestMapping`或其简化注解中声明对应的`produce`，这一点非常重要。这样我们改动的地方就非常的少了，能够适应更多的场景，而且维护起来也很简单。

### 其它策略

其实Spring MVC的内容协商还可以通过后缀扩展名实现，比如`/yourapi.json`或者`/yourapi.xml`。还有直接在客户端请求头中声明**MIME**类型。这些都不太方便操作所以就不介绍了，有兴趣可以去看官方文档。