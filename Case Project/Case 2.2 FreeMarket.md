# Case 1.2.1 FreeMarker



> 该文档包含 FreeMarket 的基本使用及常见用法

> https://github.com/black-ant/case/tree/master/case%202.2%20Freemarker
> http://www.antblack.xyz/



## 一 .  基础知识点

```java
> FreeMarker是一款模板引擎 , 用来在MVC模式的Web开发框架中生成HTML页面 , 通常由Java程序准备要显示的数据，由FreeMarker生成页面，通过模板显示准备的数据，简单来讲就是模板加数据模型，然后输出页面

// 特点 
FreeMarker不是一个Web应用框架，而适合作为Web应用框架一个组件
FreeMarker与容器无关，因为它并不知道HTTP或Servlet；FreeMarker同样可以应用于非Web应用程序环境
FreeMarker更适合作为Model2框架（如Struts）的视图组件，你也可以在模板中使用JSP标记库
    
 // 个人看法 : FreeMarker 和 JSP 很类似 , 是早期 JSP 的替代品 , 但是相对于语法而言 , 更喜欢 Thymeleaf 的风格 ,  鉴于现在使用人员越来越少 , 不进行深入的了解 , 仅作为此类的框架的简单记录和对比
```

## 二 . Spring 用法

### 2.1 Maven 

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```

### 2.2 简单输出

```html
<h2>欢迎 : ${user.username}</h2>
```

### 2.3 List 语法

```html
<#assign seq = ["winter", "spring", "summer", "autumn"]>
<#list seq as x>
	${x_index + 1}. ${x}<#if x_has_next>,</#if>
</#list>
```

### 2.4 if 语法

```html
<#assign age=11>
<#if (age>3&&age<10)>中级
<#elseif (age>10&&age<15)>高级
<#elseif (age>15)>在线炒粉
<#else> 初级
</#if>
```

### 2.5 Switch 语法

```
<#assign age=11>
<#if (age>3&&age<10)>中级
<#elseif (age>10&&age<15)>高级
<#elseif (age>15)>在线炒粉
<#else> 初级
</#if>
```

### 2.6 noparse 

```html
noparse指令指定FreeMarker不处理该指定里包含的内容,
> 即内部数据会原样输出

<#noparse>
<#list books as book>
   <tr><td>${book.name}<td>作者:${book.author}
</#list>
</#noparse>
输出如下:
<#list books as book>
   <tr><td>${book.name}<td>作者:${book.author}
</#list>
```

### 2.7 escape

```html
escape 可以用于加上escape表达式 ,但不会影响字符串内的插值,只会影响到body内出现的插值,使用escape指令的语法

<#escape x as x?html>
First name:${firstName}
Last name:${lastName}
Maiden name:${maidenName}
</#escape>

First name:${firstName?html}
Last name:${lastName?html}
Maiden name:${maidenName?html}
```

### 2.8 **assign**

```
assign指令在前面已经使用了多次,它用于为该模板页面创建或替换一个顶层变量,assign指令的用法有多种,包含创建或替换一个顶层变量,或者创建或替换多个变量等

```

### 2.9 **setting**

```
	该指令用于设置FreeMarker的运行环境,该指令的语法格式如下:<#setting name=value>,在这个格式中,name的取值范围包含如下几个:
	• locale:该选项指定该模板所用的国家/语言选项
	• number_format:指定格式化输出数字的格式
	• boolean_format:指定两个布尔值的语法格式,默认值是true,false
	• date_format,time_format,datetime_format:指定格式化输出日期的格式
	• time_zone:设置格式化输出日期时所使用的时区

```

### 2.10  **macro , nested , return****指令**

```
macro可以用于实现自定义指令,通过使用自定义指令,可以将一段模板片段定义成一个用户指令,使用macro指令的语法格式如下:

<#macro name param1 param2 ... paramN>
...
<#nested loopvar1, loopvar2, ..., loopvarN>
...
<#return>
...
</#macro>

<#macro name param1 param2 ... paramN>
...
<#nested loopvar1, loopvar2, ..., loopvarN>
...
<#return>
...
</#macro>

在上面的格式片段中,包含了如下几个部分:
	• name:name属性指定的是该自定义指令的名字,使用自定义指令时可以传入多个参数
	• paramX:该属性就是指定使用自定义指令时报参数,使用该自定义指令时,必须为这些参数传入值
	• nested指令:nested标签输出使用自定义指令时的中间部分
	• nested指令中的循环变量:这此循环变量将由macro定义部分指定,传给使用标签的模板
	• return指令:该指令可用于随时结束该自定义指令.
	
<#macro book>   //定义一个自定义指令
j2ee
</#macro>
<@book />    //使用刚才定义的指令
上面的代码输出结果为:j2ee
```



## 三 . 其他语法

```

```

