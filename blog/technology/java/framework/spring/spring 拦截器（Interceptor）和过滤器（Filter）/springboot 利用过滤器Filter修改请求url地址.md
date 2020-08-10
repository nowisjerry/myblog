# springboot 利用过滤器Filter修改请求url地址

要求：

代码中配置的url路径为http://127.0.0.1/api/associates/queryAssociatesInfo

现在要求http://127.0.0.1/associates/queryAssociatesInfo也可以同样访问同一个conroller下面的method,并且要求参数全部跟随

代码：

```java
package com.shitou.huishi.framework.filter;

import java.io.IOException;
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpServletResponseWrapper;

/**
 * 修改请求路由，当进入url为/a/b时，将其url修改为/api/a/b
 * Created by qhong on 2018/5/16 13:27
 **/
public class UrlFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }

    @Override
    public void destroy() {
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest)request;
        HttpServletResponseWrapper httpResponse = new HttpServletResponseWrapper((HttpServletResponse) response);
        System.out.println(httpRequest.getRequestURI());
        String path=httpRequest.getRequestURI();
        if(path.indexOf("/api/")<0){
            path="/api"+path;
            System.out.println(path);
            httpRequest.getRequestDispatcher(path).forward(request,response);
        }
       else {
            chain.doFilter(request,response);

        }
        return;
    }
}
```

这个类必须继承Filter类，这个是Servlet的规范。有了过滤器类以后，以前的web项目可以在web.xml中进行配置，但是spring boot项目并没有web.xml这个文件，那怎么配置？在Spring boot中，我们需要FilterRegistrationBean来完成配置。

其实现过程如下：

```java
package com.shitou.huishi.framework.filter;

import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Created by qhong on 2018/5/16 15:28
 **/
@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean registFilter() {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setFilter(new UrlFilter());
        registration.addUrlPatterns("/*");
        registration.setName("UrlFilter");
        registration.setOrder(1);
        return registration;
    }

}
```



 

 

https://www.cnblogs.com/paddix/p/8365558.html

作者：hongda

出处：https://www.cnblogs.com/hongdada/p/9046376.html

版权：本文采用「[署名 4.0 国际](https://creativecommons.org/licenses/by/4.0)」知识共享许可协议进行许可。
