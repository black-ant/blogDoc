# Spring MVC 源码逻辑



[TOC]



### RequestToViewNameTranslator 组件

```
> 请求到视图名的转换器接口
	|- 在 DispatcherServlet 中，我们已经看到，在 ModelAndView 不存在对应的视图时，会通过 RequestToViewNameTranslator 来获取默认的视图名，作为其视图
	
> DefaultRequestToViewNameTranslator
	|- 实现 RequestToViewNameTranslator 接口，默认且是唯一的 RequestToViewNameTranslator 实现类
	F- SLASH 
	F- prefix 
	F- suffix 
	F- separator 
	F- stripLeadingSlash 
```

### WebApplicationContext

```
Servlet WebApplicationContext 笔记
===============

## DispatcherServlet 

> 分为 Spring 部分 和 Servelt 部分 
	|- spring : DispatcherServlet - FrameworkServlet - HttpServletBean - EnvironmentAware - Aware - EnvironmentCapable
	|- Servlet 部分 : HttpServlet , GenericServlet , ServletConfig , Servlet

 - > HttpServletBean : 负责将ServletConfig 设置到当前的Servlet 对象中
 - > FrameworkServlet  : 负责初始化 WebApplicationContext  容器
 - > DispatcherServlet  : 负责初始化MVC的各个组件 , 以及处理客户端的请求


> HttpServletBean
  - > ConfigurableEnviroment : 通过 get set 方法设置 , get 之前会断言 , set 时若为空会调用createEnviroment 新建
  - > requiredProperties  : 通过 addRequiredProperty(String property)  添加 
  - > init () : 负责将ServletConfig 设置到当前Servlet 对象中 
  - - < 1 先获取 PropertyValue 对象 ( servlet 配置的 init-param )
  - - < 2 获取BeanWrapper 对象 ：将当前的Servlet 转换为一个BeanWrapper 对象 ，从而将 PropertyValue 注入 ( setPropertyValues )
  - - < 3 获取 ResourceLoader 对象 
  - - < 4 BeanWrapper 对象执行 registerCustomerEditor 方法 ： 此方法用于注册自定义编辑器 ， 用于解析Resource 
  - - < 5 调用子类实现initServletBean 实现自定义的初始化逻辑 
 

> FrameworkServlet 
 - > 实现ApplicationContextAware 接口 , 继承HttpServletBean 抽象类 , 负责初始化WebApplicationContext 容器 
 - > Class<?> contextClass : 创建WebApplicationContext 的类型
 - > contextConfigLocation : 配置文件的地址 ， 例如 /WEB-INF/spring-servlet.xml 
 - > WebApplicationContext : 创建的对象 
 - - < 1 通过构造器方法 ： 此方法是将对象直接传入
 - - < 2  实现ApplicationContextAware 接口 ， 将ApplicationContext 注入
 - - < 3  findWebApplicationContext() 
 - - < 4  createWebApplicationContext(WebApplicationContext parent) 
 - > initServletBean () : 进一步初始化Servlet 对象
 - - < 1  打log
 - - < 2  记录开始时间 
 - - < 3  initWebApplicationContext() 实现初始化
 - - < - < 3 . 1 获得根 WebApplicationContext 对象 , WebApplicationContextUtils.getWebApplicationContext(getServletContext());
 - - < - < 3 . 2 获得 WebApplicationContext  wac 
 - - < - < 3 . 3 构造器已经注入 ， 直接将构造器注入的对象赋值给wac 
 - - < - < 3 . 4 为 ConfigurableWebApplicationContext  且为激活 ， 则会进行激活 configureAndRefreshWebApplicationContext(cwac);
 - - < - <  - < 3 . 4 . 1  当 wac 使用了默认编号 ， 则重新设置ID ， 当 contextId  不为空 ，则使用 contextId  ， 否则 自动生成
 - - < - <  - < 3 . 4 . 2  设置 wac 的 servletContext、servletConfig、namespace 属性
 - - < - <  - < 3 . 4 . 3  添加监听器 SourceFilteringListener 到 wac 中 （ 实现将原始对象触发的事件，转发给指定监听器  ）
 - - < - <  - < 3 . 4 . 4  初始化属性资源
 - - < - <  - < 3 . 4 . 5  执行处理完 WebApplicationContext 后的逻辑
 - - < - <  - < 3 . 4 . 6  执行自定义初始化
 - - < - <  - < 3 . 4 . 7 刷新 wac ，从而初始化 wac
 - - < - < 3 . 5 前面无法获取 ， 则从 ServletContext 获取 WebApplicationContext  >  wac = findWebApplicationContext();
 - - < - < 3 . 6 仍然无法获取 ，  创建一个 WebApplicationContext 对象
 - - < - < 3 . 7 在未触发刷新时间的前提下 ，主动刷新事件 onRefresh
 - - < - <  - < 3 . 7 . 1 在 wac 执行刷新完成后，会回调在该方法中，注册的 SourceFilteringListener 监听器
 - - < - < 3 . 8 将  context  设置到 ServletContext  
 - - < 4 initFrameworkServlet() ： 该方法为空实现 ， 用于子类进行扩展
configureAndRefreshWebApplicationContext ()

```

### Servlet 3.0 集成

```
容器的初始化
===================

## 初期创建servlet 的方式 
  > 1  extends HttpServlet : 继承该类后 ， 重写doGet ， doPost 方法
  > 2  implements Filter : 重写 init , 重写 doFilter , 重写 destory 
  > 3  web. xml 中进行配置

## Spring 支持 servlet 3.0
  > implements ServletContainerInitializer 
  - - <  重写 onStartup(Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)
  - - <  - <  1 . 1 先获得List<WebApplicationInitializer> initializers
  - - <  - <  1 . 2 对传入的 webAppInitializerClasses 进行循环 ， 将 其中的class 加入  initializers list
  - - <  - <  1 . 3 对 initializers 进行 排序
  - - <  - <  1 . 4 for 循环List，获得其中的每个WebApplicationInitializer , 并且调用 initializer.onStartup(servletContext); 

## 无web .xml 创建 DispatcherServlet 
  >  onStartup(ServletContext servletContext)  
   - - <  super.onStartup(servletContext);  》 先调用父类启动逻辑
   - - <  registerDispatcherServlet(servletContext);  》 注册 DispacherServlt 
   - - < - < 1 . 1 断言 ， 判断 getServletName 返回不为NULL
   - - < - < 1 . 2 createServletApplicationContext()  》 创建 WebApplicationContext  
   - - < - < - <  1 . 2 . 1  new AnnotationConfigWebApplicationContext()
   - - < - < 1 . 3 createDispatcherServlet(servletAppContext)  》 创建 FrameworkServlet 对象
   - - < - < - <  1 . 3 . 1  return new DispatcherServlet(servletAppContext);
   - - < - < 1 . 4 setContextInitializers （） 》 设置初始化对象
   - - < - < 1 . 5 registration.setLoadOnStartup(1)  》 设置启动等级
   - - < - < 1 . 6 registration.addMapping(getServletMappings());  》 添加servletMapping
   - - < - < 1 . 7 registration.setAsyncSupported(isAsyncSupported());  》 是否支持异步
   - - < - < 1 . 8  注册过滤器

```

###  Spring Boot 集成 

```
精尽 Spring MVC 笔记 ： Spring Boot 集成 
===================


## spring Boot 加载 Servlet
  >  注册方式 一 ： Servlet 3.0 注解 + @ServletComponentScan
​```
@WebServlet("/hello")
public class HelloWorldServlet extends HttpServlet{}
@WebFilter("/hello/*")
public class HelloWorldFilter implements Filter {}
// 启动类添加
@ServletComponentSca
​```

  >  注册方式 二 ： RegistrationBean
​```
@Bean
public ServletRegistrationBean helloWorldServlet() {
    ServletRegistrationBean helloWorldServlet = new ServletRegistrationBean();
    myServlet.addUrlMappings("/hello");
    myServlet.setServlet(new HelloWorldServlet());
    return helloWorldServlet;
}

@Bean
public FilterRegistrationBean helloWorldFilter() {
    FilterRegistrationBean helloWorldFilter = new FilterRegistrationBean();
    myFilter.addUrlPatterns("/hello/*");
    myFilter.setFilter(new HelloWorldFilter());
    return helloWorldFilter;
}

​```

  >  注册方式三 ： implements ServletContextInitializer
  - - <  重写 onStartup(ServletContext servletContext) 
  - - <  - < servletContext.addServlet 
  - - <  - < servlet.addMapping(JAR_HELLO_URL);
  - - <  - < sservletContext.addFilter
  - - <  - < filter.addMappingForUrlPatterns(dispatcherTypes, true, JAR_HELLO_URL);


## SpringBoot 中 Servlet 加载流程的源码分析
 >  Initializer 被替换为 TomcatStarter
 >  TomcatStarter 中的 ServletContextInitializer 是关键
 >  EmbeddedWebApplicationContext 中的 6 层迭代加载
  - - <  onRefresh() : ApplicationContext 的生命周期方法 
  - - <  createEmbeddedServletContainer () ： 
  - - <  getSelfInitializer () ：创建了一个匿名类
  - - <  getServletContextInitializerBeans() ： 用来加载 Servlet 和 Filter 的
  - - <  ServletContextInitializerBeans 
  - - <  addServletContextInitializerBeans ()

```

### 组件一览 笔记

```
组件一览
==============

## MultipartResolver : 内容类型( Content-Type )为 multipart/* 的请求的解析器接口
  >  例如 文件上传请求 ： 将 HttpServletRequest  封装成 MultipartHttpServletRequest  
  - - <   isMultipart : 是否为 multipart 请求
  - - <   MultipartHttpServletRequest resolveMultipart(HttpServletRequest request)  : 封装请求
  - - <   cleanupMultipart(MultipartHttpServletRequest request) ： 清除资源


## LocaleResolver ： 本地化( 国际化 )解析器接口
  - - <  Locale resolveLocale(HttpServletRequest request) ： 解析要使用的语言
  - - <   void setLocale ： 设置请求使用的语言

## ThemeResolver ： 主题解析器接口
  - - <  String resolveThemeName(HttpServletRequest request) ： 解析使用的主题
  - - <  setThemeName  ：设置所使用的主题

## HandlerMapping ： 处理器匹配接口 ，根据请求获得其的处理器和拦截器们的数组
  - - <  getHandler(HttpServletRequest request) ： 获得处理器和拦截器


## HandlerAdapter ： 处理器适配器接口
  - - <   supports(Object handler) ： 是否支持该处理器
  - - <   ModelAndView handle( ... ) : 执行处理器 ， 返回ModelAndView 
  - - <   getLastModified (  ) : 返回请求的最新更新时间

## HandlerExceptionResolver ： 处理器异常解析器的接口 ， 将处理器执行时发生的异常 ， 解析成对应的ModelAndView 结果
  - - <  resolveException ： 转换成对应的 ModelAndView 结果

## RequestToViewNameTranslator ： 请求到视图名的的转换接口
  - - <  getViewName ： 根据请求 ， 获得其视图名


## ViewResolver ： 实体解析器接口 ， 根据视图名和国际化 ， 获得最终的视图 View 对象
  - - <  resolveViewName ： 获得最终的View 对象

## FlashMapManager ： flashMap 管理器接口 ， 负责重定向时 ， 保存参数到临时存储中
  - - <  FlashMap retrieveAndUpdate ： 恢复参数，并将恢复过的和超时的参数从保存介质中删除
  - - <  void saveOutputFlashMap ： 将参数保存起来
  > 重定向前 ， 保存参数到 Session 中
  > 重定向后 ， 从Session 中获得参数

```

### 请求处理一览

```
请求处理一览
==============

## FrameworkServlet
  > doGet  /  doPost  /  doPut  /  doDelete  /  doOptions  /  doTrace  /  service 
  > doGet & doPost & doPut & doDelete :  直接调用 processRequest
  > doOptions   :  processRequest , 但是Header 中若包含 Allow ， 则不交由父方法
  > doTrace :  processRequest  , 但响应的内容类型为 "message/http" ，则不需要交给父方法处理

## 不同 HttpMethod 的处理
  > service :  用于对HttpMathod 进行处理
  - - <  1 .  获得请求的方法  
  - - <  2 .  处理PATCH 请求 （ 请求为  HttpMethod.PATCH 或者 为NULL）
  - - <  3 .  调用父类 ， 处理其他的请求


## processRequest  (  )
 > 1 .  记录开始时间 
 > 2 .  准备记录异常对象 Throwable failureCause = null;
 > 3 .  获得 LocaleContext  previousLocaleContext 
 > 4 .  获得 LocaleContext localeContext
 > 5 .  获得 RequestAttributes previousAttributes
 > 6 .  获得 ServletRequestAttributes requestAttributes
 > 7 .  获得 WebAsyncManager asyncManager
 > 8 .  注册拦截器  asyncManager.registerCallableInterceptor
 > 9 . 初始化 context    initContextHolders
 > 10 .  doService(request, response);  》执行最后的逻辑  
 > 11 .  当出现异常时 ， 赋值给 failureCause 
 > 12 .  logResult  》 打印请求日志，并且日志级别为 DEBUG 
 > 13 .  publishRequestHandledEvent  》 发布 ServletRequestHandledEvent 事件

## DispatcherServlet -- doService （）
  > 1 .  调用 #logRequest(HttpServletRequest request) 方法 ， 打印请求日志，并且日志级别为 DEBUG 
  > 2 .  准备 attributesSnapshot  Map
  > 3 .  设置 Spring 框架中的常用对象到 request 的属性中
  > 4 .  request.setAttribute  》 包括 FlashMap inputFlashMap
  > 5 .  调用 #doDispatch(HttpServletRequest request, HttpServletResponse response) 方法，执行请求的分发
  > 6 .  执行 异步 操作


## doDispatch  :  此方法 用于执行请求的分发
   > 核心 1 ： 调用 #checkMultipart(HttpServletRequest request) 方法，检查是否是上传请求
   > 核心 2 ： 获得请求对应的 HandlerExecutionChain 对象 《 若获取不到，则根据配置抛出异常或返回 404 错误
   > 核心 3 ： 获得当前 handler 对应的 HandlerAdapter 对象
   > 核心 4 ： 前置处理 拦截器
   > 核心 5：  真正的调用 handler 方法，并返回视图
   > 核心 6：  后置处理 拦截器
   > 核心 7：  处理正常和异常的请求调用结果。
   > 核心 8：  当出现异常会对已经完成的拦截器进行处理

```

### RequestToViewNameTranslator

```
RequestToViewNameTranslator 笔记
=============================

>  RequestToViewNameTranslator  :  请求到视图名的转换器接口
 - - < 通过getViewName 获得其视图名

## DefaultRequestToViewNameTranslator
  >  实现 RequestToViewNameTranslator   ， 默认且唯一的 实现类
   - - <  prefix :  前缀
   - - <  suffix  :  后缀 
   - - <  stripLeadingSlash ： 是否移除开头
   - - <  stripTrailingSlash  ： 是否移除末尾
   - - <  stripExtension  ：  是否移除扩展名
   - - <  getViewName ： 获得请求名
   - - <  - < 1 . 1  this.urlPathHelper.getLookupPathForRequest(request)  》 先获得请求路径
   - - <  - < 1 . 2  获得视图名 ：prefix  +  调用transformPath(lookupPath) +suffix
   - - <  - <  - < 1 . 2  . 1 会根据上方的条件判断是否要移除 开头末尾
```



### ThemeResolver

```
ThemeResolver
	M- String resolveThemeName(HttpServletRequest request);
		?- 从请求中，解析出使用的主题
	M- void setThemeName
		?- 设置请求，所使用的主题。
		
AbstractThemeResolver
	F- defaultThemeName  = ORIGINAL_DEFAULT_THEME_NAME = "theme"
	M- setDefaultThemeName(...) 方法，设置默认的样式名。
	M- getDefaultThemeName(...) 方法，获取默认的样式名。
	
SessionThemeResolver : 将themeName 保存到 Session 中就可以了
	I- AbstractThemeResolver  : 同时对其实现
	M- resolveThemeName
		|-  (String) WebUtils.getSessionAttribute(request, THEME_SESSION_ATTRIBUTE_NAME)
	M- setThemeName
		|- WebUtils.setSessionAttribut
		
FixedThemeResolver 		

CookieThemeResolver : 将themeName 保存到 Cookie 中
	|- resolveThemeName
		|-  (String) request.getAttribute : 优先去request 中获取
		|- getCookieName -> cookieName
		|- Cookie cookie = WebUtils.getCookie(request, cookieName);
		|- cookie.getValue();
		|- request.setAttribute
	|- setThemeName
		|- request.setAttribute(THEME_REQUEST_ATTRIBUTE_NAME, themeName);
		|- addCookie(response, themeName);

```

![](C:\Users\10169\OneDrive\笔记文档\图片文件\SpringMVC\themeresolver.jpg)



### ViewResolver

```
> 1 DispatcherServlet  中 会通过 initViewResolvers 方法来初始化 viewResolvers 

DispatcherServlet
	M- initViewResolvers
        |- this.viewResolvers = null; // 置空
        B-> 如果自动扫描ViewResolvers -> detectAllViewResolvers
            |-  BeanFactoryUtils.beansOfTypeIncludingAncestors
            |- 转换为 ArrayList 后 sort
        E-> 获得名字为 VIEW_RESOLVER_BEAN_NAME 的 Bean 们
            |-  context.getBean -> ViewResolver vr 
            |-  this.viewResolvers = Collections.singletonList(vr);
        |- 	如果未获得到，则获得默认配置的 ViewResolver 类
        	|- getDefaultStrategies(context, ViewResolver.class)
        
ViewResolver 有以下几种
	- ContentNegotiatingViewResolver
	- BeanNameViewResolver
	- ThymeleafViewResolver
	- ViewResolverComposite
	- InternalResourceViewResolver
	
ContentNegotiatingViewResolver
	?- 基于内容类型来获取对应 View 的 ViewResolver 实现类(Content-Type)
	F- contentNegotiationManager
	F- ContentNegotiationManagerFactoryBean
	F- boolean useNotAcceptableStatusCode = false;
	F- List<View> defaultViews;
	F- List<ViewResolver> viewResolvers;
	M- initServletContext :初始化 viewResolvers 属性
		|- 扫描所有 ViewResolver 的 Bean 们
		B->  情况一，如果 viewResolvers 为空，则将 matchingBeans 作为 viewResolvers
			|- new ArrayList<>(matchingBeans.size());
			|- for 循环  matchingBeans
				|-  this.viewResolvers.add(viewResolver);
		E-> 如果 viewResolvers 非空，则和 matchingBeans 进行比对，判断哪些未进行初始化，那么需要进行初始化
			|- for 循环 viewResolvers
				|- 已存在在 matchingBeans 中，说明已经初始化，则直接 continue
				|- 不存在在 matchingBeans 中，说明还未初始化，则进行初始化
		|- 排序 viewResolvers 数组
		|- 设置 cnmFactoryBean 的 servletContext 属性
	M- afterPropertiesSet : 初始化 contentNegotiationManager 属性
		|- 如果 contentNegotiationManager 为空 , cnmFactoryBean.build()
	M- resolveViewName	
    	|- 获得 MediaType 数组
    		-> getMediaTypes
    	|- 获得匹配的 View 数组
    		-> getCandidateViews(viewName, locale, requestedMediaTypes)
    	|- 筛选最匹配的 View 对象
    		-> getBestView(candidateViews, requestedMediaTypes, attrs);
    	|- 筛选完成 , 返回 return bestView
    	|- 如果匹配不到 View 对象，则根据 useNotAcceptableStatusCode ，返回 NOT_ACCEPTABLE_VIEW 或 null
    M- getMediaTypes	
```

![](C:\Users\10169\OneDrive\笔记文档\图片文件\SpringMVC\viewResoler.jpg)