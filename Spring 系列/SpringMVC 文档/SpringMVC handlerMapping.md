# Spring mvc Handle Mapping 

### AbstractHandlerMapping

```
精尽 Spring MVC  AbstractHandlerMapping 笔记 
=======================

## HandlerMapping  ： 处理匹配接口
  > HandlerExecutionChain getHandler  : 获得 请求对应的处理器和拦截器们

## AbstractHandlerMapping  : 实现了【获得请求对应的处理器和拦截器们】的骨架逻辑
  >  getHandlerInternal(HttpServletRequest request) : 将详细逻辑交由子类进行
  >  defaultHandler 属性，默认处理器。在获得不到处理器时，可使用该属性
  >  interceptors 属性，配置的拦截器数组
  >  adaptedInterceptors 属性，初始化后的拦截器 HandlerInterceptor 数组
  >  initApplicationContext ( ) : 初始化拦截器
    - ? WebApplicationObjectSupport => ApplicationObjectSupport => ApplicationContextAware
    - - < 调用 #extendInterceptors(List<Object> interceptors) 方法，空方法。交给子类实现
    - - < 扫描已注册的 MappedInterceptor 的 Bean 们，添加到 mappedInterceptors 中
    - - <  将 interceptors 初始化成 HandlerInterceptor 类型，添加到 mappedInterceptors 中
  >  getHandler : 获得请求对应的 HandlerExecutionChain 对象
    - - 1 .  1  调用 #getHandlerInternal(HttpServletRequest request) 抽象方法，获得 handler 对象
    - - 1 .  2  获得不到，则调用 #getDefaultHandler() 方法，使用默认处理器
    - - 1 .  3  还是获得不到，则返回 null
    - - 1 .  4  如果找到的处理器是 String 类型，则调用 BeanFactory#getBean(String name) 方法
    - - 1 .  5  调用 #getHandlerExecutionChain(Object handler, HttpServletRequest request) 方法，获得 HandlerExecutionChain 对象
    - - 1 .  6  返回 之前 创建的 HandlerExecutionChain 对象


### 上述子类实现 ：AbstractUrlHandleMapping : 基于URL 进行实现


### 上述子类实现 ：AbstractHandleMethodMapping : 基于 method  进行实现

### MatchableHandlerMapping （）
  >  定义判断请求和指定 pattern 路径是否匹配的接口方

### DispatcherServlet 
  >  在 DispatcherServlet 中，通过调用 #initHandlerMappings(ApplicationContext context) 方法，初始化 HandlerMapping 们

```