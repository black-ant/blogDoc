# Case 2.4  Spring Thymeleaf

> 该文档包含 Spring Thymeleaf 的使用方式

> https://github.com/black-ant/case/tree/master/case%202.4%20Thymeleaf
>
>  http://www.antblack.xyz/



[TOC]



## 一 . 代码 Demo



### 1 . 1 Maven Pom

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

### 1 . 2 Controller

```javaa

@RestController
public class DemoController {

    private Logger LOG = LoggerFactory.getLogger(this.getClass());

    @GetMapping("info")
    public String gerInfo(){
        return "my info";
    }

    @GetMapping("callback")
    public ModelAndView getCallBack(){
        ModelAndView modelAndView = new ModelAndView();
        LOG.info("is in callback");
        modelAndView.setViewName("pages/callback");
        return modelAndView;
    }
}

```

### 1 . 3 View

```

```



## 二 . 基本信息

### 使用标准

```
<html xmlns:th="http://www.thymeleaf.org">
```



### Thymeleaf 标注表达式

```
标准表达式功能

• 简单表达：
变量表达式： ${...}
选择变量表达式： *{...}
消息表达式： #{...}
链接网址表达式： @{...}
片段表达式： ~{...}

• 字面
文本文字：'one text'，'Another one!'，...
号码文字：0，34，3.0，12.3，...
布尔文字：true，false
空字面： null
文字标记：one，sometext，main，...

• 文字操作：
字符串连接： +
文字替换： |The name is ${name}|

• 算术运算：
二元运算符：+，-，*，/，%
减号（一元运算符）： -

• 布尔运算：
二元运算符：and，or
布尔否定（一元运算符）： !，not

• 比较和平等：
比较：>，<，>=，<=（gt，lt，ge，le）
平等运营商：==，!=（eq，ne）

• 有条件的运营商：
IF-THEN： (if) ? (then)
IF-THEN-ELSE： (if) ? (then) : (else)
默认： (value) ?: (defaultvalue)

• 特殊代币：
无操作： _


嵌套的案例
'User is of type ' + (${user.isAdmin()} ? 'Administrator' : (${user.type} ?: 'Unknown'))

```





## 三 . 常见用法

### P标签导入数据

```
 <p th:text="#{home.welcome}">Welcome to our grocery store!</p>
```

### h:text 替换文本

```
<p th:text="${x.a}">被替换文本</p> 
```

### 字符串拼接

```
 <p th:text="${'拼接块'+x.a+'拼接块'+x.b+'拼接块'}" >被替换文本</p> 
```




### 遍历List

```
 <tr th:each="x:${xList}"> 
  <th th:text="${x.a}">被替换文本</th> 
  <th th:text="${x.b+' '+x.c}" >被替换文本</th> 
 </tr> 
```





### 格式化日期

```
 th:text="${#dates.format(x.y, 'yyyy-MM-dd HH:mm:ss')}" 
```




### 引入外部文件： 1.@{}的方式引入外部文件会包含项目名

```
 css：  th:href="@{/css/***.css}" 
 js：   th:src="@{/js/***.js}" 
 img：  th:src="@{/img/***.png}" 
 
 
 <link rel="stylesheet" type="text/css" media="all" href="../../css/gtvg.css" th:href="@{/css/gtvg.css}" />
<link rel="stylesheet" type="text/css" media="all" href="../../css/gtvg.css" data-th-href="@{/css/gtvg.css}" />

<script th:src="@{http://libs.baidu.com/jquery/2.1.4/jquery.min.js}"></script>
<script th:src="@{/js/common.js}"></script>
```



### include引入文件 

```
（1）模板文件引入layout，并且声明decorator=“自定义X”
 <html xmlns:th="http://www.thymeleaf.org"  
 xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"  
 layout:decorator="web-model"> 

 （2）模板文件写入内容，th:fragment=“自定义Y”。ps：th:remove=“tag”为了删掉外边包围着的div
 <div th:fragment="left" th:remove="tag">
 
 （3）需要引入的模板文件的html
 <div th:include="web-model（自定义X）::left(自定义Y)" ></div>
```



### 选择表达式 * 号写法

```
语法:
*{...}

案例:
	 <div th:object="${session.user}">
	    <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
	    <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
	    <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
	  </div>

等价于:
	<div>
	  <p>Name: <span th:text="${session.user.firstName}">Sebastian</span>.</p>
	  <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
	  <p>Nationality: <span th:text="${session.user.nationality}">Saturn</span>.</p>
	</div>

与$混合使用
	<div th:object="${session.user}">
	  <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
	  <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
	  <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
	</div>

#object表达式变量
	<div th:object="${session.user}">
	  <p>Name: <span th:text="${#object.firstName}">Sebastian</span>.</p>
	  <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
	  <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
	</div>

当没有执行对象的时候美元和星号是等效
	<div>
	  <p>Name: <span th:text="*{session.user.name}">Sebastian</span>.</p>
	  <p>Surname: <span th:text="*{session.user.surname}">Pepper</span>.</p>
	  <p>Nationality: <span th:text="*{session.user.nationality}">Saturn</span>.</p>
</div>
```

### 表单创建

```java
> 跳转对应页面前 : 注意生成 entity 对象
    @GetMapping("get")
    public ModelAndView getSsoConfigEntity(@RequestParam("key") String key) {
        ModelAndView modelAndView = getModelAndView("");
        modelAndView.addObject("data", jpaRepository.getOne(key));
        modelAndView.addObject("entity", new SsoAppTypeEntity());
        return modelAndView;
    }

> 对应页面  -> 注意 entity
<form th:action="@{/view/type/insert}" th:object="${entity}" th:method="post">
	<input th:type="text" th:field="*{typeCode}" class="form-control" />
	<button class="btn btn-sm btn-primary login-submit-cs" type="submit">保存</button>
</form>
        
> 接收
    @PostMapping("insert")
    public ModelAndView insert(@ModelAttribute("entity") D entity) {
        ModelAndView modelAndView = getModelAndView("list");
        modelAndView.addObject("key", jpaRepository.save(entity));
        return modelAndView;
    }
```





### 常用关键字

```
th:id    替换id      <input th:id="'xxx' + ${collect.id}"/>
th:text    文本替换    <p th:text="${collect.description}">description</p>
th:utext    支持html的文本替换   <p th:utext="${htmlcontent}">conten</p>
th:object    替换对象    <div th:object="${session.user}">
th:value    属性赋值    <input th:value="${user.name}" />
th:with    变量赋值运算    <div th:with="isEven=${prodStat.count}%2==0"></div>
th:style    设置样式    th:style="'display:' + @{(${sitrue} ? 'none' : 'inline-block')} + ''"
th:onclick    点击事件    th:onclick="'getCollect()'"
th:each    属性赋值    tr th:each="user,userStat:${users}">
th:if    判断条件    <a th:if="${userId == collect.userId}" >
th:unless    和th:if判断相反    <a th:href="@{/login}" th:unless=${session.user != null}>Login</a>
th:href    链接地址    <a th:href="@{/login}" th:unless=${session.user != null}>Login</a> />
th:switch    多路选择 配合th:case 使用    <div th:switch="${user.role}">
th:case    th:switch的一个分支    <p th:case="'admin'">User is an administrator</p>
th:fragment    布局标签，定义一个代码片段，方便其它地方引用    <div th:fragment="alert">
th:include    布局标签，替换内容到引入的文件    <head th:include="layout :: htmlhead" th:with="title='xx'"></head> />
th:replace    布局标签，替换整个标签到引入的文件    <div th:replace="fragments/header :: title"></div>
th:selected    selected选择框 选中    th:selected="(${xxx.id} == ${configObj.dd})"
th:src    图片类地址引入    <img class="img-responsive" alt="App Logo" th:src="@{/img/logo.png}" />
th:inline    定义js脚本可以使用变量    <script type="text/javascript" th:inline="javascript">
th:action    表单提交的地址    <form action="subscribe.html" th:action="@{/subscribe}">
th:remove    删除某个属性    <tr th:remove="all"> 

```

### Thymeleaf 和 JS 交互

```html
<!-- 方式一 -->
<!-- 注意有2种方式 : [(${msg})]-->
<div th:fragment="scripts(scripts)">
        <script th:inline="javascript">
            window.applicationMetadataId = [[${applicationMetadataId}]];
            window.searchId = [[${searchId}]];
            window.searchName = [[${searchName}]];
            window.remark = [[${remark}]];
        </script>
</div>

<div th:replace="views/tableauPage :: scripts(~{this :: .custom-script})"></div>


<!-- 其他案例 -->
<script th:inline="JavaScript">  
/*<![CDATA[*/  
  
    var message = [[${message}]];  
    console.log(message);  
  
/*]]>*/  
</script>  
```

### Thymleaf  跳转页面

```
// Button 


// <a>
<a th:href="@{/loginEmail}"></a>
```

### Thymeleaf 提供默认值

```
th:value="${sales!=null?sales.email:''}"
```

### thymeleaf  模板使用

####  th:inssert : 保留当前主标签，保留th:fragment主标签；

```
<footer th:fragment="copy">  
   <script type="text/javascript" th:src="@{/plugins/jquery/jquery-3.0.2.js}"></script>  
</footer> 
---->
<div th:insert="footer :: copy"></div>  
---->
<div>  
    <footer>  
       <script type="text/javascript" th:src="@{/plugins/jquery/jquery-3.0.2.js}"></script>  
    </footer>    
</div>
// 保留了fragment，和DIV 
```
#### th:replace : 舍弃当前主标签，保留th:fragment主标签；

```
<div th:replace="footer :: copy"></div>  
<footer>  
  <script type="text/javascript" th:src="@{/plugins/jquery/jquery-3.0.2.js}"></script>  
</footer>  
//DIV 没有保留
```
####  th:include : 保留当前主标签，舍弃th:fragment主标签。

##### 

```
准备 完整模板 header.html

<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultrag.net.nz/thymeleaf/layout">
<head>
<meta charset="UTF-8">
<title>Thymeleaf in action</title>
</head>
<body>
<div th:fragment="header">
<h1>Thymeleaf in action</h1>
    <a href="/users" >首页</a>
</div>
</body>
</html>

#####  导入<head th:replace="路径 :: 模块名"></head>

 <div th:replace="~{fragments/header::header}"></div>
<head th:replace="../templates/system/index/headLink :: links"></hea

#####  路径以templates 为基准的相对路径 ，具体应该是设置的


<span th:include="pages/fragments/header/head-css::common-css"></span>


```


### thymeleaf模板传值

```

<header th:fragment="header (tab)">
  <ul>
    <li><span th:class="${tab eq 'news'} ? active">news</span></li>
    <li><span th:class="${tab eq 'blog'} ? active">blog</span></li>
    <li><span th:class="${tab eq 'post'} ? active">post</span></li>
  </ul>
</header>

```

### 链接地址

```
#### 链接地址
1 .  绝对网址
- > <a th:href="@{https://www.yiibai.com/thymeleaf/}">
=> <a href="https://www.yiibai.com/thymeleaf/">

2  .  上下文相对URL
- > <a th:href="@{/order/list}">
=> <a href="/applicationpath/order/list">

3  .  服务器上下文 , 不链接到应用查询上下文资源
- >  <a th:href="@{~/billing-app/showDetails.html}">
=>  <a href="/billing-app/showDetails.html">
? - 此处没有加上应用程序路径

4  .  协议相关URL
- > <script th:src="@{//scriptserver.example.net/myscript.js}">...</script>
=> <script src="//scriptserver.example.net/myscript.js">...</script>

5  .  添加参数
- > <a th:href="@{/order/details(id=3)}">
=> <a href="/order/details?id=3">

- > <a th:href="@{/order/details(id=3,action='show_all')}"> === 添加多参数

- > <a th:href="@{/order/{id}/details(id=3,action='show_all')}"> === 使用占位符
=> <a href="/order/3/details?action=show_all">

6  .  网址片段标识符
- > <a th:href="@{/home#all_info(action='show')}">
=> <a href="/home?action=show#all_info">

7  .  URL重写

8  .  URL 使用表达式
- >  <a th:href="@{/order/details(id=${order.id},action=(${user.admin} ? 'show_all' : 'show_public'))}">
```



### Srping MVC 使用

```

#### 标准表达式语法
>  ${...} : 变量表达式。
>  *{...} : 选择表达式。
>  #{...} : 消息 (i18n) 表达式。
>  @{...} : 链接 (URL) 表达式。
>  ~{...} : 片段表达式

#### 表达式基本对象
> 1、#ctx：上下文对象
> 2、#vars：上下文变量
> 3、#locale：上下文语言环境
> 4、#httpServletRequest：（只有在Web上下文）HttpServletRequest对象
> 5、#httpSession:（只有在Web上下文）HttpSession对象。

#### 实用工具对象
 >  dates： java.util的实用方法。对象:日期格式、组件提取等.
 >  calendars：类似于#日期,但对于java.util。日历对象
 >  numbers：格式化数字对象的实用方法。
 >  strings：字符串对象的实用方法:包含startsWith,将/附加等。
 >  objects：实用方法的对象。
 >  bools：布尔评价的实用方法。
 >  arrays：数组的实用方法。
 >  lists：list集合。
 >  sets：set集合。
 >  maps：map集合。
 >  aggregates：实用程序方法用于创建聚集在数组或集合.
 >  messages：实用程序方法获取外部信息内部变量表达式,以同样的方式,因为它们将获得使用# {…}语法
 >  ids：实用程序方法来处理可能重复的id属性(例如,由于迭代)。
```

### Spring 配置

```
#### 配置
> org.thymeleaf.templateresolver.ClassLoaderTemplateResolver : 
 - - > return Thread.currentThread().getContextClassLoader().getResourceAsStream(template);
> org.thymeleaf.templateresolver.FileTemplateResolver
 - - > return new FileInputStream(new File(template));
> org.thymeleaf.templateresolver.UrlTemplateResolver
 - - > return (new URL(template)).openStream();
> org.thymeleaf.templateresolver.StringTemplateResolver
 - - > return new StringReader(templateName);

> 设置前后缀
```
templateResolver.setPrefix("/WEB-INF/templates/");
templateResolver.setSuffix(".html");

templateResolver.addTemplateAlias("adminHome","profiles/admin/home");
templateResolver.setTemplateAliases(aliasesMap);

templateResolver.setEncoding("UTF-8");

templateResolver.setCacheable(false);
templateResolver.getCacheablePatternSpec().addPattern("/users/*");

templateResolver.setCacheTTLMs(60000L);
```
#### JavaBean 使用
> 使用Bean 的值
- > model.addAttribute("product", product);
- > <dd th:text="${product.description}">
```
  <div th:object="*{customer}">
    <p><b>Name:</b> <span th:text="*{name}">Frederic Tomato</span></p>
    ...
  </div>
```

```

### 前端使用

```
#### 前端对象
> #ctx : context 容器对象 ( org.thymeleaf.context.IContext 或者 org.thymeleaf.context.IWebContext )
  - - > ${#ctx.locale}
  - - > ${#ctx.variableNames}
  - - > ${#ctx.request}
  - - > ${#ctx.response}
  - - > ${#ctx.session}
  - - > ${#ctx.servletContext}

> #request : 获取请求对象 ( 同理session , context )
  - - > ${#request.getAttribute('foo')}
  - - > ${#request.getParameter('foo')}
  - - > ${#request.getContextPath()}
  - - > ${#request.getRequestName()}
  - - > ${#session.getAttribute('foo')}
  - - > ${#session.id}
  - - > ${#session.lastAccessedTime}
  - - > ${#servletContext.getAttribute('foo')}
  - - > ${#servletContext.contextPath}

> param : 请求参数 , 获取param 中的参数
  - - > ${param.foo}           
  - - > ${param.size()}
  - - > ${param.isEmpty()}
  - - > ${param.containsKey('foo')}

> session : 获取 Session 域参数
  - - > ${session.foo}               
  - - > ${session.size()}
  - - > ${session.isEmpty()}
  - - > ${session.containsKey('foo')}

> application : 获取Application参数
  - - > ${application.foo}             
  - - > ${application.size()}
  - - > ${application.isEmpty()}
  - - > ${application.containsKey('foo')}

> #execInfo : 表达式对象 , 用于返回模板的有用信息
  - - > ${#execInfo.templateName} === 注意插入模板返回的是插入的模板名
  - - > ${#execInfo.templateMode} 
  - - > ${#execInfo.processedTemplateName}
  - - > ${#execInfo.processedTemplateMode}
  - - > ${#execInfo.templateNames}
  - - > ${#execInfo.templateModes}
  - - > ${#execInfo.templateStack}


> #messages : 用于获取外部消息
  - - > ${#messages.msg('msgKey')}
  - - > ${#messages.msg('msgKey', param1)}
  - - > ${#messages.msg('msgKey', param1, param2)}
  - - > ${#messages.msg('msgKey', param1, param2, param3)}
  - - > ${#messages.msgWithParams('msgKey', new Object[] {param1, param2, param3, param4})}
  - - > ${#messages.arrayMsg(messageKeyArray)}
  - - > ${#messages.listMsg(messageKeyList)}
  - - > ${#messages.setMsg(messageKeySet)}
  - - > ${#messages.msgOrNull('msgKey')}
  - - > ${#messages.msgOrNull('msgKey', param1)}
  - - > ${#messages.msgOrNull('msgKey', param1, param2)}
  - - > ${#messages.msgOrNull('msgKey', param1, param2, param3)}
  - - > ${#messages.msgOrNullWithParams('msgKey', new Object[] {param1, param2, param3, param4})}
  - - > ${#messages.arrayMsgOrNull(messageKeyArray)}
  - - > ${#messages.listMsgOrNull(messageKeyList)}
  - - > ${#messages.setMsgOrNull(messageKeySet)}

> #uris : 用于在thymeleaf 中执行url 操作
  - - > ${#uris.escapePath(uri)}
  - - > ${#uris.escapePath(uri, encoding)}
  - - > ${#uris.unescapePath(uri)}
  - - > ${#uris.unescapePath(uri, encoding)}
  - - > ${#uris.escapePathSegment(uri)}
  - - > ${#uris.escapePathSegment(uri, encoding)}
  - - > ${#uris.unescapePathSegment(uri)}
  - - > ${#uris.unescapePathSegment(uri, encoding)}
  - - > ${#uris.escapeFragmentId(uri)}
  - - > ${#uris.escapeFragmentId(uri, encoding)}
  - - > ${#uris.unescapeFragmentId(uri)}
  - - > ${#uris.unescapeFragmentId(uri, encoding)}
  - - > ${#uris.escapeQueryParam(uri)}
  - - > ${#uris.escapeQueryParam(uri, encoding)}
  - - > ${#uris.unescapeQueryParam(uri)}
  - - > ${#uris.unescapeQueryParam(uri, encoding)}

> #conversions : 用于将对象转换为指定的类型
  - - > ${#conversions.convert(object, 'java.util.TimeZone')}
  - - > ${#conversions.convert(object, targetClass)}

> #dates : java.util.Date 工具类
  - - > ${#dates.format(date)}
  - - > ${#dates.arrayFormat(datesArray)}
  - - > ${#dates.listFormat(datesList)}
  - - > ${#dates.setFormat(datesSet)}
  - - > ${#dates.formatISO(date)}
  - - > ${#dates.arrayFormatISO(datesArray)}
  - - > ${#dates.listFormatISO(datesList)}
  - - > ${#dates.setFormatISO(datesSet)}
  - - > ${#dates.format(date, 'dd/MMM/yyyy HH:mm')}
  - - > ${#dates.arrayFormat(datesArray, 'dd/MMM/yyyy HH:mm')}
  - - > ${#dates.listFormat(datesList, 'dd/MMM/yyyy HH:mm')}
  - - > ${#dates.setFormat(datesSet, 'dd/MMM/yyyy HH:mm')}
  - - > ${#dates.day(date)}                    // also arrayDay(...), listDay(...), etc.
  - - > ${#dates.month(date)}                  // also arrayMonth(...), listMonth(...), etc.
  - - > ${#dates.monthName(date)}              // also arrayMonthName(...), listMonthName(...), etc.
  - - > ${#dates.monthNameShort(date)}         // also arrayMonthNameShort(...), listMonthNameShort(...), etc.
  - - > ${#dates.year(date)}                   // also arrayYear(...), listYear(...), etc.
  - - > ${#dates.dayOfWeek(date)}              // also arrayDayOfWeek(...), listDayOfWeek(...), etc.
  - - > ${#dates.dayOfWeekName(date)}          // also arrayDayOfWeekName(...), listDayOfWeekName(...), etc.
  - - > ${#dates.dayOfWeekNameShort(date)}     // also arrayDayOfWeekNameShort(...), listDayOfWeekNameShort(...), etc.
  - - > ${#dates.hour(date)}                   // also arrayHour(...), listHour(...), etc.
  - - > ${#dates.minute(date)}                 // also arrayMinute(...), listMinute(...), etc.
  - - > ${#dates.second(date)}                 // also arraySecond(...), listSecond(...), etc.
  - - > ${#dates.millisecond(date)}            // also arrayMillisecond(...), listMillisecond(...), etc.
  - - > ${#dates.create(year,month,day)}
  - - > ${#dates.create(year,month,day,hour,minute)}
  - - > ${#dates.create(year,month,day,hour,minute,second)}
  - - > ${#dates.create(year,month,day,hour,minute,second,millisecond)}
  - - > ${#dates.createNow()}
  - - > ${#dates.createNowForTimeZone()}
  - - > ${#dates.createToday()}
  - - > ${#dates.createTodayForTimeZone()}

> #Calendars java.util.Calendar 对象 ,区别于date对象
  - - > ${#calendars.format(cal)}
  - - > ${#calendars.arrayFormat(calArray)}
  - - > ${#calendars.listFormat(calList)}
  - - > ${#calendars.setFormat(calSet)}
  - - > ${#calendars.formatISO(cal)}
  - - > ${#calendars.arrayFormatISO(calArray)}
  - - > ${#calendars.listFormatISO(calList)}
  - - > ${#calendars.setFormatISO(calSet)}
  - - > ${#calendars.format(cal, 'dd/MMM/yyyy HH:mm')}
  - - > ${#calendars.arrayFormat(calArray, 'dd/MMM/yyyy HH:mm')}
  - - > ${#calendars.listFormat(calList, 'dd/MMM/yyyy HH:mm')}
  - - > ${#calendars.setFormat(calSet, 'dd/MMM/yyyy HH:mm')}
  - - > ${#calendars.day(date)}                // also arrayDay(...), listDay(...), etc.
  - - > ${#calendars.month(date)}              // also arrayMonth(...), listMonth(...), etc.
  - - > ${#calendars.monthName(date)}          // also arrayMonthName(...), listMonthName(...), etc.
  - - > ${#calendars.monthNameShort(date)}     // also arrayMonthNameShort(...), listMonthNameShort(...), etc.
  - - > ${#calendars.year(date)}               // also arrayYear(...), listYear(...), etc.
  - - > ${#calendars.dayOfWeek(date)}          // also arrayDayOfWeek(...), listDayOfWeek(...), etc.
  - - > ${#calendars.dayOfWeekName(date)}      // also arrayDayOfWeekName(...), listDayOfWeekName(...), etc.
  - - > ${#calendars.dayOfWeekNameShort(date)} // also arrayDayOfWeekNameShort(...), listDayOfWeekNameShort(...), etc.
  - - > ${#calendars.hour(date)}               // also arrayHour(...), listHour(...), etc.
  - - > ${#calendars.minute(date)}             // also arrayMinute(...), listMinute(...), etc.
  - - > ${#calendars.second(date)}             // also arraySecond(...), listSecond(...), etc.
  - - > ${#calendars.millisecond(date)}        // also arrayMillisecond(...), listMillisecond(...), etc.
  - - > ${#calendars.create(year,month,day)}
  - - > ${#calendars.create(year,month,day,hour,minute)}
  - - > ${#calendars.create(year,month,day,hour,minute,second)}
  - - > ${#calendars.create(year,month,day,hour,minute,second,millisecond)}
  - - > ${#calendars.createForTimeZone(year,month,day,timeZone)}
  - - > ${#calendars.createForTimeZone(year,month,day,hour,minute,timeZone)}
  - - > ${#calendars.createForTimeZone(year,month,day,hour,minute,second,timeZone)}
  - - > ${#calendars.createForTimeZone(year,month,day,hour,minute,second,millisecond,timeZone)}
  - - > ${#calendars.createNow()}
  - - > ${#calendars.createNowForTimeZone()}
  - - > ${#calendars.createToday()}
  - - > ${#calendars.createTodayForTimeZone()}

>  #Numbers
  - - > ${#numbers.formatInteger(num,3)}
  - - > ${#numbers.arrayFormatInteger(numArray,3)}
  - - > ${#numbers.listFormatInteger(numList,3)}
  - - > ${#numbers.setFormatInteger(numSet,3)}
  - - > 
  - - > ${#numbers.formatInteger(num,3,'POINT')}
  - - > ${#numbers.arrayFormatInteger(numArray,3,'POINT')}
  - - > ${#numbers.listFormatInteger(numList,3,'POINT')}
  - - > ${#numbers.setFormatInteger(numSet,3,'POINT')}
  - - > 
  - - > ${#numbers.formatDecimal(num,3,2)}
  - - > ${#numbers.arrayFormatDecimal(numArray,3,2)}
  - - > ${#numbers.listFormatDecimal(numList,3,2)}
  - - > ${#numbers.setFormatDecimal(numSet,3,2)}
  - - > 
  - - > ${#numbers.formatDecimal(num,3,2,'COMMA')}
  - - > ${#numbers.arrayFormatDecimal(numArray,3,2,'COMMA')}
  - - > ${#numbers.listFormatDecimal(numList,3,2,'COMMA')}
  - - > ${#numbers.setFormatDecimal(numSet,3,2,'COMMA')}
  - - > 
  - - > ${#numbers.formatDecimal(num,3,'POINT',2,'COMMA')}
  - - > ${#numbers.arrayFormatDecimal(numArray,3,'POINT',2,'COMMA')}
  - - > ${#numbers.listFormatDecimal(numList,3,'POINT',2,'COMMA')}
  - - > ${#numbers.setFormatDecimal(numSet,3,'POINT',2,'COMMA')}
  - - > 
  - - > ${#numbers.formatCurrency(num)}
  - - > ${#numbers.arrayFormatCurrency(numArray)}
  - - > ${#numbers.listFormatCurrency(numList)}
  - - > ${#numbers.setFormatCurrency(numSet)}
  - - > 
  - - > ${#numbers.formatPercent(num)}
  - - > ${#numbers.arrayFormatPercent(numArray)}
  - - > ${#numbers.listFormatPercent(numList)}
  - - > ${#numbers.setFormatPercent(numSet)}
  - - > 
  - - > ${#numbers.formatPercent(num, 3, 2)}
  - - > ${#numbers.arrayFormatPercent(numArray, 3, 2)}
  - - > ${#numbers.listFormatPercent(numList, 3, 2)}
  - - > ${#numbers.setFormatPercent(numSet, 3, 2)}
  - - > 
  - - > ${#numbers.sequence(from,to)}
  - - > ${#numbers.sequence(from,to,step)}

> Strings : StringObject 
  - - > ${#strings.toString(obj)}                           // also array*, list* and set*
  - - > 
  - - > ${#strings.isEmpty(name)}
  - - > ${#strings.arrayIsEmpty(nameArr)}
  - - > ${#strings.listIsEmpty(nameList)}
  - - > ${#strings.setIsEmpty(nameSet)}
  - - > 
  - - > ${#strings.defaultString(text,default)}
  - - > ${#strings.arrayDefaultString(textArr,default)}
  - - > ${#strings.listDefaultString(textList,default)}
  - - > ${#strings.setDefaultString(textSet,default)}
  - - > 
  - - > ${#strings.contains(name,'ez')}                     // also array*, list* and set*
  - - > ${#strings.containsIgnoreCase(name,'ez')}           // also array*, list* and set*
  - - > 
  - - > ${#strings.startsWith(name,'Don')}                  // also array*, list* and set*
  - - > ${#strings.endsWith(name,endingFragment)}           // also array*, list* and set*
  - - > 
  - - > ${#strings.indexOf(name,frag)}                      // also array*, list* and set*
  - - > ${#strings.substring(name,3,5)}                     // also array*, list* and set*
  - - > ${#strings.substringAfter(name,prefix)}             // also array*, list* and set*
  - - > ${#strings.substringBefore(name,suffix)}            // also array*, list* and set*
  - - > ${#strings.replace(name,'las','ler')}               // also array*, list* and set*
  - - > 
  - - > ${#strings.prepend(str,prefix)}                     // also array*, list* and set*
  - - > ${#strings.append(str,suffix)}                      // also array*, list* and set*
  - - > 
  - - > ${#strings.toUpperCase(name)}                       // also array*, list* and set*
  - - > ${#strings.toLowerCase(name)}                       // also array*, list* and set*
  - - > 
  - - > ${#strings.arrayJoin(namesArray,',')}
  - - > ${#strings.listJoin(namesList,',')}
  - - > ${#strings.setJoin(namesSet,',')}
  - - > ${#strings.arraySplit(namesStr,',')}                // returns String[]
  - - > ${#strings.listSplit(namesStr,',')}                 // returns List<String>
  - - > ${#strings.setSplit(namesStr,',')}                  // returns Set<String>
  - - > 
  - - > ${#strings.trim(str)}                               // also array*, list* and set*
  - - > 
  - - > ${#strings.length(str)}                             // also array*, list* and set*
  - - > 
  - - > ${#strings.abbreviate(str,10)}                      // also array*, list* and set*
  - - > 
  - - > ${#strings.capitalize(str)}                         // also array*, list* and set*
  - - > ${#strings.unCapitalize(str)}                       // also array*, list* and set*
  - - > 
  - - > ${#strings.capitalizeWords(str)}                    // also array*, list* and set*
  - - > ${#strings.capitalizeWords(str,delimiters)}         // also array*, list* and set*
  - - > 
  - - > ${#strings.escapeXml(str)}                          // also array*, list* and set*
  - - > ${#strings.escapeJava(str)}                         // also array*, list* and set*
  - - > ${#strings.escapeJavaScript(str)}                   // also array*, list* and set*
  - - > ${#strings.unescapeJava(str)}                       // also array*, list* and set*
  - - > ${#strings.unescapeJavaScript(str)}                 // also array*, list* and set*
  - - > 
  - - > ${#strings.equals(first, second)}
  - - > ${#strings.equalsIgnoreCase(first, second)}
  - - > ${#strings.concat(values...)}
  - - > ${#strings.concatReplaceNulls(nullValue, values...)}
  - - > 
  - - > ${#strings.randomAlphanumeric(count)}

>  Objects : object 对象
  - - > ${#objects.nullSafe(obj,default)}
  - - > ${#objects.arrayNullSafe(objArray,default)}
  - - > ${#objects.listNullSafe(objList,default)}
  - - > ${#objects.setNullSafe(objSet,default)}

> Booleans : #bools 对象
  - - > ${#bools.isTrue(obj)}
  - - > ${#bools.arrayIsTrue(objArray)}
  - - > ${#bools.listIsTrue(objList)}
  - - > ${#bools.setIsTrue(objSet)}
  - - > 
  - - > ${#bools.isFalse(cond)}
  - - > ${#bools.arrayIsFalse(condArray)}
  - - > ${#bools.listIsFalse(condList)}
  - - > ${#bools.setIsFalse(condSet)}
  - - > 
  - - > ${#bools.arrayAnd(condArray)}
  - - > ${#bools.listAnd(condList)}
  - - > ${#bools.setAnd(condSet)}
  - - > 
  - - > ${#bools.arrayOr(condArray)}
  - - > ${#bools.listOr(condList)}
  - - > ${#bools.setOr(condSet)}


> Arrays 
  - - > ${#arrays.toArray(object)}
  - - > 
  - - > ${#arrays.toStringArray(object)}
  - - > ${#arrays.toIntegerArray(object)}
  - - > ${#arrays.toLongArray(object)}
  - - > ${#arrays.toDoubleArray(object)}
  - - > ${#arrays.toFloatArray(object)}
  - - > ${#arrays.toBooleanArray(object)}
  - - > 
  - - > ${#arrays.length(array)}
  - - > 
  - - > ${#arrays.isEmpty(array)}
  - - > 
  - - > ${#arrays.contains(array, element)}
  - - > ${#arrays.containsAll(array, elements)}

> Lists :
  - - > ${#lists.toList(object)}
  - - > 
  - - > ${#lists.size(list)}
  - - > 
  - - > ${#lists.isEmpty(list)}
  - - > 
  - - > ${#lists.contains(list, element)}
  - - > ${#lists.containsAll(list, elements)}
  - - > 
  - - > ${#lists.sort(list)}
  - - > ${#lists.sort(list, comparator)}

> Sets :
  - - > ${#sets.toSet(object)}
  - - > 
  - - > ${#sets.size(set)}
  - - > 
  - - > ${#sets.isEmpty(set)}
  - - > 
  - - > ${#sets.contains(set, element)}
  - - > ${#sets.containsAll(set, elements)}

> Maps : 
  - - > ${#maps.size(map)}
  - - > 
  - - > ${#maps.isEmpty(map)}
  - - > 
  - - > ${#maps.containsKey(map, key)}
  - - > ${#maps.containsAllKeys(map, keys)}
  - - > ${#maps.containsValue(map, value)}
  - - > ${#maps.containsAllValues(map, value)}

> Aggregates : 用于数组统计
  - - > ${#aggregates.sum(array)}
  - - > ${#aggregates.sum(collection)}
  - - > ${#aggregates.avg(array)}
  - - > ${#aggregates.avg(collection)}

```

### 常用

```
#### 常用
1 > input 
- >  <input type="text" name="userName" value="Yiibai" th:value="${user.name}" />


2 > each 循环

<tr th:each="user:${userModel.userList}">
     <td th:text="${user.id}"></td>
      <td th:text="${user.email}"></td>
 </tr>


3 > if 判断
? - - if 判断通过则会生成该代码

<tr th:if="${userModel.userList.size()} eq 0">


4 > 文本

<span th:text="${recipient}"></span>



5 > form 表单提交

<form action="/users" th:action="@{/users}" method="POST" th:object="${userModel.user}">
    <input type="text" name="name" th:value="*{name}">
    <input type="submit" value="提交" >
</form>


6  > 格式输出

//小数格式化
<dd th:text="${'￥' + #numbers.formatDecimal(product.price, 1, 2)}">100</dd>

//日期格式化
<dd th:text="${#dates.format(product.availableFrom, 'yyyy-MM-dd')}">28-Jun-2018</dd>


7 > 字符串转义

<div th:utext="${html}">Some unescaped text</div>
//当后台传递的参数是一段HTML代码的时候 , 通过该方法进行字符串转义



8 > 显示 bean 参数

<div th:object="${session.user}">
  <p>Name: <span th:text="${#object.firstName}">Sebastian</span>.</p>
  <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
  <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
</div>


```



## 问题异常

### Error resolving template [test], template might not exist or might not be accessible by any of the configured Template Resolvers

```
> 1 先打包 , 检查是否成功打包进去
	查看打包路径 \target\classes 是否存在 template 文件夹
	
> 2 打断点 : 判断路径是否正确
	抛出异常的地方开始 , 向下可以看到路径
	
	

```

### 打成jar 包找不到模板 view

```java
// 单项目
加配置 

// 聚合项目
<plugin>
                <artifactId>maven-resources-plugin</artifactId>
                <version>3.0.2</version>
                <executions>
                    <execution>
                        <id>copy-resources</id>
                        <phase>process-sources</phase>
                        <goals>
                            <goal>copy-resources</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>${basedir}/target/classes/</outputDirectory>
                            <resources>
                                <!-- 将app.pms.cust.api中的配置文件复制到该宿主工程中 -->
                                <resource>
                                    <directory>${basedir}/../antsso-web/src/main/resources</directory>
                                    <includes>
                                        <include>static/**</include>
                                        <include>template/**</include>
                                    </includes>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```