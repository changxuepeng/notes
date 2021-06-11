# UserAgentUtils

```xml
<dependency>
<groupId>eu.bitwalker</groupId>
<artifactId>UserAgentUtils</artifactId>
<version>1.20</version>
</dependency>
```

```java
rivate SearchLog initSearchLog(HttpServletRequest request) {

// userAgent中有很多获取请求信息的方法
UserAgent userAgent = UserAgent.parseUserAgentString(request.getHeader("User-Agent"));
SearchLog searchLog = new SearchLog();

searchLog.setSearchExpression(getParam("searchExpression", request));
searchLog.setUserId(SessionUtil.getUserId());
searchLog.setUsername(SessionUtil.getCurrentUserName());
// searchLog.setSearchTime(new DateTime().toString("yyyy-MM-dd HH:mm:ss"));
searchLog.setSearchTime(new Date());
    // 获取客户端请求的浏览器类型
searchLog.setBrowserType(userAgent.getBrowser().toString());

String s = request.getParameter("searchChannel");
searchLog.setSearchChannel(request.getParameter("searchChannel"));

// 获得终端设备的IP地址 
searchLog.setTerminalIp(request.getRemoteAddr());

return searchLog;
}
```

