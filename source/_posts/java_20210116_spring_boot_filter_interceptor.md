---
layout:     post
title:      SpringBoot 过滤器和拦截器的使用
date:       2021-01-16 12:25:13
author:     liangtong
catalog: true
categories: Java
tags: Spring AOP

---

#### 

![](/post/java/20210116/filter_interceptor.png)

#### 场景说明

**适配演示场景需求，数据只看不允许修改。**即需要在特定场合下拦截所有的POST 、 PUT 、 DELETE等方法，不允许修改数据。



#### 过滤器

##### 类注解

拦截特定的URL（比如以下示例的`/text-context`），重构3个方法，其中 `init`和`destory`方法一般用不到。

`doFilter`方法是过滤器的核心，在里边可以进行过滤操作，对符合需求的进行执行` filterChain.doFilter(request,response)`通过

```Java 
@Component
@WebFilter(urlPatterns = "/test-context",filterName = "blosTest")
public class TestFilter implements Filter{
  	@Override
    public void init(FilterConfig filterConfig) throws ServletException {
 
    }
 
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request= (HttpServletRequest) servletRequest;
        String method = request.getMethod();
        if ("POST".equals(method) || "PUT".equals(method) || "DELETE".equals(method)) {
                return;//拦截返回
        }
        HttpServletResponse response = (HttpServletResponse) servletResponse;
        System.out.printf("过滤器实现");
        filterChain.doFilter(request,response);
    }
 
    @Override
    public void destroy() {
 
    }
}
```



##### 配置启动类

通过配置Bean的方式，可以多个过滤器

```Java
@Component
public class xxxxFilterConfig{
		//过滤器
    @Bean
    public FilterRegistrationBean filterRegistrationBean(){
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        List<String> urlPatterns = new ArrayList<String>();
 
        TestFilter testFilter = new TestFilter();   //new过滤器，过滤器不用添加注解
        urlPatterns.add("/text-context");      //指定需要过滤的url
        filterRegistrationBean.setFilter(testFilter);       //set
        filterRegistrationBean.setUrlPatterns(urlPatterns);     //set
        return filterRegistrationBean;
    }

}
```



#### 拦截器

主要

+ 实现`HandlerInterceptor`接口 ，编写拦截器

  接口中包含3个方法：

  **preHandle是请求执行前执行的，**

  **postHandler是请求结束执行的，但只有preHandle方法返回true的时候才会执行。**

  **afterCompletion是视图渲染完成后才执行，同样需要preHandle返回true**

```Java 
public class MyInterceptor implements HandlerInterceptor {
    //在请求处理之前进行调用（Controller方法调用之前
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        System.out.printf("preHandle被调用");
        String method = httpServletRequest.getMethod();
        if ("POST".equals(method) || "PUT".equals(method) || "DELETE".equals(method)) {
                return false;//拦截返回
        }
        return true;
    }
 
    //请求处理之后进行调用，但是在视图被渲染之前（Controller方法调用之后）
    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle被调用");
    }
 
    //在整个请求结束之后被调用，也就是在DispatcherServlet 渲染了对应的视图之后执行（主要是用于进行资源清理工作）
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        System.out.println("afterCompletion被调用");
    }
}

```

+ 配置

```Java
@Configuration
class WebMvcConfigurer extends WebMvcConfigurerAdapter{
        //增加拦截器
        public void addInterceptors(InterceptorRegistry registry){
            registry.addInterceptor(new MyInterceptor())    //指定拦截器类
                    .addPathPatterns("/Handles");        //指定该类拦截的url
        }
 }

```



#### AOP

思想比较简单，对特定package里的方法进行AOP操作

```Java 
@Aspect
@Component
@Slf4j
public class TeachModeAspect {

    @Value("${system.mode:teach}")
    private String systemMode;

    @Pointcut("execution(* xx.yy.zz.controller..web.*.*(..))")
    public void pointcut() {
        log.info("拦截请求start");
    }

    @Before("pointcut()")
    public void doBefore(JoinPoint joinPoint) {
        if (systemMode.equals("teach")){
            ServletRequestAttributes attributes =
                    (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
            HttpServletRequest request = attributes.getRequest();
            //拦截所有 PUT 、 POST 、 DELETE 方法
            String method = request.getMethod();
            if ("POST".equals(method) || "PUT".equals(method) || "DELETE".equals(method)) {
                throw new DataException("XX模式，限制操作！");
            }
        }
    }

}
```



#### 总结

+ 过滤器和拦截器的触发时机不同：

  + 过滤器是在请求进入容器后，进入servlet之前进行预处理的

  + 拦截器是在请求经过servlet后进行处理的

+ 拦截器由Spring提供并管理，可以获取IOC容器里的各个bean；过滤器不行。
+ 过滤器是基于回调函数，拦截器是基于代理实现
+ 过滤器依赖servlet容器回调完成，拦截器通过AOP方式来执行
+ 过滤器的生命周期由servlet容器管理，拦截器由IOC容器管理。







