---
title: JAVA服务器端配置跨域请求配置类
date: 2023-03-29 16:24:32
tags: 后端配置跨域
description: 使用java后端来解决cors的跨域问题
keywords: 跨域,JAVA,cors,cors跨域
cover: https://img.mp.itc.cn/upload/20160824/050119e8f37a4d5aa52a4571053f4027_th.jpg
---

## 方法一：添加cors全局配置(推荐)
**com/wwk/seckill/config/CorsConfig.java**
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

/**
 * 全局跨域配置
 */
@Configuration
public class CorsConfig {

    @Bean
    public CorsFilter corsFilter() {
        //创建跨域配置，增加CORS配置信息。
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        //跨域请求默认不包含cookie，设置为true，可以包含cookie。
        corsConfiguration.setAllowCredentials(true);
        //支持哪些来源的请求跨越,支持
        corsConfiguration.addAllowedOrigin("*");
        //支持哪些头信息
        corsConfiguration.addAllowedHeader("*");
        //支持哪些方法跨越
        corsConfiguration.addAllowedMethod("*");
        //增加映射路径
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        // /**表示所有请求都支持跨域
        source.registerCorsConfiguration("/**", corsConfiguration);

        //返回新的CorsFilter
        return new CorsFilter(source);
    }

}

```

## 方法二：添加cors配置类
**WebMvcConfig**
```java

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;

/**
 * WebMvcConfig：配置类继承WebMvcConfigurationSupport
 * 重写addCorsMappings 配置实现跨域
 */
@Configuration
public class WebMvcConfig extends WebMvcConfigurationSupport {
    @Override
    protected void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("POST","GET","PUT","OPTIONS","DELETE")
                .maxAge(3600)
                .allowCredentials(true);
    }
}



```

`具体来说，它允许所有来源（allowedOrigins("*")）的POST、GET、PUT、OPTIONS和DELETE方法（allowedMethods("POST","GET","PUT","OPTIONS","DELETE")），并允许凭据（allowCredentials(true)）的访问，设置了最大缓存时间为3600秒（maxAge(3600)）。`


## 方法三：使用Filter方法实现跨域
1. Filter类配置跨域拦截
```java

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.xml.crypto.dsig.spec.XPathType;
import java.io.IOException;

//对映射路径配置，所有
@WebFilter(urlPatterns = "*")
public class CorsFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        Filter.super.init(filterConfig);
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletResponse response = (HttpServletResponse) servletResponse;
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json;charset=UTF-8");
        response.setHeader("Access-Control-Allow-Origin","*");
        response.setHeader("Access-Control-Allow-Credentials","true");
        response.setHeader("Access-Control-Allow-Methods","*");
        response.setHeader("Access-Control-Allow-Headers","Content-Type,Authorization");
        response.setHeader("Access-Control-Expose-Headers","*");
        filterChain.doFilter(request,response);




    }

    @Override
    public void destroy() {
        Filter.super.destroy();
    }
}



```

2. 在启动类添加@ServletComponentScan注解,用来扫描Filter过滤器
```java

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletComponentScan;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@EnableDiscoveryClient
@SpringBootApplication
/**
 * @ServletComponentScan 是 Spring 框架中的一个注解，
 * 它可以自动扫描 @WebServlet、@WebFilter 和 @WebListener 注解，
 * 并将它们注册为 Servlet、Filter 和 Listener。
 */
@ServletComponentScan
public class CommodityApplication {
    public static void main(String[] args) {
        SpringApplication.run(CommodityApplication.class,args);
    }
}
```
