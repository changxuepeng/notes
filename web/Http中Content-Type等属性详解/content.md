# Http中Content-Type等属性详解

https://blog.csdn.net/xiaoyao2246/article/details/83308505

### 正文

敬请关注博客，后期不断更新优质博文，谢谢

这里讲解Content-Type的可用值，以及在[spring](http://lib.csdn.net/base/javaee) MVC中如何使用它们来映射请求信息。

#### 1.Content-Type

 MediaType，即是Internet Media Type，互联网媒体类型；也叫做MIME类型，在Http协议消息头中，使用Content-Type来表示具体请求中的媒体类型信息。

例如： Content-Type: text/html;charset:utf-8;

常见的媒体格式类型如下：

> text/html ： HTML格式
> text/plain ：纯文本格式      
> text/xml ：  XML格式
> image/gif ：gif图片格式    
> image/jpeg ：jpg图片格式 
> image/png：png图片格式

以application开头的媒体格式类型：

>   application/xhtml+xml ：XHTML格式
>    application/xml     ： XML数据格式
>    application/atom+xml  ：Atom XML聚合格式    
>    application/json    ： JSON数据格式
>    application/pdf       ：pdf格式  
>    application/msword  ： Word文档格式
>    application/octet-stream ： 二进制流数据（如常见的文件下载）
>    application/x-www-form-urlencoded ： <form encType="">中默认的encType，form表单数据被编码为key/value格式发送到服务器（表单默认的提交数据的格式）

 另外一种常见的媒体格式是上传文件之时使用的：

>  multipart/form-data ： 需要在表单中进行文件上传时，就需要使用该格式

 以上就是我们在日常的开发中，经常会用到的若干content-type的内容格式。

#### 2.Spring MVC中关于关于Content-Type类型信息的使用

首先我们来看看RequestMapping中的Class定义：

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {
    String name() default "";

    @AliasFor("path")
    String[] value() default {};

    @AliasFor("value")
    String[] path() default {};

    RequestMethod[] method() default {};

    String[] params() default {};

    String[] headers() default {};

    String[] consumes() default {};

    String[] produces() default {};
}
```

> value:  指定请求的实际地址， 比如 /action/info之类。
> method：  指定请求的method类型， GET、POST、PUT、DELETE等
> consumes： 指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;
> produces:    指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回
> params： 指定request中必须包含某些参数值是，才让该方法处理
> headers： 指定request中必须包含某些指定的header值，才能让该方法处理请求
>
> 其中，consumes， produces使用content-typ信息进行过滤信息；headers中可以使用content-type进行过滤和判断。

#### 3. 使用示例

###### 3.1 headers

```java
@RequestMapping(value = "/test", method = RequestMethod.GET, headers="Referer=http://www.ifeng.com/")    
    public void testHeaders(@PathVariable String ownerId, @PathVariable String petId) {        
      // implementation omitted    
 }
```

 这里的Headers里面可以匹配所有Header里面可以出现的信息，不局限在Referer信息。

示例2

```java
  @RequestMapping(value = "/response/ContentType", headers = "Accept=application/json")    
    public void response2(HttpServletResponse response) throws IOException {    
        //表示响应的内容区数据的媒体类型为json格式，且编码为utf-8(客户端应该以utf-8解码)    
        response.setContentType("application/json;charset=utf-8");    
        //写出响应体内容    
        String jsonData = "{\"username\":\"zhang\", \"password\":\"123\"}";    
        response.getWriter().write(jsonData);    
    } 

```

服务器根据请求头“Accept=application/json”生产json数据。

当你有如下Accept头，将遵守如下规则进行应用：

> ①Accept：text/html,application/xml,application/json
>       将按照如下顺序进行produces的匹配 ①text/html ②application/xml ③application/json
> ②Accept：application/xml;q=0.5,application/json;q=0.9,text/html
>       将按照如下顺序进行produces的匹配 ①text/html ②application/json ③application/xml
>       参数为媒体类型的质量因子，越大则优先权越高(从0到1)
> ③Accept：*/*,text/*,text/html
>       将按照如下顺序进行produces的匹配 ①text/html ②text/* ③*/*

即匹配规则为：最明确的优先匹配。

## Requests部分

| Header              | 解释                                                         | 示例                                                    |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------- |
| Warning             | 关于消息实体的警告信息                                       | Warn: 199 Miscellaneous warning                         |
| Accept              | 指定客户端能够接收的内容类型                                 | Accept: text/plain, text/html                           |
| Accept-Charset      | 浏览器可以接受的字符编码集。                                 | Accept-Charset: iso-8859-5                              |
| Accept-Encoding     | 指定浏览器可以支持的web服务器返回内容压缩编码类型。          | Accept-Encoding: compress, gzip                         |
| Accept-Language     | 浏览器可接受的语言                                           | Accept-Language: en,zh                                  |
| Accept-Ranges       | 可以请求网页实体的一个或者多个子范围字段                     | Accept-Ranges: bytes                                    |
| Authorization       | HTTP授权的授权证书                                           | Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==       |
| Cache-Control       | 指定请求和响应遵循的缓存机制                                 | Cache-Control: no-cache                                 |
| Connection          | 表示是否需要持久连接。（HTTP 1.1默认进行持久连接）           | Connection: close                                       |
| Cookie              | HTTP请求发送时，会把保存在该请求域名下的所有cookie值一起发送给web服务器。 | Cookie: $Version=1; Skin=new;                           |
| Content-Length      | 请求的内容长度                                               | Content-Length: 348                                     |
| Content-Type        | 请求的与实体对应的MIME信息                                   | Content-Type: application/x-www-form-urlencoded         |
| Date                | 请求发送的日期和时间                                         | Date: Tue, 15 Nov 2010 08:12:31 GMT                     |
| Expect              | 请求的特定的服务器行为                                       | Expect: 100-continue                                    |
| From                | 发出请求的用户的Email                                        | From: user@email.com                                    |
| Host                | 指定请求的服务器的域名和端口号                               | Host: www.zcmhi.com                                     |
| If-Match            | 只有请求内容与实体相匹配才有效                               | If-Match: “737060cd8c284d8af7ad3082f209582d”            |
| If-Modified-Since   | 如果请求的部分在指定时间之后被修改则请求成功，未被修改则返回304代码 | If-Modified-Since: Sat, 29 Oct 2010 19:43:31 GMT        |
| If-None-Match       | 如果内容未改变返回304代码，参数为服务器先前发送的Etag，与服务器回应的Etag比较判断是否改变 | If-None-Match: “737060cd8c284d8af7ad3082f209582d”       |
| If-Range            | 如果实体未改变，服务器发送客户端丢失的部分，否则发送整个实体。参数也为Etag | If-Range: “737060cd8c284d8af7ad3082f209582d”            |
| If-Unmodified-Since | 只在实体在指定时间之后未被修改才请求成功                     | If-Unmodified-Since: Sat, 29 Oct 2010 19:43:31 GMT      |
| Max-Forwards        | 限制信息通过代理和网关传送的时间                             | Max-Forwards: 10                                        |
| Pragma              | 用来包含实现特定的指令                                       | Pragma: no-cache                                        |
| Proxy-Authorization | 连接到代理的授权证书                                         | Proxy-Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ== |
| Range               | 只请求实体的一部分，指定范围                                 | Range: bytes=500-999                                    |
| Referer             | 先前网页的地址，当前请求网页紧随其后,即来路                  | Referer: http://www.zcmhi.com/archives/71.html          |
| TE                  | 客户端愿意接受的传输编码，并通知服务器接受接受尾加头信息     | TE: trailers,deflate;q=0.5                              |
| Upgrade             | 向服务器指定某种传输协议以便服务器进行转换（如果支持）       | Upgrade: HTTP/2.0, SHTTP/1.3, IRC/6.9, RTA/x11          |
| User-Agent          | User-Agent的内容包含发出请求的用户信息                       | User-Agent: Mozilla/5.0 (Linux; X11)                    |
| Via                 | 通知中间网关或代理服务器地址，通信协议                       | Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)             |

## Responses 部分 

![在这里插入图片描述](content.assets/1)

![在这里插入图片描述](content.assets/2)

![在这里插入图片描述](content.assets/3)

3.2 params的示例

```java
  @RequestMapping(value = "/test/{userId}", method = RequestMethod.GET, params="myParam=myValue")    
    public void findUser(@PathVariable String userId) {        
      // implementation omitted    
    }   

```

仅处理请求中包含了名为“myParam”，值为“myValue”的请求，起到了一个过滤的作用。

3.3 consumes/produces

```java
   @Controller    
    @RequestMapping(value = "/users", method = RequestMethod.POST, consumes="application/json", produces="application/json")    
    @ResponseBody  
    public List<User> addUser(@RequestBody User userl) {        
        // implementation omitted    
        return List<User> users;  
    }    

```

方法仅处理request Content-Type为“application/json”类型的请求. produces标识==>处理request请求中Accept头中包含了"application/json"的请求，同时暗示了返回的内容类型为application/json;

### 总结

  在本文中，首先介绍了Content-Type主要支持的格式内容，然后基于@RequestMapping标注的内容介绍了主要的使用方法，其中,headers, consumes,produces,都是使用Content-Type中使用的各种媒体格式内容，可以基于这个格式内容来进行访问的控制和过滤。