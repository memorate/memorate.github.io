---
layout: post
title: SpringBoot - 默认异常处理机制探索
tags:
- Java
- SpringBoot
categories: SpringBoot
description: Exception处理机制
---  
**SpringBoot默认异常处理机制探索**

<!-- more -->
## 前言
使用 Postman 和浏览器访问 SpringBoot 的接口**异常时默认**会返回如下信息，本文着重于**探索异常抛出到返回下图中信息的整个过程**。  
`注：`异常处理过程实质上是一次特殊的请求处理过程。  
![]({{ "/assets/img/20200728/20200728001.jpg"}})![]({{ "/assets/img/20200728/20200728002.jpg"}})  
## 流程
![]({{ "/assets/img/20200728/20200728003.png"}})
## 自动配置
1)自动配置类**ErrorMvcAutoConfiguration**中配置了一些组装错误信息页所需要的组件(Bean);  
2)第五个注解注入的**ServerProperties**类中定义了**ErrorProperties**类，ErrorProperties中又定义了**默认的异常处理路径**是`/error`;
```java
package org.springframework.boot.autoconfigure.web.servlet.error;

@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class })
@AutoConfigureBefore(WebMvcAutoConfiguration.class)
@EnableConfigurationProperties({ ServerProperties.class, ResourceProperties.class, WebMvcProperties.class })
public class ErrorMvcAutoConfiguration {
    // 省略代码
}
```
以下是ErrorMvcAutoConfiguration中定义的几个较为重要的组件(Bean)：  
**1.DefaultErrorAttributes**  
该Bean用于接收并传递异常相关的关键属性，如时间戳、请求状态、异常类名、异常信息、异常请求地址等。
```java
@Bean
@ConditionalOnMissingBean(value = ErrorAttributes.class, search = SearchStrategy.CURRENT)
public DefaultErrorAttributes errorAttributes() {
    return new DefaultErrorAttributes(this.serverProperties.getError().isIncludeException());
}
```
**2.BasicErrorController**  
该Bean用于提取异常中的关键信息，并将其封装成错误信息页返回给请求调用者。(详细解析在[这里](#here))  
```java
@Bean
@ConditionalOnMissingBean(value = ErrorController.class, search = SearchStrategy.CURRENT)
public BasicErrorController basicErrorController(ErrorAttributes errorAttributes,
		ObjectProvider<ErrorViewResolver> errorViewResolvers) {
    return new BasicErrorController(errorAttributes, this.serverProperties.getError(),
			errorViewResolvers.orderedStream().collect(Collectors.toList()));
}
```
**3.ErrorPageCustomizer**  
该Bean用于向Spring注册**ErrorPage**(错误页抽象成的类)，注册时将ErrorPage的URL路径设为了**/error**。
```java
@Bean
public ErrorPageCustomizer errorPageCustomizer(DispatcherServletPath dispatcherServletPath) {
    return new ErrorPageCustomizer(this.serverProperties, dispatcherServletPath);
}
```
**4.其他Beans**  
除了上述三个Beans，还有**PreserveErrorControllerTargetClassPostProcessor**和
内置配置类**DefaultErrorViewResolverConfiguration**、**WhitelabelErrorViewConfiguration**。
## 一、StandardHostValve
1)Tomcat采用**Pipeline**、**Valve**机制(相关内容可自行查询);  
2)**StandardHostValve**是Tomcat中的一个**默认基础Valve**，负责处理**Pipeline**中流传过来的请求;  
3)StandardHostValve会在Context中寻找合适的ErrorPage(未找到则使用默认的ErrorPage)，并根据ErrorPage里的location转发请求;  
```java
package org.apache.catalina.core;
// 以下是节选的部分与本文有关的逻辑处理代码，完整代码自行搜索查看
final class StandardHostValve extends ValveBase {
    // valve的invoke()方法
    public final void invoke(Request request, Response response) throws IOException, ServletException {
        // 获取到我们抛出的异常
        Throwable t = (Throwable) request.getAttribute(RequestDispatcher.ERROR_EXCEPTION);
        if (response.isErrorReportRequired()) {
            AtomicBoolean result = new AtomicBoolean(false);
            response.getCoyoteResponse().action(ActionCode.IS_IO_ALLOWED, result);
            if (result.get()) {
                if (t != null) {
                    // 进入throwable()方法
                    throwable(request, response, t);
                } else {
                    status(request, response);
                }
            }
        }
    }

    // throwable()方法，用于处理request中的异常，产生异常对应的response
    protected void throwable(Request request, Response response, Throwable throwable) {
            Context context = request.getContext();
            Throwable realError = throwable;
            // 根据异常类名来查找对应的ErrorPage
            // findErrorPage()最终执行了ErrorPageSupport类中的find()方法
            // find()方法实际是去ErrorPageSupport维护的一个 Map<String, ErrorPage> 中查找ErrorPage
            // 查询Map的key是throwable对象所代表类的完整类名
            ErrorPage errorPage = context.findErrorPage(throwable);
            if (errorPage != null) {
                // 省略
            } else {
                // 找不到合适的ErrorPage，设置HttpStatus为500
                response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
                // 将返回标记为Error
                response.setError();
                // 进入status()方法
                status(request, response);
            }
        }

    // status()方法，用于处理 HttpStatus
    private void status(Request request, Response response) {
        int statusCode = response.getStatus();
        Context context = request.getContext();
        // 根据HttpStatus来查找对应的ErrorPage
        // 去ErrorPageSupport维护的另一个 Map<Integer, ErrorPage> 中查找ErrorPage
        ErrorPage errorPage = context.findErrorPage(statusCode);
        if (errorPage == null) {
            // 找到则使用默认的ErrorPage
            errorPage = context.findErrorPage(0);
        }
        if (errorPage != null && response.isErrorReportRequired()) {
            // 省略部分属性设置代码

            // 进入custom()方法
            // custom()方法根据errorPage里的location将请求发送到相应的Controller
            // 这里会根据 /error 这个地址，将请求发送到BasicErrorController(当然中间还会经过DispatcherServlet)
            if (custom(request, response, errorPage)) {
                // 省略
            }
        }
    }

    // custom()方法，用于转发请求
    private boolean custom(Request request, Response response, ErrorPage errorPage) {
        try {
            ServletContext servletContext =
                request.getContext().getServletContext();
            // errorPage.getLocation()值为 /error
            RequestDispatcher rd =
                servletContext.getRequestDispatcher(errorPage.getLocation());
            if (response.isCommitted()) {
                // 省略
            } else {
                // 转发请求
                rd.forward(request.getRequest(), response.getResponse());
                response.setSuspended(false);
            }
            return true;
        } catch (Throwable t) {
            // 省略
        }
    }
}
```  
## 二、DispatcherServlet
1、**DispatcherServlet** 是 `org.springframework.web.servlet` 包下的一个 Java 类。  
```text
官方注释：
Central dispatcher for HTTP request handlers/controllers, e.g. for web UI controllers or HTTP-based remote service exporters. 
Dispatches to registered handlers for processing a web request, providing convenient mapping and exception handling facilities.

个人翻译：
DispatcherServlet 是 SpringBoot 中 HTTP 请求的中央调度器，为 Web UI 或基于 HTTP 的远程发射器发送的请求提供服务。
其主要作用是向注册的可以处理 web 请求的 handlers 分发请求，并提供方便的映射、异常处理设施。

个人理解：
DispatcherServlet 类是 SpringBoot 的调度器，它负责组织和协调不同组件完成请求并返回响应结果。
DispatcherServlet 的主要任务是：①将请求发送至对应的 Controller/Handler、②请求结果处理及返回（正常及异常处理结果）。(本文只聚焦于异常处理部分)
```
## 三、<span id="here">BasicErrorController</span>  

