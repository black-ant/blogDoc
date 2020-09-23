# SpringSecurity 文档

[TOC]



## 一 . 前言

OAuth 2 是一个授权框架，或称授权标准，它可以使第三方应用程序或客户端获得对HTTP服务上（例如 Google，GitHub ）用户帐户信息的有限访问权限。OAuth 2 通过将用户身份验证委派给托管用户帐户的服务以及授权客户端访问用户帐户进行工作。综上，OAuth 2 可以为 Web 应用 和桌面应用以及移动应用提供授权流程。

## 二 . OAuth2 角色

OAuth 2 标准中定义了以下几种角色：

- 资源所有者（Resource     Owner）
- 资源服务器（Resource     Server）
- 授权服务器（Authorization     Server）
- 客户端（Client）

```makefile
## 2.1 资源所有者（Resource Owner）

资源所有者是 OAuth 2 四大基本角色之一，在 OAuth 2 标准中，资源所有者即代表授权客户端访问本身资源信息的用户（User），也就是应用场景中的“开发者A”。客户端访问用户帐户的权限仅限于用户授权的“范围”（aka. scope，例如读取或写入权限）。

如果没有特别说明，下文中出现的"用户"将统一代表资源所有者。

## 2.2 资源/授权服务器（Resource/Authorization Server）

资源服务器托管了受保护的用户账号信息，而授权服务器验证用户身份然后为客户端派发资源访问令牌。

在上述应用场景中，Github 既是授权服务器也是资源服务器，个人信息和仓库信息即为资源（Resource）。而在实际工程中，不同的服务器应用往往独立部署，协同保护用户账户信息资源。

## 2.3 客户端（Client）

在 OAuth 2 中，客户端即代表意图访问受限资源的第三方应用。在访问实现之前，它必须先经过用户者授权，并且获得的授权凭证将进一步由授权服务器进行验证。

如果没有特别说明，下文中将不对"应用"，“第三方应用”，“客户端”做出区分。
```





## 三 . OAuth 2 的授权流程

目前为止你应该对 OAuth 2 的角色有了些概念，接下来让我们来看看这几个角色之间的抽象授权流程图和相关解释。

请注意，实际的授权流程图会因为用户返回授权许可类型的不同而不同。但是下图大体上能反映一次完整抽象的授权流程。

![Abstract](file:///C:\Users\10169\OneDrive\笔记文档\图片文件\springsecurity/clip_image001.png)

```
1. Authrization Request  -> 客户端向用户请求对资源服务器的authorization grant。
2. Authorization Grant（Get） -> 如果用户授权该次请求，客户端将收到一个authorization grant。
3. Authorization Grant（Post） 
	-> 客户端向授权服务器发送它自己的客户端身份标识和上一步中的authorization grant，请求访问令牌。
4. Access Token（Get） 
	-> 如果客户端身份被认证，并且authorization grant也被验证通过，授权服务器将为客户端派发accesstoken。
	-> 授权阶段至此全部结束。
5. Access Token（Post && Validate） 
	-> 客户端向资源服务器发送access token用于验证并请求资源信息。
6. Protected Resource（Get） 
	-> 如果access token验证通过，资源服务器将向客户端返回资源信息。
```



## 四 .  管理源码解析

### 4 . 1 AuthenticationManager 核心管理接口

```
1. 如果认证成功，返回 Authentication (通常它的authenticated属性为true authenticated=true) .
2. 如果认证失败，抛出 AuthenticationException 异常.
3. 如果无法判断，返回 null .
```

### 4 . 2 AbstractAuthenticationProcessingFilter

```

```

### 4 . 3 AbstractUserDetailsAuthenticationProvider

```
用于对 UserDetails 进行校验
M- additionalAuthenticationChecks
M- afterPropertiesSet
M- authenticate : 认证


```

### 4 . 4 AbstractAuthenticationProcessingFilter

```
> 对所有的请求进行过滤 , 判断是否需要进行处理 

M- doFilter
	-> requiresAuthentication(request, response) : 所有非认证的请求 , 均会在这个里面判断后进行跳转
	-> Authentication  authResult = attemptAuthentication(request, response); 进行认证并且返回结果
		-> username / password / roles 
M- successfulAuthentication : 对成功的认证进行记录
	-> 
```

### 4 . 5  HttpSessionRequestCache

```
> 保存拦截前访问的request 对象
```

### 4 . ? SavedRequestAwareAuthenticationSuccessHandler

```
> 认证成功的重定向
M- onAuthenticationSuccess
	-> 如果判断没有 Request 信息 , 会被清空所有认证信息!认证失败可能是这里的原因
	getRedirectStrategy().sendRedirect(request, response, target)
		-> 获取重定向策略并且进行重定向
```

### 4 . ? DefaultRedirectStrategy : 默认重定向策略

```

```

### 4 . ? RequestCacheAwareFilter

```
> 对请求进行拦截 , 并且缓存请求内容
M- doFilter
	-> requestCache.getMatchingRequest((HttpServletRequest) request, (HttpServletResponse) response)
		-> request匹配，则取出，该操作同时会将缓存的request从session中删除
	->  chain.doFilter(wrappedSavedRequest == null ? request : wrappedSavedRequest,response)
		-> 使用缓存的 Request
		
		
```

### 4 . ? DefaultOAuth2RequestFactory : 生成 OAuth2 请求

```

```

## 五 . 工具源码解析



## 六 .  通用类源码解析



## 七 . 协议源码解析



## 八 . 其他类

### CsrfFilter :  跨域 Token 处理

```

```

### HeaderWriterFilter : 对 Header 头进行重写

```

```

### OnCommittedResponseWrapper : 对 Response 进行处理

```

```

###  org.apache.catalina.connector.Response

```
> 此类中进行了重定向的处理 , 核心代码为 
M- sendRedirect
	- getContext().getUseRelativeRedirects() : 是否进行相对位置重定向
```

### Interface RequestCache 

```java
// 将request缓存到session中
void saveRequest(HttpServletRequest request, HttpServletResponse response);
    
// 从session中取request
SavedRequest getRequest(HttpServletRequest request, HttpServletResponse response);
    
// 获得与当前request匹配的缓存，并将匹配的request从session中删除
HttpServletRequest getMatchingRequest(HttpServletRequest request,HttpServletResponse response);
    
// 删除缓存的request
void removeRequest(HttpServletRequest request, HttpServletResponse response);

```

### ExceptionTranslationFilter

```
> 处理AuthenticationException和AccessDeniedException两种异常

AuthenticationException : 为登录状态下访问受保护的资源

AccessDeniedException : 登录了但是权限不足

> ExceptionTranslationFilter 持有两个处理类，分别是AuthenticationEntryPoint和AccessDeniedHandler。
规则1. 如果异常是 AuthenticationException，使用 AuthenticationEntryPoint 处理
规则2. 如果异常是 AccessDeniedException 且用户是匿名用户，使用 AuthenticationEntryPoint 处理
规则3. 如果异常是 AccessDeniedException 且用户不是匿名用户，如果否则交给 AccessDeniedHandler 处理。

> AccessDeniedHandlerImpl :对异常的处理是返回403错误码
	I- AccessDeniedHandler 
	
> LoginUrlAuthenticationEntryPoint : 该类的处理是转发或重定向到登录页面
	I- AuthenticationEntryPoint 
```

### 5 . 10 常见Filter

| **别名**                     | **类名称**                                             | **Namespace Element or Attribute**                           |
| ---------------------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| CHANNEL_FILTER               | ChannelProcessingFilter                                | http/intercept-url[@requires](https://github.com/requires)-channel |
| SECURITY_CONTEXT_FILTER      | SecurityContextPersistenceFilter                       | http                                                         |
| CONCURRENT_SESSION_FILTER    | ConcurrentSessionFilter                                | session-management/concurrency-control                       |
| HEADERS_FILTER               | HeaderWriterFilter                                     | http/headers                                                 |
| CSRF_FILTER                  | CsrfFilter                                             | http/csrf                                                    |
| LOGOUT_FILTER                | LogoutFilter                                           | http/logout                                                  |
| X509_FILTER                  | X509AuthenticationFilter                               | http/x509                                                    |
| PRE_AUTH_FILTER              | AbstractPreAuthenticatedProcessingFilter(  Subclasses) | N/A                                                          |
| CAS_FILTER                   | CasAuthenticationFilter                                | N/A                                                          |
| FORM_LOGIN_FILTER            | UsernamePasswordAuthenticationFilter                   | http/form-login                                              |
| BASIC_AUTH_FILTER            | BasicAuthenticationFilter                              | http/http-basic                                              |
| SERVLET_API_SUPPORT_FILTER   | SecurityContextHolderAwareRequestFilter                | http/[@servlet](https://github.com/servlet)-api-provision    |
| JAAS_API_SUPPORT_FILTER      | JaasApiIntegrationFilter                               | http/[@jaas](https://github.com/jaas)-api-provision          |
| REMEMBER_ME_FILTER           | RememberMeAuthenticationFilter                         | http/remember-me                                             |
| ANONYMOUS_FILTER             | AnonymousAuthenticationFilter                          | http/anonymous                                               |
| SESSION_MANAGEMENT_FILTER    | SessionManagementFilter                                | session-management                                           |
| EXCEPTION_TRANSLATION_FILTER | ExceptionTranslationFilter                             | http                                                         |
| FILTER_SECURITY_INTERCEPTOR  | FilterSecurityInterceptor                              | http                                                         |
| SWITCH_USER_FILTER           | SwitchUserFilter                                       | N/A                                                          |

```
> Filter 的使用方式

• addFilterBefore(Filter filter, Class<? extends Filter> beforeFilter)
	?- 在 beforeFilter 之前添加 filter

• addFilterAfter(Filter filter, Class<? extends Filter> afterFilter)
	?- 在 afterFilter 之后添加 filter

• addFilterAt(Filter filter, Class<? extends Filter> atFilter)
	?- 在 atFilter 相同位置添加 filter， 此 filter 不覆盖 filter

```



## 六 常用操作

### 6 . 1 自定义一个认证流程

```
1. 增加一个token (EmailCodeAuthenticationToken)
2. 增加一个provider (EmailCodeAuthenticationProvider)
3. 增加一个filter (EmailCodeAuthenticationProcessingFilter)
4. 在SecurityConfiguration中把三个项加入相应的位置



```

### 6 . 2  **HttpSecurity** 常用方法

| **方法**            | **说明**                                                     |
| ------------------- | ------------------------------------------------------------ |
| openidLogin()       | 用于基于  OpenId 的验证                                      |
| headers()           | 将安全标头添加到响应                                         |
| cors()              | 配置跨域资源共享（  CORS ）                                  |
| sessionManagement() | 允许配置会话管理                                             |
| portMapper()        | 允许配置一个PortMapper(HttpSecurity#(getSharedObject(class)))，其他提供SecurityConfigurer的对象使用PortMapper 从 HTTP 重定向到 HTTPS 或者从 HTTPS 重定向到  HTTP。默认情况下，Spring Security使用一个PortMapperImpl映射 HTTP 端口8080到 HTTPS 端口8443，HTTP 端口80到 HTTPS 端口443 |
| jee()               | 配置基于容器的预认证。  在这种情况下，认证由Servlet容器管理  |
| x509()              | 配置基于x509的认证                                           |
| rememberMe          | 允许配置“记住我”的验证                                       |
| authorizeRequests() | 允许基于使用HttpServletRequest限制访问                       |
| requestCache()      | 允许配置请求缓存                                             |
| exceptionHandling() | 允许配置错误处理                                             |
| securityContext()   | 在HttpServletRequests之间的SecurityContextHolder上设置SecurityContext的管理。 当使用WebSecurityConfigurerAdapter时，这将自动应用 |
| servletApi()        | 将HttpServletRequest方法与在其上找到的值集成到SecurityContext中。 当使用WebSecurityConfigurerAdapter时，这将自动应用 |
| csrf()              | 添加 CSRF 支持，使用WebSecurityConfigurerAdapter时，默认启用 |
| logout()            | 添加退出登录支持。当使用WebSecurityConfigurerAdapter时，这将自动应用。默认情况是，访问URL”/ logout”，使HTTP  Session无效来清除用户，清除已配置的任何#rememberMe()身份验证，清除SecurityContextHolder，然后重定向到”/login?success” |
| anonymous()         | 允许配置匿名用户的表示方法。 当与WebSecurityConfigurerAdapter结合使用时，这将自动应用。 默认情况下，匿名用户将使用org.springframework.security.authentication.AnonymousAuthenticationToken表示，并包含角色 “ROLE_ANONYMOUS” |
| formLogin()         | 指定支持基于表单的身份验证。如果未指定FormLoginConfigurer#loginPage(String)，则将生成默认登录页面 |
| oauth2Login()       | 根据外部OAuth  2.0或OpenID Connect 1.0提供程序配置身份验证   |
| requiresChannel()   | 配置通道安全。为了使该配置有用，必须提供至少一个到所需信道的映射 |
| httpBasic()         | 配置 Http  Basic 验证                                        |
| addFilterAt()       | 在指定的Filter类的位置添加过滤器                             |

### 6 .  3  一个完整的认证流程

```java
> 1 AbstractAuthenticationProcessFilter.doFilter()
	?- 对所有请求进行拦截 , 判断是否需要认证
> 2 xxxxxAuthenticationFilter
    ?- 判断路径进行单一认证Filter
    - create xxxxAuthenticationToken
> 3 ProvideManager.authnticate
    ?- 判断用什么Provide 进行处理
> 4 xxxxAuthenticationProvide
    ?- 进行用户判断和比对
    ?- 必要情况下生成新 Token 对象
> 5 UserDetailsService
    ?- 调用 Service 层获取用户信息
    
```

### 6 . 4 加密流程的使用

```java
// Step 1 : 对注册密码进行加密
// 可以理解为保存到数据库的数据密码即为加密的


// Step 2 : 配置Config
	/**
     * 添加 UserDetailsService， 实现自定义登录校验
     */
    @Override
    protected void configure(AuthenticationManagerBuilder builder) throws Exception{
        builder.userDetailsService(anyUserDetailsService)
                .passwordEncoder(passwordEncoder());
    }
	/**
     * 密码加密
     */
    @Bean
    public BCryptPasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }


// 密码加密其实很好理解 ,对比的时候通过同样的方式加密是最简单的做法 : 
// 可以扩展 : 
```

### 6 .5  一个绑定流程

```java
自定义一个 Security 的认证流程有几个组成部分
1 Filter : extends AbstractAuthenticationProcessingFilter
	?- 在 Filter 中 , 会过滤指定的地址 , 并且生成对应的Token

2 Token :  extends AbstractAuthenticationToken
	?- Token 是在其中流转的信息媒介
	
3 provide : extends AuthenticationProvider
	?- 重写 supports 方法 , 用于表示是否可以处理
	?- 重写 authenticate 方法 , 用于认证
	
4 WebSecurityConfig 中进行绑定
protected void configure(AuthenticationManagerBuilder auth)  中绑定 provide
 protected void configure(HttpSecurity http) 中绑定 filter
 
 
 // 附录
// 整体处理: ProviderManager
    
```



## 问题 : 

#### Security 重定向到相对地址

```java
> 主要涉及类 : org.apache.catalina.connector.Response

if (getRequest().getCoyoteRequest().getSupportsRelativeRedirects() &&
                    getContext().getUseRelativeRedirects()) {
     locationUri = location;
} else {
     locationUri = toAbsolute(location);
}

// 如果 Client Redirect 为 Http:// 起头的 ,则正常跳转

> toAbsolute
    > location.startsWith("//")
    > (scheme.equals("http") && port != 80)
```

#### security 通过code 获取 token 返回 401

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security
                .tokenKeyAccess("permitAll()")
                .checkTokenAccess("permitAll()")
                .allowFormAuthenticationForClients();

    }

}

```

#### Returns the authorities granted to the user. Cannot return 

```java
    public ArrayList<GrantedAuthority> getUserRolesByType() {
        ArrayList<GrantedAuthority> authRoles = new ArrayList<GrantedAuthority>();
        authRoles.add(new SimpleGrantedAuthority("ROLE_ADMIN"));
        authRoles.add(new SimpleGrantedAuthority("ROLE_USER"));
        return authRoles;
    }
```

####  There is no PasswordEncoder mapped for the id "null"

```json
{bcrypt}$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG 
{noop}password 
{pbkdf2}5d923b44a6d129f3ddf3e3c8d29412723dcbde72445e8ef6bf3b508fbf17fa4ed4d6b99ca763d8dc 
{scrypt}$e0801$8bWJaSu2IKSn9Z9kM+TPXfOc/9bdYSrN1oD9qfVThWEwdRTnO7re7Ei+fUZRJ68k9lTyuTeUp4of4g24hHnazw==$OAOec05+bXxvuu/1qZ6NUR+xQYvYv7BeL1QxwRpY5Pc=  
{sha256}97cde38028ad898ebc02e690819fa220e88c62e0699403e94fff291cfffaf8410849f27605abcbc0


// 例如前方的 加密方式

// 配置加密
 private PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
}

        // Security 5 的 默认加密处理
        if (!BCRYPT_PATTERN.matcher(super.getClientSecret()).matches()) {
            return new BCryptPasswordEncoder().encode(super.getClientSecret());
        } else {
            return super.getClientSecret();
        }

// 不使用加密
@Bean  
public static PasswordEncoder passwordEncoder(){  
	return NoOpPasswordEncoder.getInstance();  
}  


保证加密方式
```

