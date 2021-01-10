#  SAML 协议总结

[TOC]



## 一  简介

```java
SAML 是一种 XML 格式的语言 ，即为安全断言语言 ，（安全 + 断言）
断言 ： 断言通俗点说 通过 某种模型语句 对 某个 判断进行表述 （ A 不是 超级管理员 ）  

> 技术架构 : 
- Extensible Markup Language (XML)
- XML Schema
- XML Signature
- XML Encryption (SAML 2.0 only)
- Hypertext Transfer Protocol (HTTP)
- SOAP

> 主要部分
SAML Assertions 断言：定义交互的数据格式 (XML)
SAML Protocols 协议：定义交互的消息格式 (XML+processing rules)
SAML Bindings 绑定：定义如何与常见的通信协议绑定 (HTTP,SOAP)
SAML Profile 使用框架：给出对 SAML 断言及协议如何使用的建议 (Protocols+Bindings)


> SAML Protocols 
SAML Protocols 描述了 SAML 元素（包括断言）如何被打包到 SAML 请求和响应元素中，并规定 SAML 实体（IdP、SP 等）处理这些元素时必须遵守的处理规则
// 常见的有 : urn:oasis:names:tc:SAML:1.0:protocol


> SAML Bindings( 绑定 )
SAML Bindings( 绑定 ) 是 SAML Protocols 信息到一个标准信息格式或者通信协议的映射过程。例如 SAML SOAP 绑定就定义了一个 SAML 消息如何被封装到 SOAP envelope 中。

> SAML Profile( 使用框架 )
SAML Profile( 使用框架 ) 描述了如何使用 SAML 协议信息和断言来处理特定的业务用户实例。SAML 2.0 较之 SAML1.1 提供了更多使用框架，不过目前最常被使用的依然是 Web Browser SSO。本文下一章将重点介绍 Web Browser SSO Profile。
```

<img src="C:\Users\10169\OneDrive\笔记文档\图片文件\安全框架\saml001.jpg" style="zoom: 200%;" />

### 1.2 主要流程

```java
1 请求端作为一个使用者 , 期望访问服务商的资源
2  SP 发现这是一个未认证的请求 , 此时会返回一个HTML , Form 的隐藏域中有一个 SAML 的认证请求数据包 , 实际上这里不需要手动  , HTML 会通过JS 自动执行到 IDP 中
3 通过 JS 会自动提交到 IDP 中 , 此时 IDP 接收到请求后 , 返回验证界面
( 2-3 步在浏览器端不可见 )
4 IDP 给浏览器返回一个待登录的页面
5 用户进行了认证 , 并且成功提交到 IDP
6 IDP 组装一个HTML Form , 其中包含一个Response ,Response 中有一个对成功用户的断言 (用户信息及权限) , 有认证情况 , 及私钥签名等
7  HTML 被自动提交到 SP , 前面说了 第六步的时候会同时返回一个私钥 , 这一步中  ,SP 会通过公钥校验断言是否合法
8 剩下的就是判断是一个合法用户 ,然后重定向到请求的页面中


```



<img src="C:\Users\10169\OneDrive\笔记文档\图片文件\安全框架\SAML01.png" style="zoom:200%;" />

## 二  知识点 

### 2 . 1 角色 

```JAVA
>  Service Provider + Identity Provider + Client

SP(Service Provider): 向用户提供正式商业服务的实体，通常需要认证一个用户的身份；
IDP(Identity Provider): 提供用户的身份鉴别，确保用户是其所声明的身份

断言 (Assertions) 即信息：断言是在 SAML 中用来描述认证的对象，其中包括一个用户在什么时间、以什么方式被认证，同时还可以包括一些扩展信息，比如用户的 Email 地址和电话等等。

协议 (Protocol) 即通信：协议规定如何执行不同的行为。这些行为被细化成一些列的 Request 和 Response 对象，而在这些请求和相应的对象中包含了行为所特别需要的信息。比如，认证请求协议（AuthnRequest Protocol）就规定了一个 SP 如何请求去获得一个被认证的与用户。

绑定 (Binding) 即传输：绑定定义了 SAML 信息如何使用通信协议被传输的。比如，HTTP 重定向绑定，即声明 SAML 信息将通过 HTTP 重定向消息传输；再比如 SAML SOAP 绑定，声明了通过 SOAP 来传递 SAML 消息。比如上面 AuthnRequest 就声明了 Http-POst 绑定

元数据 (MetaData)：SAML 的元数据是配置数据，其包含关于 SAML 通信各方的信息，比如通信另一方的 ID、Web Service 的 IP 地址、所支持的绑定类型以及通信中实用的密钥等等。
    
// 关于 IDP 和 SP 的角色定位 , 与OAuth不同 , 认证中心和资源服务被刻意的区分为了2个概念 ,在这2个概率里面 , 我一直有一个纠结 : 身份信息到底到谁手上?
// 关于这个其实不需要太纠结 , IDP 里面是存在身份数据的 , 同样 SP 里面也可能存在身份数据 , 通常而言 , IDP是一个认证中心 , 它认证完成后生成的credential在绝大多数下只会包含一个Username , 然后SP会拿着username 去直接使用或者获取更详细的信息  

```

### 2.  2 声明

```
> 常见的三种声明
认证声明：声明用户是否已经认证，通常用于单点登录。
属性声明：声明某个 Subject 所具有的属性。
授权决策声明：声明某个资源的权限，即一个用户在资源 R 上具有给定的 E 权限而能够执行 A 操作。
```

### 2 . 3  访问流程 

```
> 前置条件 : 
client 在   Identity Provider 有映射 ，可以被判断合法 ，认证
SP - IP : 双方相互信任 ， 双方持有对方的公钥
```

1 .  请求权限访问资源
2 .  服务商判断无权限 ，返回一个 saml 格式 的 认证请求数据包 ，这个数据包 会放在 一个 form 的隐藏域中 



```xml
<samlp:AuthnRequest
    xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol"
    xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion"
    ID="aaf23196-1773-2113-474a-fe114412ab72"
    Version="2.0"
    IssueInstant="2004-12-05T09:21:59"
    AssertionConsumerServiceIndex="0"
    AttributeConsumingServiceIndex="0">
    <saml:Issuer>https://sp.example.com/SAML2</saml:Issuer>
    <samlp:NameIDPolicy
      AllowCreate="true"
      Format="urn:oasis:names:tc:SAML:2.0:nameid-format:transient"/>
  </samlp:AuthnRequest>

```



3 . 通过JS 自动提交到 IDP 中 , action 地址
4 .  IDP 准备进行验证 ，返回一个认证页面 ，通过某种方式认证
5 .  IDP 认证合法 ，进而生成断言 ，通过私钥签名 ，包装为Respose  格式 ，放在form 中 返回


``` xml
<saml:Assertion
   xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion"
   xmlns:xs="http://www.w3.org/2001/XMLSchema"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   ID="b07b804c-7c29-ea16-7300-4f3d6f7928ac"
   Version="2.0"
   IssueInstant="2004-12-05T09:22:05">
   <saml:Issuer>https://idp.example.org/SAML2</saml:Issuer>
   <ds:Signature
     xmlns:ds="http://www.w3.org/2000/09/xmldsig#">...</ds:Signature>
   <saml:Subject>
   。。。。。
   </saml:Subject>
   <saml:Conditions
   。。。。
   </saml:Conditions>
   <saml:AuthnStatement
     AuthnInstant="2004-12-05T09:22:00"
     SessionIndex="b07b804c-7c29-ea16-7300-4f3d6f7928ac">
     <saml:AuthnContext>
       <saml:AuthnContextClassRef>
         urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport
      </saml:AuthnContextClassRef>
     </saml:AuthnContext>
   </saml:AuthnStatement>
   <saml:AttributeStatement>
     <saml:Attribute
       xmlns:x500="urn:oasis:names:tc:SAML:2.0:profiles:attribute:X500"
       x500:Encoding="LDAP"
       NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri"
       Name="urn:oid:1.3.6.1.4.1.5923.1.1.1.1"
       FriendlyName="eduPersonAffiliation">
       <saml:AttributeValue
         xsi:type="xs:string">member</saml:AttributeValue>
       <saml:AttributeValue
         xsi:type="xs:string">staff</saml:AttributeValue>
     </saml:Attribute>
   </saml:AttributeStatement>
 </saml:Assertion>
 
```

6 . SP 接收到该断言 ，通过公钥 验证了 断言的签名 ，判断合法 ，返回最终页面 



### 2 . 4 SAML 标准结构

```
    层次 一 ： Assertion :  断言 <saml:Assertion>
    层次 二 ： Protocols ：规定如何请求( samlrequest）和回复(samlresponse）saml消息,当然包含 assertion消息<samlp:AuthnRequest> + <samlp:Response> 
    层次 三 ：Bindings : 绑定  ，决定用何种方式进行传输 
    层次 四 ：Profile ： 配套方案 ，
```

###  2.5 证书 问题处理

#### 2.5.1 密钥和公钥

```java
// 查看 Demo 中 , 一个完整的请求录入了以下 Config
- KeystorePassword : 密钥密码
- PrivateKeyPassword
- KeystorePath : 密钥文件
- IdentityProviderMetadataResourceUrl : IDP MetaData
- ServiceProviderMetadataResource : SP Metaadata

可信实体包含公钥的证书会以X.509证书格式发布在metadata中，而对应的私钥则安全保存在本地。这些密钥被用于消息层面的签名和加密，而SAML消息在传输过程中由TLS协议来进行安全交换。
    
> 阶段一 : 当IDP 拿到 SP 的请求时 , 证书的作用并不明显 , 主要有如下的作用
    - 确定请求来着信任的 SP
    - 
    
> 阶段二 : 当 SP 获取 IDP 的反馈时 , SP 会做以下几件事
    - 通过签名确定来自已知的 IDP
    - 获取使用IDP私钥签名的内容
    - 使用 IDP 公钥验证签名
    

1 SAML核心配置元素 : MetaData , KeyStore 
     - keyStore : 加密的方式通常为RSA , 
     - MetaData 元数据 : 元数据是双方交互的重要依据 , 它类似于一个路线表(或者字典) , Client 端通常会通过Rest 接口拿到Server端提供的元数据 , 然后通过元数据中提供的地址登数据完成回调等操作 (注意 ,MetaData 有2份)

2 IDP MetaData
    -- sign : 签名
    -- encryption : 加密 
    -- SingleLogoutService : 不同类型认证调用的地址

3 SP MetaData 

4 IDP 和 SP 分别持有对方的 MetaData , 先通过MetaData 上的 Sign 和 encryption 进行校验 , 然后 通过 定义的地址调用对方的信息
    
    
```



## 三  SAML SSO Web

### 3 . 1  SAML 2.0 HTTP POST binding

```
SAML 2.0 HTTP POST binding 定义了由 HTML 表单控制（ Form Control）来传送 SAML 协议消息的机制
	- 这个绑定可能和 HTTP 重定向绑定（Redirect Binding）结合使用。
	- 适用于当 SAML 请求者和响应者需要通过 HTTP 用户代理来通信时。
	- 该绑定具体需要传送的数据可以通过 SAML 2.0 的文档描述文件中找到。
```

## # 附录

### 名词概念

```java
// Assertion
	A part of SAML message (an XML document) which provides facts about subject of the assertion (typically about the authenticated user). Assertions can contain information about authentication, associated attributes or authorization decisions.
	SAML 消息(XML 文档)的一部分，它提供关于断言主题的事实(通常是关于经过身份验证的用户)。断言可以包含有关身份验证、关联属性或授权决策的信息

// Artifact 	
	Identifier which can be used to retrieve a complete SAML message from identity or service provider using a back-channel binding. 
	标识符，该标识符可用于从标识或使用后台通道绑定的服务提供者检索完整的 SAML 消息
    
// Bnding 
	Mechanism used to deliver SAML message. Bindings are divided to front-channel bindings which use web-browser of the user for message delivery (e.g. HTTP-POST or HTTP-Redirect) and back-channel bindings where identity provider and service provider communicate directly (e.g. using SOAP calls in Artifact binding). 
	用于传递 SAML 消息的机制。绑定分为前端通道绑定和后端通道绑定，前者使用用户的 web 浏览器进行消息传递(例如 HTTP-POST 或 HTTP-Redirect) ，后者使身份提供者和服务提供者直接通信(例如在 Artifact 绑定中使用 SOAP 调用)
    
// Dscovery 	
    Mechanism used to determine which identity provider should be used to authenticate user currently interacting with the service provider. 
   用于确定应该使用哪个身份提供程序来验证当前与服务提供程序交互的用户
    
// Metadata 	
    Document describing one or multiple identity and service providers. Metadata typically includes entity identifier, public keys, endpoint URLs, supported bindings and profiles, and other capabilities or requirements. Exchange of metadata between identity and service providers is typically the first step for establishment of federation. 
    描述一个或多个身份和服务提供者的文档。元数据通常包括实体标识符、公钥、端点 url、支持的绑定和配置文件以及其他功能或需求。身份和服务提供者之间的元数据交换通常是建立联合的第一步
        
// Profile
	Standardized combination of protocols, assertions, bindings and processing instructions used to achieve a particular use-case such as single sign-on, single logout, discovery, artifact resolution. 
	用于实现特定用例的协议、断言、绑定和处理指令的标准化组合，如单点登录、单点注销、发现、工件解析
        
// Protocol
	Definition of format (schema) for SAML messages used to achieve particular functionality such as requesting authentication from IDP, performing single logout or requesting attributes from IDP. 
	为 SAML 消息定义格式(模式) ，用于实现特定功能，例如从 IDP 请求身份验证、执行单个注销或从 IDP 请求属性
        
// Identity provider (IDP) 身份提供者(IDP)	
        Entity which knows how to authenticate users and provides information about their identity to service providers/relaying parties using federation protocols. 
        知道如何认证用户并使用联邦协议向服务提供者/中继方提供有关其身份的信息的实体
        
//Service provider (SP) 服务供应商	
	Your application which communicates with the identity provider in order to obtain information about the user it interacts with. User information such as authentication state and user attributes is provided in form of security assertions. 
     您的应用程序与身份提供者通信，以获取与其交互的用户的信息。身份验证状态和用户属性等用户信息是以安全断言的形式提供的
        
//Single Sign-On (SSO) 单点登录(SSO)	
        Process enabling access to multiple web sites without need to repeatedly present credentials necessary for authentication. Various federation protocols such as SAML, WS-Federation, OpenID or OAuth can be used to achieve SSO use-cases. Information such as means of authentication, user attributes, authorization decisions or security tokens are typically provided to the service provider as part of single sign-on. 
	允许访问多个网站的进程，而不需要重复提供身份验证所需的凭证。可以使用各种联邦协议(如 SAML、 WS-Federation、 OpenID 或 OAuth)来实现 SSO 用例。身份验证方式、用户属性、授权决策或安全令牌等信息通常作为单点登录的一部分提供给服务提供者
        
//Single Logout (SLO) 单一登出(SLO)	
	Process terminating authenticated sessions at all resources which were accessed using single sign-on. Techniques such as redirecting user to each of the SSO participants or sending a logout SOAP messages are typically used. 
	在使用单点登录访问的所有资源上处理终止身份验证会话。通常使用的技术包括将用户重定向到每个 SSO 参与者或发送注销 SOAP 消息
```

###  证书内容解析

### Metadata  IDP 证书

```xml
<EntityDescriptor
    xmlns="urn:oasis:names:tc:SAML:2.0:metadata"
    xmlns:ds="http://www.w3.org/2000/09/xmldsig#"
    xmlns:shibmd="urn:mace:shibboleth:metadata:1.0"
    xmlns:mdui="urn:oasis:names:tc:SAML:metadata:ui" entityID="http://127.0.0.1/security/idp">
    <IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol urn:oasis:names:tc:SAML:1.1:protocol urn:mace:shibboleth:1.0">
        <Extensions>
            <shibmd:Scope regexp="false">scope</shibmd:Scope>
        </Extensions>
        <KeyDescriptor use="signing">
            <ds:KeyInfo>
                <ds:X509Data>
                    <ds:X509Certificate></ds:X509Certificate>
                </ds:X509Data>
            </ds:KeyInfo>
        </KeyDescriptor>
        <KeyDescriptor use="encryption">
            <ds:KeyInfo>
                <ds:X509Data>
                    <ds:X509Certificate></ds:X509Certificate>
                </ds:X509Data>
            </ds:KeyInfo>
        </KeyDescriptor>
        <NameIDFormat>urn:mace:shibboleth:1.0:nameIdentifier</NameIDFormat>
        <NameIDFormat>urn:oasis:names:tc:SAML:2.0:nameid-format:transient</NameIDFormat>
        <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="http://127.0.0.1/security/idp/profile/SAML2/POST/SLO"/>
        <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="http://127.0.0.1/security/idp/profile/SAML2/POST/SSO"/>
        <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="http://127.0.0.1/security/idp/profile/SAML2/Redirect/SSO"/>
        <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:SOAP" Location="http://127.0.0.1/security/idp/profile/SAML2/SOAP/ECP"/>
    </IDPSSODescriptor>
</EntityDescriptor>
```

### MetaData sp 

```xml
<?xml version="1.0" encoding="UTF-8"?><md:EntityDescriptor xmlns:md="urn:oasis:names:tc:SAML:2.0:metadata" ID="_5p2k1rpmlbr0hphrontb6zwiplqac1xxpzvzdma" entityID="http://127.0.0.1:9081/client/saml/callback" validUntil="2040-05-07T14:08:50.499Z">
    <ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
        <ds:SignedInfo>
            <ds:CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"/>
            <ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256"/>
            <ds:Reference URI="#_5p2k1rpmlbr0hphrontb6zwiplqac1xxpzvzdma">
                <ds:Transforms>
                    <ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature"/>
                    <ds:Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"/>
                </ds:Transforms>
                <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
                <ds:DigestValue>GSp3lIKMQs70Q6FQYHWFhVaGKJv31AiRTuXOcyO78mk=</ds:DigestValue>
            </ds:Reference>
        </ds:SignedInfo>
        <ds:SignatureValue>
            ObNzvhAMTtBXE5Pm1Vxh3rw6/tsa+jhs1tnySl4ibkr7MtqAFVfnaSS+xXTqQeQ7ARxB2MeNogVF&#13;
            Hh99LfYJ6ITgp7fKDQE1Rs+ZHfYYedgl2ROR3O7rOrs7DpwJRfcVnshV09SVr1y5FD7jyCN13nij&#13;
            ndGFnFDo6UO4D89s4RIx4fXEEVEB213342134123421412wcq3onDc4XpXlRdE3MQjoFq9B+4CbR&#13;
            diRG+lI+rYTTNV+wT0cyWPYzcH1LzIXElGEqI52ZuWKltg3G68N63Jm06V/TH1qFCxpFB00uhTPl&#13;
            dD6CPgyaSH4iP4eETyClFSNXyjohKU2gLbBYMg==
        </ds:SignatureValue>
        <ds:KeyInfo>
            <ds:X509Data>
                <ds:X509Certificate>MIICrTCCAZWgAwIBAgIBATANBgkqhkiG9w0BAQUFADAaMRgwFgYDVQQDDA9ERVNLVE9QLTdNMzBP
                    SzMwHhcNMjAwNTA3MTQwODQ2WhcNMjEwNTA3MTQwODQ3WjAaMRgwFgYDVQQDDA9ERVNLVE9QLTdN
                    MzBPSzMwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCLX2P1Je1XgI6QgVDIU/48Na//
                    mFiFQAFEvUCEjsGh9GKESJgv+GxtLw1c7OkR+8gGQEjWZ+HeFry7g+hZDLEhxxHJQO9tKhVb4QQh
                    uueRzzr/G9gw/dErnqNRr5p7i3TVHi/uC1oEn72VI+ud9UVYcWEU2d11UBpx28qDkwBBxR0igpjg
                    0f2iZ7EOgwG6+lbs4s8lKB/OSEKMRc3Gb4GJ0CWLQM/2q2UTzOE1wONuo7Hln6j7u6a05hrQ4su2
                    Pl+Pb1Ya7gnQ9WHSsxg2mH12DwOLKxEGg97+GQv9nv/py/2oJ+l+dTj3ljnsSdV+Xt8zqrZAdcN5
                    7EmXDTRG9lq5AgMBAAEwDQYJKoZIhvcNAQEFBQADggEBAGDuQ3pT9aWHbmf/ebUy/BhHiV38TYp4
                    WOFa/r+lvMvXUdC8tssBHwzStalA+nue3UFu+WLoERptx9Dwme68ulDxzMlK6+M3lwRnDnuofJL4
                    DRSU5tsUa5bPtBn0Gk6UBuGH213421423141234214214124axCALJ6qrsrZ0Rpy7Tow5dhCpWMw
                    hybS6f6CXyrYlz8WtsJmS8QNWDZWn2q5H20V3AxYnCirOKEYD5+Id1yFbqzSydxYuixa2cms+MWc
                    oE0SjFW0hn2h7Ekk+dNRKpw28Po1aaqE3BvCX5PoyXcNwfOgFBCh3vGdopDbGtc9i8YJE/FITW7x
                    2Gwpy+U=</ds:X509Certificate>
            </ds:X509Data>
        </ds:KeyInfo>
    </ds:Signature>
    <md:Extensions xmlns:alg="urn:oasis:names:tc:SAML:metadata:algsupport">
        <alg:SigningMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256"/>
        <alg:SigningMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha384"/>
        <alg:SigningMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha512"/>
        <alg:SigningMethod Algorithm="http://www.w3.org/2000/09/xmldsig#rsa-sha1"/>
        <alg:SigningMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#ecdsa-sha256"/>
        <alg:SigningMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#ecdsa-sha384"/>
        <alg:SigningMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#ecdsa-sha512"/>
        <alg:SigningMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#ecdsa-sha1"/>
        <alg:SigningMethod Algorithm="http://www.w3.org/2000/09/xmldsig#dsa-sha1"/>
        <alg:SigningMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#hmac-sha256"/>
        <alg:SigningMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#hmac-sha384"/>
        <alg:SigningMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#hmac-sha512"/>
        <alg:SigningMethod Algorithm="http://www.w3.org/2000/09/xmldsig#hmac-sha1"/>
        <alg:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
        <alg:DigestMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#sha384"/>
        <alg:DigestMethod Algorithm="http://www.w3.org/2000/09/xmldsig#sha1"/>
    </md:Extensions>
    <md:SPSSODescriptor AuthnRequestsSigned="false" WantAssertionsSigned="false" protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol urn:oasis:names:tc:SAML:1.0:protocol urn:oasis:names:tc:SAML:1.1:protocol">
        <md:Extensions xmlns:init="urn:oasis:names:tc:SAML:profiles:SSO:request-init">
            <init:RequestInitiator Binding="urn:oasis:names:tc:SAML:profiles:SSO:request-init" Location="http://127.0.0.1:9081/client/saml/callback?client_name=SAML2Client"/>
        </md:Extensions>
        <md:KeyDescriptor use="signing">
            <ds:KeyInfo xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
                <ds:X509Data>
                    <ds:X509Certificate>MIICrTCCAZWgAwIBAgIBATANBgkqhkiG9w0BAQUFADAaMRgwFgYDVQQDDA9ERVNLVE9QLTdNMzBP
                        SzMwHhcNMjAwNTA3MTQwODQ2WhcNMjEwNTA3MTQwODQ3WjAaMRgwFgYDVQQDDA9ERVNLVE9QLTdN
                        MzBPSzMwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCLX2P1Je1XgI6QgVDIU/48Na//
                        mFiFQAFEvUCEjsGh9GKESJgv+GxtLw1c7OkR+8gGQEjWZ+HeFry7g+hZDLEhxxHJQO9tKhVb4QQh
                        uueRzzr/G9gw/dErnqNRr5p7i3TVHi/uC1242141241241241WEU2d11UBpx28qDkwBBxR0igpjg
                        0f2iZ7EOgwG6+lbs4s8lKB/OSEKMRc3Gb4GJ0CWLQM/2q2UTzOE1wONuo7Hln6j7u6a05hrQ4su2
                        Pl+Pb1Ya7gnQ9WHSsxg2mH12DwOLKxEGg97+GQv9nv/py/2oJ+l+dTj3ljnsSdV+Xt8zqrZAdcN5
                        7EmXDTRG9lq5AgMBAAEwDQYJKoZIhvcNAQEFBQADggEBAGDuQ3pT9aWHbmf/ebUy/BhHiV38TYp4
                        WOFa/r+lvMvXUdC8tssBHwzStalA+nue3UFu+WLoERptx9Dwme68ulDxzMlK6+M3lwRnDnuofJL4
                        DRSU5tsUa5bPtBn0Gk6UBuGH21421342134214IAOvcFJH+DaxCALJ6qrsrZ0Rpy7Tow5dhCpWMw
                        hybS6f6CXyrYlz8WtsJmS8QNWDZWn2q5H20V3AxYnCirOKEYD5+Id1yFbqzSydxYuixa2cms+MWc
                        oE0SjFW0hn2h7Ekk+dNRKpw28Po1aaqE3BvCX5PoyXcNwfOgFBCh3vGdopDbGtc9i8YJE/FITW7x
                        2Gwpy+U=</ds:X509Certificate>
                </ds:X509Data>
            </ds:KeyInfo>
        </md:KeyDescriptor>
        <md:KeyDescriptor use="encryption">
            <ds:KeyInfo xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
                <ds:X509Data>
                    <ds:X509Certificate>MIICrTCCAZWgAwIBAgIBATANBgkqhkiG9w0BAQUFADAaMRgwFgYDVQQDDA9ERVNLVE9QLTdNMzBP
                        SzMwHhcNMjAwNTA3MTQwODQ2WhcNMjEwNTA3MTQwODQ3WjAaMRgwFgYDVQQDDA9ERVNLVE9QLTdN
                        MzBPSzMwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCLX2P1Je1XgI6QgVDIU/48Na//
                        mFiFQAFEvUCEjsGh9GKESJgv+GxtLw1c7OkR+8gGQEjWZ+HeFry7g+hZDLEhxxHJQO9tKhVb4QQh
                        uueRzzr/G9gw/dErnqNRr5p7i3TVHi/uC1oEn72VI+ud9UVY124214124px28qDkwBBxR0igpjg
                        0f2iZ7EOgwG6+lbs4s8lKB/OSEKMRc3Gb4GJ0CWLQM/2q2UTzOE1wONuo7Hln6j7u6a05hrQ4su2
                        Pl+Pb1Ya7gnQ9WHSsxg2mH12DwOLKxEGg97+GQv9nv/py/2oJ+l+dTj3ljnsSdV+Xt8zqrZAdcN5
                        7EmXDTRG9lq5AgMBAAEwDQYJKoZIhvcNAQEFBQADggEBAGDuQ3pT9aWHbmf/ebUy/BhHiV38TYp4
                        WOFa/r+lvMvXUdC8tssBHwzStalA+nue3UFu+WLoERptx9Dwme68ulDxzMlK6+M3lwRnDnuofJL4
                        DRSU5tsUa5bPtBn0Gk6UBuGHBzpv1542142423344vcFJH+DaxCALJ6qrsrZ0Rpy7Tow5dhCpWMw
                        hybS6f6CXyrYlz8WtsJmS8QNWDZWn2q5H20V3AxYnCirOKEYD5+Id1yFbqzSydxYuixa2cms+MWc
                        oE0SjFW0hn2h7Ekk+dNRKpw28Po1aaqE3BvCX5PoyXcNwfOgFBCh3vGdopDbGtc9i8YJE/FITW7x
                        2Gwpy+U=</ds:X509Certificate>
                </ds:X509Data>
            </ds:KeyInfo>
        </md:KeyDescriptor>
        <md:SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="http://127.0.0.1:9081/client/saml/callback?client_name=SAML2Client&amp;logoutendpoint=true"/>
        <md:SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST-SimpleSign" Location="http://127.0.0.1:9081/client/saml/callback?client_name=SAML2Client&amp;logoutendpoint=true"/>
        <md:SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="http://127.0.0.1:9081/client/saml/callback?client_name=SAML2Client&amp;logoutendpoint=true"/>
        <md:SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:SOAP" Location="http://127.0.0.1:9081/client/saml/callback?client_name=SAML2Client&amp;logoutendpoint=true"/>
        <md:NameIDFormat>urn:oasis:names:tc:SAML:2.0:nameid-format:transient</md:NameIDFormat>
        <md:NameIDFormat>urn:oasis:names:tc:SAML:2.0:nameid-format:persistent</md:NameIDFormat>
        <md:NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress</md:NameIDFormat>
        <md:NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified</md:NameIDFormat>
        <md:AssertionConsumerService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="http://127.0.0.1:9081/mfa-client/saml/callback?client_name=SAML2Client" index="0"/>
        <md:AttributeConsumingService index="0">
            <md:RequestedAttribute FriendlyName="eduPersonPrincipalName" Name="urn:oid:1.3.6.1.4.1.5923.1.1.1.6" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri" isRequired="false"/>
        </md:AttributeConsumingService>
    </md:SPSSODescriptor>
</md:EntityDescriptor>

```

### X509 证书的结构

```
> version : x509 证书结构定义的版本 , 
> algorithm identifier -> signature  : 实际上不包含任何签名信息。这个属性值是签名算法名称
> extensions : 扩展属性。通常CA会利用这个属性增加一些非标准定义的内容。


> signature -> encrypted  : 将上面的属性做hash，然后使用CA的私钥对hash执行数字签名，得到值存放在这个属性
```

|       |                                              |      |
| ----- | -------------------------------------------- | ---- |
| saml  | urn:oasis:names:tc:SAML:2.0:assertion        |      |
| samlp | urn:oasis:names:tc:SAML:2.0:protocol         |      |
| md    | urn:oasis:names:tc:SAML:2.0:metadata         |      |
| ecp   | urn:oasis:names:tc:SAML:2.0:profiles:SSO:ecp |      |
| ds    |                                              |      |
| xenc  |                                              |      |
|       |                                              |      |
|       |                                              |      |
|       |                                              |      |
|       |                                              |      |
|       |                                              |      |



### SAML 证书生成

```java
public static void genX509(String certPath, String password,
                               int keysize, String algorithm) throws Exception {
            // 创建KeyStore
            KeyStore store = KeyStore.getInstance("PKCS12");
            store.load(null, null);
            // 生成一对公私钥，这部分如果自己已经有了PublicKey，可以直接在下面使用，不用生成
            KeyPairGenerator kpg = KeyPairGenerator.getInstance("RSA");
            kpg.initialize(keysize);
            KeyPair keyPair = kpg.generateKeyPair();
            // 这个字符串根据自己情况填
            String issuer = "C=CN,ST=BJ,L=BJ,O=dingtalk,OU=dingtalk,CN=dingtalk";
            String subject = issuer;
            X509V3CertificateGenerator certGen = new X509V3CertificateGenerator();
            certGen.setSerialNumber(BigInteger.valueOf(System.currentTimeMillis()));
            certGen.setIssuerDN(new X500Principal(issuer));
            certGen.setNotBefore(new Date(System.currentTimeMillis() - 10 * 365
                    * 24 * 60 * 60 * 1000));
            certGen.setNotAfter(new Date(System.currentTimeMillis() + 10 * 365 * 24
                    * 60 * 60 * 1000));
            certGen.setSubjectDN(new X500Principal(subject));
            certGen.setPublicKey(keyPair.getPublic());// 此处可直接传入线程的PublicKey
            if (algorithm == null || algorithm.equals("")) {
                certGen.setSignatureAlgorithm("SHA256WithRSAEncryption");
            } else {
                certGen.setSignatureAlgorithm(algorithm);
            }
            X509Certificate cert = certGen.generateX509Certificate(keyPair
                    .getPrivate());
            // 私钥有现成的也可直接传入
            store.setKeyEntry("alias", keyPair.getPrivate(),
                    password.toCharArray(), new Certificate[] { cert });
            // 导出为 cer 证书 和 keystore文件
            try {
                FileOutputStream fos = new FileOutputStream(certPath + ".cer");
                FileOutputStream ksFOS = new FileOutputStream(certPath + ".keystore");
                fos.write(cert.getEncoded());
                fos.close();
                store.store(ksFOS, "123".toCharArray());
                ksFOS.close();
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            } catch (CertificateEncodingException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }
    }




    public static void main(String[] args) throws Exception {
        X509Service impl=new X509ServiceImpl();
        String certDestPath = System.getProperty("user.dir") + "/src/main/resources/static/test/aaaa";

        genX509(certDestPath, "123", Default_KeySize, "");
        impl.printCert(certDestPath + ".keystore", "123");

    }
```