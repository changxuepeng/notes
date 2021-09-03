# feign拦截器--RequestInterceptor

https://blog.csdn.net/wudiyong22/article/details/103801874

在使用feign做服务间调用的时候，如何修改请求的头部或编码信息呢，可以通过实现RequestInterceptor接口的apply方法，feign在发送请求之前都会调用该接口的apply方法，所以我们也可以通过实现该接口来记录请求发出去的时间点。

下是参考文章的内容

## RequestInterceptor

feign-core-10.2.3-sources.jar!/feign/RequestInterceptor.java

```java

public interface RequestInterceptor {
 
  /**
   * Called for every request. Add data using methods on the supplied {@link RequestTemplate}.
   */
  void apply(RequestTemplate template);
}
```

- RequestInterceptor接口定义了apply方法，其参数为RequestTemplate；它有一个抽象类为BaseRequestInterceptor，还有几个实现类分别为BasicAuthRequestInterceptor、FeignAcceptGzipEncodingInterceptor、FeignContentGzipEncodingInterceptor

## BasicAuthRequestInterceptor

feign-core-10.2.3-sources.jar!/feign/auth/BasicAuthRequestInterceptor.java

```java
public class BasicAuthRequestInterceptor implements RequestInterceptor {
 
  private final String headerValue;
 
  /**
   * Creates an interceptor that authenticates all requests with the specified username and password
   * encoded using ISO-8859-1.
   *
   * @param username the username to use for authentication
   * @param password the password to use for authentication
   */
  public BasicAuthRequestInterceptor(String username, String password) {
    this(username, password, ISO_8859_1);
  }
 
  /**
   * Creates an interceptor that authenticates all requests with the specified username and password
   * encoded using the specified charset.
   *
   * @param username the username to use for authentication
   * @param password the password to use for authentication
   * @param charset the charset to use when encoding the credentials
   */
  public BasicAuthRequestInterceptor(String username, String password, Charset charset) {
    checkNotNull(username, "username");
    checkNotNull(password, "password");
    this.headerValue = "Basic " + base64Encode((username + ":" + password).getBytes(charset));
  }
 
  /*
   * This uses a Sun internal method; if we ever encounter a case where this method is not
   * available, the appropriate response would be to pull the necessary portions of Guava's
   * BaseEncoding class into Util.
   */
  private static String base64Encode(byte[] bytes) {
    return Base64.encode(bytes);
  }
 
  @Override
  public void apply(RequestTemplate template) {
    template.header("Authorization", headerValue);
  }
}
```

- BasicAuthRequestInterceptor实现了RequestInterceptor接口，其apply方法往RequestTemplate添加名为Authorization的header

## BaseRequestInterceptor

spring-cloud-openfeign-core-2.2.0.M1-sources.jar!/org/springframework/cloud/openfeign/encoding/BaseRequestInterceptor.java

```java
public abstract class BaseRequestInterceptor implements RequestInterceptor {
 
    /**
     * The encoding properties.
     */
    private final FeignClientEncodingProperties properties;
 
    /**
     * Creates new instance of {@link BaseRequestInterceptor}.
     * @param properties the encoding properties
     */
    protected BaseRequestInterceptor(FeignClientEncodingProperties properties) {
        Assert.notNull(properties, "Properties can not be null");
        this.properties = properties;
    }
 
    /**
     * Adds the header if it wasn't yet specified.
     * @param requestTemplate the request
     * @param name the header name
     * @param values the header values
     */
    protected void addHeader(RequestTemplate requestTemplate, String name,
            String... values) {
 
        if (!requestTemplate.headers().containsKey(name)) {
            requestTemplate.header(name, values);
        }
    }
 
    protected FeignClientEncodingProperties getProperties() {
        return this.properties;
    }
}
```

- BaseRequestInterceptor定义了addHeader方法，往requestTemplate添加非重名的header

### FeignAcceptGzipEncodingInterceptor

spring-cloud-openfeign-core-2.2.0.M1-sources.jar!/org/springframework/cloud/openfeign/encoding/FeignAcceptGzipEncodingInterceptor.java

```java

public class FeignAcceptGzipEncodingInterceptor extends BaseRequestInterceptor {
 
    /**
     * Creates new instance of {@link FeignAcceptGzipEncodingInterceptor}.
     * @param properties the encoding properties
     */
    protected FeignAcceptGzipEncodingInterceptor(
            FeignClientEncodingProperties properties) {
        super(properties);
    }
 
    /**
     * {@inheritDoc}
     */
    @Override
    public void apply(RequestTemplate template) {
 
        addHeader(template, HttpEncoding.ACCEPT_ENCODING_HEADER,
                HttpEncoding.GZIP_ENCODING, HttpEncoding.DEFLATE_ENCODING);
    }
 
}
```

- FeignAcceptGzipEncodingInterceptor继承了BaseRequestInterceptor，它的apply方法往RequestTemplate添加了名为Accept-Encoding，值为gzip,deflate的header

### FeignContentGzipEncodingInterceptor

spring-cloud-openfeign-core-2.2.0.M1-sources.jar!/org/springframework/cloud/openfeign/encoding/FeignContentGzipEncodingInterceptor.java

```java
public class FeignContentGzipEncodingInterceptor extends BaseRequestInterceptor {
 
    /**
     * Creates new instance of {@link FeignContentGzipEncodingInterceptor}.
     * @param properties the encoding properties
     */
    protected FeignContentGzipEncodingInterceptor(
            FeignClientEncodingProperties properties) {
        super(properties);
    }
 
    /**
     * {@inheritDoc}
     */
    @Override
    public void apply(RequestTemplate template) {
 
        if (requiresCompression(template)) {
            addHeader(template, HttpEncoding.CONTENT_ENCODING_HEADER,
                    HttpEncoding.GZIP_ENCODING, HttpEncoding.DEFLATE_ENCODING);
        }
    }
 
    /**
     * Returns whether the request requires GZIP compression.
     * @param template the request template
     * @return true if request requires compression, false otherwise
     */
    private boolean requiresCompression(RequestTemplate template) {
 
        final Map<String, Collection<String>> headers = template.headers();
        return matchesMimeType(headers.get(HttpEncoding.CONTENT_TYPE))
                && contentLengthExceedThreshold(headers.get(HttpEncoding.CONTENT_LENGTH));
    }
 
    /**
     * Returns whether the request content length exceed configured minimum size.
     * @param contentLength the content length header value
     * @return true if length is grater than minimum size, false otherwise
     */
    private boolean contentLengthExceedThreshold(Collection<String> contentLength) {
 
        try {
            if (contentLength == null || contentLength.size() != 1) {
                return false;
            }
 
            final String strLen = contentLength.iterator().next();
            final long length = Long.parseLong(strLen);
            return length > getProperties().getMinRequestSize();
        }
        catch (NumberFormatException ex) {
            return false;
        }
    }
 
    /**
     * Returns whether the content mime types matches the configures mime types.
     * @param contentTypes the content types
     * @return true if any specified content type matches the request content types
     */
    private boolean matchesMimeType(Collection<String> contentTypes) {
        if (contentTypes == null || contentTypes.size() == 0) {
            return false;
        }
 
        if (getProperties().getMimeTypes() == null
                || getProperties().getMimeTypes().length == 0) {
            // no specific mime types has been set - matching everything
            return true;
        }
 
        for (String mimeType : getProperties().getMimeTypes()) {
            if (contentTypes.contains(mimeType)) {
                return true;
            }
        }
 
        return false;
    }
 
}
```

- FeignContentGzipEncodingInterceptor继承了BaseRequestInterceptor，其apply方法先判断是否需要compression，即mimeType是否符合要求以及content大小是否超出阈值，需要compress的话则添加名为Content-Encoding，值为gzip,deflate的header

## 小结

- RequestInterceptor接口定义了apply方法，其参数为RequestTemplate；它有一个抽象类为BaseRequestInterceptor，还有几个实现类分别为BasicAuthRequestInterceptor、FeignAcceptGzipEncodingInterceptor、FeignContentGzipEncodingInterceptor
- BasicAuthRequestInterceptor实现了RequestInterceptor接口，其apply方法往RequestTemplate添加名为Authorization的header
- BaseRequestInterceptor定义了addHeader方法，往requestTemplate添加非重名的header；FeignAcceptGzipEncodingInterceptor继承了BaseRequestInterceptor，它的apply方法往RequestTemplate添加了名为Accept-Encoding，值为gzip,deflate的header；FeignContentGzipEncodingInterceptor继承了BaseRequestInterceptor，其apply方法先判断是否需要compression，即mimeType是否符合要求以及content大小是否超出阈值，需要compress的话则添加名为Content-Encoding，值为gzip,deflate的header

**值得注意的是：该拦截器所在的线程不是controller层接收到请求的线程，因为feign是利用多个线程池来发送请求的，所以，主线程中的ThreadLocal对象在这里失效了，解决方案如下（参考资料：**https://bbs.huaweicloud.com/blogs/106521**）：**
https://stackoverflow.com/questions/34719809/unreachable-security-context-using-feign-requestinterceptor 里面提到了使用 HystrixRequestVariableDefault，通过该类的注释可以知道，这个是Hystrix 为自身执行的线程提供的一个类似于ThreadLocal 的类，但是它与ThreadLocal 的不同之处在于，该Locals 的作用范围提升到了这个User Request Scope 级别，通过其注解可知，它是通过父类线程与子类线程共享的方式来共用该Locals 中的信息；所以，达到了User Request 线程与Hystrix 线程共用 attributes 的目的；由此，可以猜测，Hystrix 的线程是由当前Request 线程所创建的子线程；不过，使用的过程中，需要注意的是，HystrixRequestVariable 必须在每一个请求开始的时候进行初始化；也就是说，我们可以将 Request Context 中的有用信息存储到HystrixRequestVariableDefault中达到与Hystrix Context 共享信息；也就实现了Request Context 中的属性与Hystrix Context 之间共享的目的。

总的来说就是用HystrixRequestVariableDefault代替ThreadLocal来作为同一个请求不同环节传递变量，ThreadLocal作用于同一个线程，HystrixRequestVariableDefault作用于同一个请求。

关于HystrixRequestVariableDefault的更多说明，可以直接看该类源码注释。

具体做法看：https://bbs.huaweicloud.com/blogs/106521