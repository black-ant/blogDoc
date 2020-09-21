# SpringBoot SpringApplication笔记

> 该文档包含 Springboot SpringApplication 的源码逻辑 , 可用通过该文档快速了解 Springboot SpringApplication 的流程及后续复习

[TOC]



## 源码分析

### SpringApplication 

```java
> 常见的 SpringBoot 启动类的格式
	1 > 使用 @SpringBootApplication 注解，标明是 Spring Boot 应用。通过它，可以开启自动配置的功能。
	2 > main 方法 : 调用 SpringApplication#run(Class<?>... primarySources) 方法，启动 Spring Boot 应用
	
> SpringApplication 
	F- resourceLoader 属性，资源加载器
	F- primarySources 属性，主要的 Java Config 类的数组。在文初提供的示例，就是 MVCApplication 类。
	F- webApplicationType 属性，调用 WebApplicationType#deduceFromClasspath() 方法，通过 classpath ，判断 Web 应用类型。
	F- listeners 属性，ApplicationListener 数组。
	F- mainApplicationClass 属性，调用 #deduceMainApplicationClass() 方法，获得是调用了哪个 #main(String[] args) 方法
	M- getSpringFactoriesInstances : 先加载指定类的数组 ,然后创建对象 , 最后排序
		-> loadFactoryNames : 加载指定类型对应的，在 META-INF/spring.factories 里的类名的数组
		-> createSpringFactoriesInstances : 用于创建对象
		-> AnnotationAwareOrderComparator : 排序对象们
	M- run()
         T-> 计时 ,创建 StopWatch  并且启动 , 用于记录启动的时长
		-> configureHeadlessProperty() : 配置 headless 属性
            ?- Headless模式是系统的一种配置模式。在该模式下，系统缺少了显示设备、键盘或鼠标
            ?- 该模式下可以创建轻量级组件 , 收集字体等前置工作
		-> getRunListeners : 获取 SpringApplicationRunListeners ,并且开启监听 listeners.starting()
		-> 1 创建  ApplicationArguments 对象
		-> 2 prepareEnvironment 加载属性配置(传入 listener + arguments )。 
			执行完成后，所有的 environment 的属性都会加载进来 (application.properties等)
		-> 3 打印banner(printBanner) 
			 准备异常对象(getSpringFactoriesInstances.SpringBootExceptionReporter )
		-> 4 创建Spring 容器(createApplicationContext)
		-> 5 调用所有初始化类的 initialize 方法(prepareContext) , 初始化Spring 容器
		-> 6 刷新容器(refreshContext) , 执行 Spring 容器的初始化的后置逻辑(afterRefresh)
         T-> 计时完成    
		-> 7 通知 SpringApplicationRunListener  , 执行异常处理等收尾
	M- prepareEnvironment
		-> 1 调用 #getOrCreateEnvironment() 方法，创建 ConfigurableEnvironment(environment) 对象
		-> 2 configureEnvironment(ConfigurableEnvironment environment, String[] args) 
			?- 配置 environment 变量
		-> 2.2 ConfigurationPropertySources.attach(environment)   
		-> 3 通知 SpringApplicationRunListener , 环境变量OK
		-> 4 绑定 enviroment 到 SpringApplication
		-> 5 转换自定义 enviroment
	M- configureEnvironment
    	IF addConversionService : 需要添加 conversionService
            -> 设置 environment 的 conversionService 属性
    	-> 增加 environment 的 PropertySource 属性源
    	-> 配置 environment 的 activeProfiles 属性
    M- createApplicationContext
    	-> 根据 webApplicationType 类型，获得对应的 ApplicationContext 对象
    		- SERVLET -- AnnotationConfigServletWebServerApplicationContext
    		- REACTIVE -- AnnotationConfigReactiveWebServerApplicationContext
    		ELSE- AnnotationConfigApplicationContext
	M- prepareContext : 准备 ApplicationContext 对象，主要是初始化它的一些属性
		-> 1 设置 context 的 environment 属性
		-> 2 调用 #postProcessApplicationContext(ConfigurableApplicationContext context) 方法，
			?- 设置 context 的一些属性
		-> 3 调用 #applyInitializers(ConfigurableApplicationContext context) 方法，
			?- 初始化 ApplicationContextInitializer 
		-> 4 调用 SpringApplicationRunListeners#contextPrepared
			?- 通知 SpringApplicationRunListener 的数组，Spring 容器准备完成
		-> 5 设置 beanFactory 的属性
		-> 6 调用 #load(ApplicationContext context, Object[] sources) 方法，加载 BeanDefinition 们
			-> 创建 BeanDefinitionRegistry 对象
			-> 设置 loader 属性
			-> 执行BeanDefine 加载
	M- refreshContext : 启动（刷新） Spring 容器
    	-> 调用 AbstractApplicationContext#refresh() 方法，启动（刷新）Spring 容器
    	-> 调用 ConfigurableApplicationContext#registerShutdownHook() 方法，注册 ShutdownHook 钩子
    		?- 用于注销相应的 Bean 们
    M- callRunners :调用 ApplicationRunner 或者 CommandLineRunner 的运行方法
    	-> 获得所有 Runner 们，并进行排序 , 遍历 , 执行逻辑
        	- runners.addAll
        	- AnnotationAwareOrderComparator.sort(runners);
        	- callRunner((CommandLineRunner) runner, args);
    	
    	
// 简述 : 
1 准备计时 , 进入Headless 模式 , 开启监听
2 加载 Environment 属性 , 打印Banner , 准备异常收集类
3 创建容器 , 调用 initialize初始化容器 , 执行 Spring 容器的初始化的后置逻辑
4 通知Lisener , 记录时间 , 异常处理收尾
```

### 监听器

```
SpringApplicationRunListeners : SpringApplicationRunListener 数组的封装

SpringApplicationRunListener : SpringApplication 运行的监听器接口

EventPublishingRunListener : 
	 ?- 实现 SpringApplicationRunListener、Ordered 接口
	 ?- 将 SpringApplicationRunListener 监听到的事件，转换成对应的 SpringApplicationEvent 事件
	 ?- 发布到监听器们
```