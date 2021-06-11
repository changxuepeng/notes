# filter过滤ajax，重定向登录页面

默认ajax是不支持重定向的，因为ajax本身就是局部刷新，不重新加载页面的。

过滤ajax重定向的方法：

后台代码

```java

package cn.vsx.hamster.production.system;
 
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
 
@WebFilter(filterName = "LoginFilter",urlPatterns = "/*")
public class loginFilter implements Filter{
 
    private static Logger logger = LoggerFactory.getLogger(loginFilter.class);
 
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain){
        try {
            HttpServletRequest httpServletRequest = (HttpServletRequest)servletRequest;
            HttpServletResponse httpServletResponse = (HttpServletResponse)servletResponse;
            String url = httpServletRequest.getRequestURL().substring(httpServletRequest.getContextPath().length());
 
            logger.info("获取到url请求{}",url);
 
            if (url.contains("**")){
                filterChain.doFilter(httpServletRequest,httpServletResponse);
                return;
            }
 
            String[] filter = {"*","*","*","*","*"};
            for (String s : filter) {
                if (url.contains(s)){
                    filterChain.doFilter(httpServletRequest,httpServletResponse);
                    return;
                }
            }
 
            HttpSession httpSession = httpServletRequest.getSession();
            if (httpSession.getAttribute("adminCode") != null){
                logger.info("获取到session中的adminCode，放行");
                filterChain.doFilter(httpServletRequest,httpServletResponse);
            }else{
                logger.info("未获取到session中的adminCode，跳转登录页面");
                if (httpServletRequest.getHeader("x-requested-with") != null  && "XMLHttpRequest".equals(httpServletRequest.getHeader("x-requested-with"))){
                    httpServletResponse.setHeader("sessionstatus","timeout");
                    httpServletResponse.setStatus(403);
                    httpServletResponse.addHeader("loginPath",httpServletRequest.getScheme()+"://" + httpServletRequest.getServerName() + ":" + httpServletRequest.getServerPort() + "/production");
                    filterChain.doFilter(httpServletRequest,httpServletResponse);
                    logger.info("获取到ajax请求,跳转登录页面");
                    return;
                }
                httpServletResponse.sendRedirect(httpServletRequest.getScheme()+"://" + httpServletRequest.getServerName() + ":" + httpServletRequest.getServerPort() + "/production");
            }
        } catch (Exception e) {
           logger.error("拦截器报错:" ,e);
        }
    }
 
}
```

公共JS

```js
$.ajaxSetup( {
    //设置ajax请求结束后的执行动作
    complete :
        function(XMLHttpRequest, textStatus) {
            // 通过XMLHttpRequest取得响应头，sessionstatus
            var sessionstatus = XMLHttpRequest.getResponseHeader("sessionstatus");
            if (sessionstatus == "timeout") {
                var win = window;
                while (win != win.top){
                    win = win.top;
                }
                win.location.href= XMLHttpRequest.getResponseHeader("loginPath");
            }
        }
});
 
```

