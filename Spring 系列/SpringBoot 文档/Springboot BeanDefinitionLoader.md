# Springboot BeanDefinitionLoader

```
> BeanDefinitionLoader
	|- BeanDefinition 加载器（Loader），负责 Spring Boot 中，读取 BeanDefinition 
	F- final Object[] sources; //  来源的数组
	F- AnnotatedBeanDefinitionReader  //注解的 BeanDefinition 读取器
	F- XmlBeanDefinitionReader  // XML 的 BeanDefinition 读取器
	F- BeanDefinitionReader // Groovy 的 BeanDefinition 读取器
	F- ClassPathBeanDefinitionScanner  // Classpath 的 BeanDefinition 扫描器
	F- ResourceLoader  // 资源加载器
	CM- BeanDefinitionLoader
		|- 断言非 Null 非 empty
		|- 创建 AnnotatedBeanDefinitionReader 对象 : 设置给 annotatedReader 属性
		|- 创建 XmlBeanDefinitionReader 对象 : 设置给 xmlReader 属性
		|- 创建 GroovyBeanDefinitionReader 对象 : 设置给 groovyReader 属性
		|- 创建 ClassPathBeanDefinitionScanner 对象
	F- Set<Class<?>> primarySources
	M- setBeanNameGenerator : 设置供底层读取器和扫描器使用的bean名称生成器
	M- setResourceLoader : 设置要由基础读取器和扫描器使用的资源加载器。
	M- setEnvironment
	M- load :  
		|- 遍历 sources 数组，逐个加载
	M- load(Object source)
		|- 如果是 Class 类型，则使用 AnnotatedBeanDefinitionReader 执行加载
		|- 果是 Resource 类型，则使用 XmlBeanDefinitionReader 执行加载
		|- 如果是 Package 类型，则使用 ClassPathBeanDefinitionScanner 执行加载
		|- 如果是 CharSequence 类型，则各种尝试去加载
		|- 无法处理的类型，抛出 IllegalArgumentException 异常
	M- load(Class<?> source)
		|- 先执行 Groovy 相关操作
		|- isComponent(source) -> 执行注册
			|- 如果有 @Component 注解，则返回 true
			|- 无构造器 , 等返回 false 
			|- AnnotatedBeanDefinitionReader#register 注册
	M- load(Resource source) : 使用 XmlBeanDefinitionReader 执行加载
		|- 先执行 Groovy 相关操作
		|- 使用 XmlBeanDefinitionReader 加载 BeanDefinition
	M- load(GroovyBeanDefinitionSource source) : Groovy 操作
	M- load(Package source) : 使用 ClassPathBeanDefinitionScanner 执行加载
		|- this.scanner.scan(source.getName());
	M- load(CharSequence source) : 各种尝试去加载
		|- 解析 source 。因为，有可能里面带有占位符。
		|- 尝试按照 Class 进行加载
			|- load(ClassUtils.forName(resolvedSource, null))
		|- 尝试按照 Resource 进行加载
			|- findResources(resolvedSource); 
			|- 遍历 resources 数组，调用 #isLoadCandidate(Resource resource) 方法
		|- 尝试按照 Package 进行加载
		|- 
    M- findResources	
		|- 1 创建 ResourceLoader 对象
		|- 2 获取Resource 数组
		|- 3 返回 Resource 对象
	M- isLoadCandidate
    	|- resource 不存在 , 返回false
    	|- 如果resource是ClassPathResource	
    		|- getPath
    		|- return Package.getPackage
		|- 最终返回true
	M- findPackage
		|- 获得 source 对应的 Package 。如果存在，则返回
		|- 创建 ResourcePatternResolver 对象
		|- 尝试加载 source 目录下的 class 们
		|- 遍历 resources 数组
			|- 获得类名
			|- 按照 Class 进行加载 BeanDefinition
		|- 返回 Package
```