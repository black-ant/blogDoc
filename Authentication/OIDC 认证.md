---

---

# OIDC 笔记: 



[TOC]



## 一  基础知识点

OpenID Connetction :  OIDC= (Identity, Authentication) + OAuth 2.0 , OAuth 2 构建了一个身份层 ， 是一个基于 OAuth 2 的身份认证标准协议 ，

OIDC 提供了ID Token 来解决 第三方客户端 标识用户身份 的问题 ，在Oauth2 的授权流程中 ，一并提供用户的身份认证信息给第三方客户端 ，ID token 使用JWT 格式进行包装 ， 得益于 JWT 的包容性 ，紧凑性 和 防篡改机制  ， 并且提供 UserInfo 

**区别 ：** OIDC 在 OAuth2  AccessToken 的基础上增加了身份认证信息 ，通过公钥私钥配合校验获取身份等其他信息—– 即idToken ，OIDC 允许所有的客户端 请求和 接收有关身份验证的会话和最终用户的信息。



------

**OIDC 1.0 规范 ：**

```
> 定义了 核心OpenID Connect功能
> 定义客户端如何动态发现有关OpenID提供程序的信息
> 定义客户端如何动态注册OpenID提供程序
> 定义了几种特定的新OAuth 2.0响应类型
> 定义如何使用由用户代理使用HTTP POST自动提交的HTML表单值返回OAuth 2.0授权响应参数
> 定义如何管理OpenID Connect会话，包括基于postMessage的注销和RP启动的注销功能
> 定义在RP页面上不使用OP iframe的前端通道注销机制
> 定义注销机制，该注销机制使用正在注销的OP和RP之间的直接反向通道通信
> 定义OP和RP的集合如何通过联合运营商建立信任
```



**OIDC构成  ：** core , Discovery , Dynamic Registration  , OAuth 2.0 Multiple Response Types  , OAuth 2.0 Form Post Response Mode ,Session Management  ,Front-Channel Logout , Back-Channel Logout

- core : 定义 OIDC 的核心功能 ，在OAuth 2 之上 构建身份认证 ，以及使用 Claims 来传递用户信息 

- Discovery :  发现服务  ,  用于客户端动态的获取OIDC 服务相关的元数据描述信息

- Dynamic Registration ： 动态注册服务 ， 使客户端可以动态的注册到OIDC 的 OP 

- OAuth 2.0 Multiple Response Types ：可选。针对OAuth2的扩展，提供几个新的response_type。

- OAuth 2.0 Form Post Response Mode：可选。针对OAuth2的扩展，OAuth2回传信息给客户端是通过URL的querystring和fragment这两种方式，这个扩展标准提供了一基于form表单的形式把数据post给客户端的机制。

- Session Management ：可选。Session管理，用于规范OIDC服务如何管理Session信息

- Front-Channel Logout：可选。基于前端的注销机制，使得RP（这个缩写后面会解释）可以不使用OP的iframe来退出

- Back-Channel Logout：可选。基于后端的注销机制，定义了RP和OP直接如何通信来完成注销

 

------



**主要术语 :**

```
>   EU : END USER : 一个用户单位
>   RP : Relying Party , 受信任客户端 ，身份认证消费方
>   OP : OpenID Provider : 有能力提供EU 认证的服务 ， 为RP 提供 EU 身份认证信息
>   ID Token  : JWT 格式 的数据 ， 包含 EU 身份认证的信息
>   UserInfo Endpoint : 用户信息接口 ，必须为HTTPS 
```



 

## 二   . OIDC 工作流程

    1. RP 向 OP 申请 授权 ，OP 返回授权Access Token 以及 ID Token  ,  使用Access Token 向 User Info EndPoint 请求 信息
    2. AuthN 请求虽然是复用OAuth 2 的  Authorization 请求 ，但是用途不一样 ，OIDC 的 authN scope 参数 必须要有一个openid 的参数 
          
    》 OIDC 单点登录流程
      > 1 . 用户点击登录 ，触发对OIDC-SERVER 的认证请求
         |-> request : 包含参数URL , 指向登录成功后的跳转地址
         |-> response ： 返回 302 ，Location 指向 OIDC-SERVER ，Set-Cookie 设置了 nonce的cookie
      > 2 . 向 OIDC-SERVER 发起 authc 请求
         |-> client_id=implicit-client ：发起认证请求的客户端的唯一标识
         |-> reponse_mode=form_post ：使用form表单的形式返回数据
         |-> response_type=id_token ：返回包含类型 id_token
         |-> scope=openid profile ：返回包含有openid这一项
         |-> state ：等同于OAuth2 state ，用于保证客户端一致性
         |-> nonce ： 写入的cookie 值
         |-> redirect_uri ： 认证成功后的回调地址
      > 3 . OIDC-SERVER 验证 authc 请求
         |-> client_id是否有效，redircet_uri是否合法 等一系列验证
      > 4 . 引导用户登录 ，以及用户登录 
         |-> resumeURL 
         |-> username + password     
      > 5 . 返回一个自动提交form 表单的页面
         |-> id_token：id_token即为认证的信息，OIDC的核心部分，采用JWT格式包装的一个字符串
         |-> scope：用户允许访问的scope信息
         |-> state : 类似
         |-> session_state ：会话状态    
      > 6 . 验证数据有效性，构造自身登录状态
         |-> 客户端验证id_token的有效性 ，保证客户端得到的id_token是oidc-sercer.dev颁发的
    
    》 OIDC 统一退出流程
      > 1 . 退出自身状态 ，触发对OIDC-SERVER 的退出请求
      > 2 . 向 OIDC-SERVER 发起 退出请求
      > 3 . OIDC-SERVER 验证退出请求
      > 4 . OIDC-SERVER 退出自身状态
      > 5 . 返回html 页面
      > 1 .   
      
    The RP (Client) sends a request to the OpenID Provider (OP).
    The OP authenticates the End-User and obtains authorization.
    The OP responds with an ID Token and usually an Access Token.
    The RP can send a request with the Access Token to the UserInfo Endpoint.
    The UserInfo Endpoint returns Claims about the End-User. 
      
    +--------+                                   +--------+
    |        |                                   |        |
    |        |---------(1) AuthN Request-------->|        |
    |        |                                   |        |
    |        |  +--------+                       |        |
    |        |  |        |                       |        |
    |        |  |  End-  |<--(2) AuthN & AuthZ-->|        |
    |        |  |  User  |                       |        |
    |   RP   |  |        |                       |   OP   |
    |        |  +--------+                       |        |
    |        |                                   |        |
    |        |<--------(3) AuthN Response--------|        |
    |        |                                   |        |
    |        |---------(4) UserInfo Request----->|        |
    |        |                                   |        |
    |        |<--------(5) UserInfo Response-----|        |
    |        |                                   |        |
    +--------+                                   +--------+


​          

## 三  .  什么是  ID Token

```
> ID Token 是一个安全令牌 ， 包含用户信息的JWT 格式的数据结构 ，ID Token 主要包含 ：
>  iss = Issuer Identifier：必须。提供认证信息者的唯一标识
>  sub = Subject Identifier：必须。iss提供的EU的标识，在iss范围内唯一。它会被RP用来标识唯一的用户
  aud = Audience(s) ：标识ID Token的受众。必须包含OAuth2的client_id
>  exp = Expiration time ： 过期时间，超过此时间的ID Token会作废不再被验证通过
>  iat = Issued At Time ：JWT的构建的时间 
>  auth_time = AuthenticationTime ： EU完成认证的时间
>  nonce：RP发送请求的时候提供的随机字符串，用来减缓重放攻击，也可以来关联ID Token和RP本身的Session信息
>  acr = Authentication Context Class Reference：可选。表示一个认证上下文引用值，可以用来标识认证上下文类
>  amr = Authentication Methods References：可选。表示一组认证方法
>  azp = Authorized party：可选。结合aud使用。只有在被认证的一方和受众（aud）不一致时才使用此值，一般情况下很少使用

```

 

## 四  OIDC 的认证流程

```
>  Authorization Code Flow：使用OAuth2的授权码来换取Id Token和Access Toke
>  Implicit Flow：使用OAuth2的Implicit流程获取Id Token和Access Token
>  Hybrid Flow：混合Authorization Code Flow+Implici Flow
```

 

###  基于Authorization Code的认证请求 
```
> 使用OAuth 2 的 Authorization Code 的方式 来完成 用户 身份认证 ，所有 Token Endpoint 来发放 
> OIDC的Authentication Request
    >  scope：必须。OIDC的请求必须包含值为“openid”的scope的参数。
    >  response_type：必选。同OAuth2。
    >  client_id：必选。同OAuth2。
    >  redirect_uri：必选。同OAuth2。
    >  state：推荐。同OAuth2。防止CSRF, XSRF。

额外参数
    >  response_mode ： 用何种方式返回数据
    >  nonce ： ID Token 中的 nonce
    >  display  :  授权服务器呈现怎样的页面给EU （ page , popup , touch , wap）
    >  prompt : 允许传递多个值 ，指示授权服务器 是否引导EU 重新认证和同意授权 （ none , login ,consent ,  select_account ）
    >  max_age : EU 认证信息的有效时间
    >  ui_locales : 用户界面的本地化语言设置
    >  id_token_hint : 之前发送的ID Token 
    >  login_hint : 提示登录标识符
    >  acr_value : Authentication Context Class Reference values，对应ID Token中的acr的Claim。此参数允许多个值出现，使用空格分割
```




### 几个常见的示例

```JAVA
//请求

GET /authorize?
response_type=code
&scope=openid%20profile%20email
&client_id=s6BhdRkqt3
&state=af0ifjsldkj
&redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb HTTP/1.1
    
Host: server.example.com

 

// 基于 authorization code 响应
  HTTP/1.1 302 Found
  Location: <https://client.example.org/cb>?code=SplxlOBeZQQYbYS6WxSbIA&state=af0ifjsldkj

 

//响应Token

  HTTP/1.1 200 OK
  Content-Type: application/json
  Cache-Control: no-store
  Pragma: no-cache

  {
   "access_token": "SlAV32hkKG",
   "token_type": "Bearer",
   "refresh_token": "8xLOxBtZp8",
   "expires_in": 3600,
   "id_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjFlOWdkazcifQ.ewogImlzc
yI6ICJodHRwOi8vc2VydmVyLmV4YW1wbGUuY29tIiwKICJzdWIiOiAiMjQ4Mjg5
NzYxMDAxIiwKICJhdWQiOiAiczZCaGRSa3F0MyIsCiAibm9uY2UiOiAibi0wUzZ
fV3pBMk1qIiwKICJleHAiOiAxMzExMjgxOTcwLAogImlhdCI6IDEzMTEyODA5Nz
AKfQ.ggW8hZ1EuVLuxNuuIJKX_V8a_OMXzR0EHR9R6jgdqrOOF4daGU96Sr_P6q
Jp6IcmD3HP99Obi1PRs-cwh3LO-p146waJ8IhehcwL7F09JdijmBqkvPeB2T9CJ
NqeGpe-gccMg4vfKjkM8FcGvnzZUN4_KSP0aAp1tOJ1zZwgjxqGByKHiOtX7Tpd
QyHE5lcMiKPXfEIQILVq0pc_E2DzL7emopWoaoZTF_m0_N0YzFC6g6EJbOEoRoS
K5hoDalrcvRYLSrQAZZKflyuVCyixEoV9GfNQC3_osjzw2PAithfubEEBLuVVk4
XUVrWOLrLl0nx7RkKU8NXNHq-rvKMzqg"
}

 
```

 

 

### OIDC Discovery 规范

```
定义了一个服务发现的规范，它定义了一个api（ /.well-known/openid-configuration ），这个api返回一个json数据结构，其中包含了一些OIDC中提供的服务以及其支持情况的描述信息，这样可以使得oidc服务的RP可以不再硬编码OIDC服务接口信息

相信大家都看得懂的，它包含有授权的url，获取token的url，注销token的url，以及其对OIDC的扩展功能支持的情况等等信息，这里就不再详细解释每一项了
```

 

### OAuth2 扩展：Multiple Response Types

```
response_type 是 OIDC 的请求参数之一 , OIDC 继承了OAuth的 Code 和 Token,  同时扩展了 新类型 id_token
```



###  会话管理
```
>  Session Management ：可选。Session管理，用于规范OIDC服务如何管理Session信息。
>  Front-Channel Logout：可选。基于前端的注销机制。
>  Back-Channel Logout：可选。基于后端的注销机制。

```

 

### Front-Channel Logout 退出登录
```
>  1 client 自身退出登录
>  2 通知OIDC 退出登录 ，并且携带必要的数据
>  3 OIDC 验证 有效性 ，并且退出登陆
>  4 返回相关的html ，包含iframe ,分别包含其他站点的退出登录URL , 其他站点验证退出登录的有效性 ，并且退出登录 
>  RP退出登录的URL地址（这个在RP注册的时候会提供给OIDC服务）
>  URL中的sessionid这个参数
```

### OIDC 的好处

```
OIDC使得身份认证可以作为一个服务存在。
OIDC可以很方便的实现SSO（跨顶级域）。
OIDC兼容OAuth2，可以使用Access Token控制受保护的API资源。
OIDC可以兼容众多的IDP作为OIDC的OP来使用。
OIDC的一些敏感接口均强制要求TLS，除此之外，得益于JWT,JWS,JWE家族的安全机制，使得一些敏感信息可以进行数字签名、加密和验证，进一步确保整个认证过程中的安全保障。
```

## 五 OIDC 的使用

### 5 . 1 OIDC 的 SpringBoot 使用

```

```



### 参考

[OpenID Connect](https://openid.net/connect/)