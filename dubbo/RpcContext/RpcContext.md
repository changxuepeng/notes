https://www.iteye.com/blog/ncs123-2428255

### Dubbo之RpcContext详解

**一、RpcContext简介**
  RpcContext 是一个 ThreadLocal 的临时状态记录器，当接收到 RPC 请求，或发起 RPC 请求时，RpcContext 的状态都会变化。
  比如：A调B，B再调C，则B机器上，在B调C之前，RpcContext记录的是A调B的信息，在B调C之后，RpcContext记录的是B调C的信息。

**二、RpcContext的使用**
  **消费端**

Java代码 [![收藏代码](https://www.iteye.com/images/icon_star.png)](javascript:void())

```java
// 远程调用之前，通过attachment传KV给提供方  
RpcContext.getContext().setAttachment("userKey", "userValue");  
// 远程调用  
xxxService.xxx();  
// 本端是否为消费端，这里会返回true  
boolean isConsumerSide = RpcContext.getContext().isConsumerSide();  
// 获取最后一次调用的提供方IP地址  
String serverIP = RpcContext.getContext().getRemoteHost();  
// 获取当前服务配置信息，所有配置信息都将转换为URL的参数  
String application = RpcContext.getContext().getUrl().getParameter("application");  
// 注意：每发起RPC调用，上下文状态会变化  
yyyService.yyy();  
// 此时 RpcContext 的状态已变化   
RpcContext.getContext();    
```

  **服务端**

Java代码 [![收藏代码](https://www.iteye.com/images/icon_star.png)](javascript:void())