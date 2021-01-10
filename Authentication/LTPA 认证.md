# LTPA : 轻量级第三方认证

[TOC]



##  一  简介 ：

```
> 名称 ：Lightweight Third-Party Authentication
> 功能 :
	该LTPA 是 一项 IBM 协议 ，用于 在 WebSphere®Application Server 中提供基于 cookie 或二进制安全性令牌的认证机制 ，其支持 单点登录 SSO .
	这些服务器包括 WebSphere® 和 DataPower®。要对其中一个或多个服务器实现单点登录解决方案，您可以将 WebSEAL 配置为支持 LTPA 认证

> 目的 : 
	LTPA 令牌认证的目的是将 LTPA 令牌从第一个 Web Service（其认证生成客户机）流动到下游 Web Service
	
	
> 重点 : 
基于 Cookie 的轻量级第三方认证机制 (LTPA)
通常用于IBM 相关的服务 , 其他的服务暂不清楚

```

<img src="C:\Users\10169\OneDrive\笔记文档\图片文件\其他\LTPA\ltpa01.jpg" style="zoom:200%;" />



### 1 . 1 LTPA Cookie 

```java
LTPA Cookie（充当 WebSphere/DataPower 的认证令牌）包含用户身份、密钥和令牌数据、缓冲区长度以及到期信息。此信息通过使用受密码保护的密钥进行加密，此密钥在 WebSEAL 与其他支持 LTPA 的服务器之间进行共享


WebSEAL 仅支持 LTPA V2 (LtpaToken2) Cookie。与先前版本的令牌相比，LtpaToken2 包含更强的加密功能，并且使您能够向令牌添加多个属性。此令牌包含认证身份和附加信息，例如，用于联系原始登录服务器的属性，以及当确定唯一性需要考虑除身份以外的其他内容时，用于查找主体集的唯一高速缓存密钥。

// LtpaToken2 针对 WebSphere Application Server V5.1.0.2（对于 z/OS®）和 V5.1.1（对于分布式系统）以及更高版本生成。

```

### 1 . 2 认证流程

```
1 当未经认证的用户对 WebSEAL 受保护资源发出请求时，WebSEAL 将首先确定是否提供了 LTPA Cookie。
	- 如果提供了 LTPA Cookie，那么它将验证此 Cookie 的内容，并在验证成功后根据此 Cookie 中包含的用户名和到期时间创建新会话。
	- 如果未提供 LTPA Cookie，那么 WebSEAL 将继续使用其他已配置的认证机制对用户进行认证。
2 完成认证操作后，将在 HTTP 响应中插入新的 LTPA Cookie，并将其传递回客户机以供其他支持 LTPA 的认证服务器使用。
```

![](C:\Users\10169\OneDrive\笔记文档\图片文件\其他\LTPA\ltpa_simple_client_server.jpg)



### 1 . 3 Token 的不同区别

```
LtpaToken
LtpaToken 用于与 WebSphere Application Server 的前发行版进行互操作。此令牌仅包含认证身份属性。
LtpaToken 针对 WebSphere Application Server V5.1.0.2 之前的发行版（对于 z/OS®）或 V5.1.1（对于分布式系统）生成。

LtpaToken2
LtpaToken2 包含更强的加密功能，并且您能够向令牌添加多个属性。此令牌包含认证身份和其他信息（例如，属性）。属性用于联系原始登录服务器和唯一高速缓存密钥。如果在确定唯一性时要考虑除身份以外的其他内容，还将使用属性来查找主题。
```



##  二  WebSphere  功能 ：

```
支持在一个因特网域中的一组web 服务器间使用单一登录的认证策略 ，通过密码术 可支持分布式环境的安全性 ，web 用户只需对  WebSphere Application Server 或 Domino 服务器认证一次 ，认证将会通过服务器进行共享 
```

### 2 . 1  WebSphere 部分

```
本部分描述适用于已实施WebSphere系列产品应用和Domino平台应用，或WebSphere与Domino之间已完成单点登录。在这样的环境中与构异系统实现单点登录。

一个通过有效的LTPA Cookie能够在同一个认证域中所有服务器被自动认证。此Cookie中包含认证信息和时间戳。这些信息通过共享的3DES Key进行了bis 加密。使用公共密钥/私有密钥进行签名。
LTPA Cookie通过3DES密钥使用DESede/ECB/PKCS5P进行加密。此密钥也是采用DESede/ECB/PKCS5P进行加密，加密后再使用提供的密码再进行SHA Hash，生成24个字节的密钥，再进行Base64编码。
 
如果你要解析LTPA Token，先得使用Key的密码，生成3DES密钥；再使用3DES密钥解密Token Cookie。也可以使用公共/私有密钥来签名或验证LTPA Cookie。
```

### 2 . 2 LTPA 的使用

```
服务器配置好 LTPA 认证方式 ，用户通过浏览器成功登陆 ，服务器发送一个包含 LTPA Token 的  session cookie 给浏览器
```

### 2 . 3 LTPA 的特点

```
一个 有效的 LTPA Cookie 能够在同一个认证域中所有的服务器自动认证 ，将其中的认证信息和时间戳 通过3DES key 进行 bis 加密 ，使用公共秘钥和私有秘钥进行签名

```

### 2 . 4 LTPA  令牌策略属性  
```java
 - >  title : 必须 ，策略的标题 ，字符串  
 - >  description  : 非必须 , 对策略的描述 ，字符串
 - >  key : LTPA 秘钥 ，必须 ，用于生成 LTPA 令牌 的LTPA 秘钥名称 -  包括 ：
     -- >  my-ltpa-key （策略默认为 1.0），my-ltpa-key:2.0.0 （使用特定版本），my-ltpa-key:latest （ 最新版本 ）
     -- >  authenticatedUserName ： 已认证用户名 
     -- >  tokenVersion ： 令牌版本
     -- >  tokenOutput ： 令牌用于放置已生成令牌的位置 （cookie 头 ，WSSec 头 ）
     -- >  tokenExpiry ： 整数 ，令牌过期时间  
```

### 2 . 5  WebSphere LTPA 生成原理

```
首先，这个 cookie 由以下部分组成，以%进行分隔：
    - 用户信息，格式为u:user\:<RealmName>/<UserDN>，
    	如：u:user\:VGOLiveRealm/CN=squallzhong,O=VGOLive Technology
    - 过期时间
    - 签名信息，如：
u:user\:VGOLiveRealm/CN=squallzhong,O=VGOLive Technology%1301558320666%Cy2CAeru5kEElGj0hrvYsKW2ZVsvvcu6Un573aeX55OO4G3EMYWc0e/ZbqDp1z7MS+dLzniuUH4sYWCMpnKdm7ZGabwmV+WcraBl+y+yzwcl722gHVMOnDZAW7U3jEay9Tk2yG4yXkMWU+617xndpVxke2jtS5wIyVVM3q7UDPw=
```

### 2 . 6  WebSphere 异构系统所需信息

```java
从WebSphere系统中导出ltpa的key文件，使用文本文件打开

com.ibm.websphere.CreationDate=Thu Mar 31 11\:08\:09 GMT+08\:00 2011 com.ibm.websphere.ltpa.version=1.0 com.ibm.websphere.ltpa.3DESKey=7dH4i81YepbVe+gF9XVUzE4C1Ca5g6A4Q69OFobJV9g\= com.ibm.websphere.CreationHost=wasserver com.ibm.websphere.ltpa.PrivateKey=N3bnOE1IbiXNsHXxxemC98iiCnmtw3JUuQvdFjEyh9r2gu+FlQRmG8xp5RBltqc6raI4EgYFhTr+t5/tmRQrFqfNKgvujeJZODeCspohi1V4C0qit7DOoqD9xOOn9Rzdb4PIuJM3ekwuBiZZYTYu7q0TANDygc7VbmwoD3xMPCk5svyvFJ/VshPyg5f7Q+VNM8dlIitU4gK9Qp8VZEqjGoXsYYzYYTQgnwAVtR2GfZtXKlf24EPXSkgUz9j8FwTvcylcKwjS22d6eVjciyAzInnxPqxE2iMRPEFDatHZFox3flsqBswmeDQrAGv8zIiffgP1DLKdjozUyAG+50v97xx7u1RtIrB4B01ik8DuLhw\= com.ibm.websphere.ltpa.Realm=VGOLiveRealm com.ibm.websphere.ltpa.PublicKey=AM04If2+ElGSyVRF0ZEesgvC59vGw8gSIfptjfoXj8iz4C7Ip/KVAu2PDkpQi3LUN/FgVF696tmsegBThks9rmMMHzOix/vGP2721dQZKbD7plOLdWtiY2AYZChsBVkOF26DfiWJ6euxD+a+KNcrfDnu2AXRC/tKncIUJV4LbeJdAQAB


- 所使用的DNS域，如：vgolive.com
- 过期时间(分钟)，如：30
- LTPA 3DESKey 密钥及密钥的保护密码
- Base DN，如：O=VGOLive Technology或DC=vgolive,DC=com

注意 : 
// 1 在3DESKey中的反斜杠只是为了在JAVA中可解释等于号，所以正确的3DESKey为7dH4i81YepbVe+gF9XVUzE4C1Ca5g6A4Q69OFobJV9g=

// 2 用户名可以通过Base Dn验证字进行拼接，如异构系统中用户名为SquallZhong，相应在Domino中的DN就是CN=SquallZhong,O=DigiWin。此类情况仅限于LDAP中的CN与异构系统中的用户名一致。如果不一致，需要异构系统做用户对应
```

## 三  Domain 部分

```
本部分的描述仅适用于单一的Domino平台应用与构异系统实现单点登录。
```


### 3 . 1  LTPA Cookie   

```java
- >  LTPA token 版本（4字节）
- >  创建时间（8字节）
- >  过期时间（8字节）
- >  用户名（可变长度）
- >  Domino LTPA 密钥（20字节）

|- 在与 Domino 做 SSO 的时候，会使用 LTPA Token的认证方式，本文描述它的生成原理，通过它我们可以自己编码生成身份认证的 cookie，实现 SSO。
	首先，这个 cookie 由以下部分组成：
    - LTPA token 版本（4字节）
    - 创建时间（8字节）
    - 过期时间（8字节）
    - 用户名（可变长度）
    - Domino LTPA 密钥（20字节）
    
接下来分别说明各部分的具体内容：
    - LTPA token 版本目前 Domino 只有一种值：0x0001
    - 创建时间为以十六进制方式表示的Unix time，例如:2009-04-09 13:52:42 (GMT +8) = 1239256362 = 49DD8D2A。
    - 过期时间=创建时间 + SSO 配置文档的过期时间（LTPA_TokenExpiration域）
    - 用户名为 Names 中用户文档的FullName域值；如：Squall Zhong/Digiwin
    - Domino LTPA 密钥通过 Base64编码后，保存在 SSO 配置文档的LTPA_DominoSecret域中


// 当然不能将密钥直接发送给浏览器，所以将上述部分合并起来（如上图），计算 SHA-1 校验和。

	然后用 SHA-1 校验和替换掉 Domino LTPA 密钥，最后再将内容通过 Base64 编码，形成最终的 cookie 发送给浏览器（如上图）。这样如果 cookie 中的任何内容被修改，校验和就不对了，达到了防篡改的效果。所以最终LTPA Cookie所得到的值为以下公式组成：
	SHA-1=LTPA版本号+创建时间+过期时间+用户名+Domino LTPA 密钥
	LTPA Cookie= Base64(LTPA版本号+创建时间+过期时间+用户名+SHA-1)
        
        
        
```

### 3 . 2  Domain 	异构系统所需信息

```
    - Domino 所使用的DNS域，如：vgolive.com
    - 过期时间(分钟)，如：30
    - Domino LTPA 密钥
    - Domino验证字名称，如：/O=VGOLive Technology
    
注：用户名可以通过Domino验证字进行拼接，如异构系统中用户名为SquallZhong，相应在Domino中的DN就是CN=SquallZhong/O=VGOLive Technology。此类情况仅限于Domino中的CN与异构系统中的用户名一致。如果不一致，需要异构系统做用户对应
```



### 3 . 3 加密方式
    - >  使用 3DES 秘钥 进行 DESede/ECB/PKCS5P 加密
    - >  秘钥采用 DESede/ECB/PKCS5P进行加密
    - >  通过提供的密码进行 SHA Hash
    - >  对生成的24字节秘钥 进行 Base64 编码



### 3 . 4 LDAP 目录访问连接池

```

```



## 附录 

### #1 准备工作

```
> 搭建服务器 : WebSEAL
```

### # 2 LTPA Token 组成

```java
// 一个 LTPA 的格式
[token] = BASE64([header][creation time][expiration time][username][SHA-1 hash])
    
Header:  LtpaToken 版本（长度4），Domino的固定为[0x00][0x01][0x02][0x03]
Creation time: 创建时间戳（长度8），格式为Unix time比如[2010-03-12 00:21:49]为4B99189D
expiration time：过期时间戳（长度8） 同上
username： 用户名（长度不定）
SHA-1 hash：SHA-1校验和（长度20）
	- 由前面所说的密钥和其余的Token资料合并而成，合成公式如下
	- [SHA-1 hash] = SHA-1([header][creation time][expiration time][username][shared secret]) 
```

![](C:\Users\10169\OneDrive\笔记文档\图片文件\其他\LTPA\001.gif)

![](C:\Users\10169\OneDrive\笔记文档\图片文件\其他\LTPA\002.gif)

### # 3 Domain 解析 Token

```
1 Base64解码LtpaToken。

2 截取最前面20字节，最后面20字节，中间部分就是用户名。如果用户名在本系统中不正确，返回无效的LtpaToken。

3 截取最后面20字节，是SHA-1校验和。用Token中的其余部分和本系统中的密钥生成新的SHA-1校验和，如果2个校验和不匹配。返回无效的LtpaToken。

4 当前服务器时间必须大于创建时间，小于失效时间。否则返回无效的LtpaToken。

5 最后解析通过了，完成用户的登录

```

### # 4 Java 实现 LTPA Cookie 解析

```java
 
        // LTPA 3DES 密钥 
        String ltpa3DESKey = "7dH4i81YepbVe+gF9XVUzE4C1Ca5g6A4Q69OFobJV9g="; 
        // LTPA 密钥密码 
        String ltpaPassword = "Passw0rd"; 
        try { 
			// 获得加密key 
            byte[] secretKey = getSecretKey(ltpa3DESKey, ltpaPassword); 
            // 使用加密key解密ltpa Cookie 
            String ltpaPlaintext = new String(decryptLtpaToken(tokenCipher, 
                    secretKey)); 
            displayTokenData(ltpaPlaintext); 
        } catch (Exception e) { 
            System.out.println("Caught inner: " + e); 
        } 
 
    //获得安全Key 
    private static byte[] getSecretKey(String ltpa3DESKey, String password) 
            throws Exception { 
        // 使用SHA获得key密码的hash值 
        MessageDigest md = MessageDigest.getInstance("SHA"); 
        md.update(password.getBytes()); 
        byte[] hash3DES = new byte[24]; 
        System.arraycopy(md.digest(), 0, hash3DES, 0, 20); 
        // 使用0替换后4个字节 
        Arrays.fill(hash3DES, 20, 24, (byte) 0); 
        // BASE64解码 ltpa3DESKey 
        byte[] decode3DES = Base64.decodeBase64(ltpa3DESKey.getBytes()); 
        // 使用key密码hash值解密已Base64解码的ltpa3DESKey 
        return decrypt(decode3DES, hash3DES); 
    } 
    //解密LtpaToken 
    public static byte[] decryptLtpaToken(String encryptedLtpaToken, byte[] key) 
            throws Exception { 
        // Base64解码LTPAToken 
        final byte[] ltpaByteArray = Base64.decodeBase64(encryptedLtpaToken 
                .getBytes()); 
        // 使用key解密已Base64解码的LTPAToken 
        return decrypt(ltpaByteArray, key); 
    } 
    // DESede/ECB/PKC5Padding解方法 
    public static byte[] decrypt(byte[] ciphertext, byte[] key) 
            throws Exception { 
        final Cipher cipher = Cipher.getInstance("DESede/ECB/PKCS5Padding"); 
        final KeySpec keySpec = new DESedeKeySpec(key); 
        final Key secretKey = SecretKeyFactory.getInstance("TripleDES") 
                .generateSecret(keySpec); 
        cipher.init(Cipher.DECRYPT_MODE, secretKey); 
        return cipher.doFinal(ciphertext); 
    } 

```

### # 5 	

```java
 /** 
02     * 为指定用户创建有效的LTPA Token.创建时间为<tt>now</tt>. 
03     * 
04     * @param username 
05     *            - 用户名，注：使用用户全称，如：CN=SquallZhong/O=VGOLive Technology 
06     * @param creationTime 
07     *            - 创建时间 
08     * @param durationMinutes 
09     *            - 到期时间，单位：分钟 
10@param ltpaSecretStr 
11     *            - Domino Ltpa 加密字符串 
12     * @return - 返回已Base64编码的Ltpa Cookie. 
13     * @throws NoSuchAlgorithmException 
14     * @throws Base64DecodeException 
15     */
16    public static String createLtpaToken(String username, 
17            GregorianCalendar creationTime, int durationMinutes, 
18            String ltpaSecretStr) throws NoSuchAlgorithmException { 
19        // Base64解码ltpaSecretStr 
20        byte[] ltpaSecret = Base64.decodeBase64(ltpaSecretStr.getBytes()); 
21        // 用户名字节数组 
22        byte[] usernameArray = username.getBytes(); 
23        byte[] workingBuffer = new byte[preUserDataLength 
24                + usernameArray.length + ltpaSecret.length]; 
25 
26        // 设置ltpaToken版本至workingBuffer 
27        System.arraycopy(ltpaTokenVersion, 0, workingBuffer, 0, 
28                ltpaTokenVersion.length); 
29        // 获得过期时间，过期时间=当前时间+到期时间(分钟) 
30        GregorianCalendar expirationDate = (GregorianCalendar) creationTime 
31                .clone(); 
32        expirationDate.add(Calendar.MINUTE, durationMinutes); 
33 
34        // 转换创建时间至16进制字符串 
35        String hex = dateStringFiller 
36                + Integer.toHexString( 
37                        (int) (creationTime.getTimeInMillis() / 1000)) 
38                        .toUpperCase(); 
39        // 设置创建时间至workingBuffer 
40        System.arraycopy(hex.getBytes(), hex.getBytes().length 
41                - dateStringLength, workingBuffer, creationDatePosition, 
42                dateStringLength); 
43 
44        // 转换过期时间至16进制字符串 
45        hex = dateStringFiller 
46                + Integer.toHexString( 
47                        (int) (expirationDate.getTimeInMillis() / 1000)) 
48                        .toUpperCase(); 
49        // 设置过期时间至workingBuffer 
50        System.arraycopy(hex.getBytes(), hex.getBytes().length 
51                - dateStringLength, workingBuffer, expirationDatePosition, 
52                dateStringLength); 
53 
54        // 设置用户全称至workingBuffer 
55        System.arraycopy(usernameArray, 0, workingBuffer, preUserDataLength, 
56                usernameArray.length); 
57 
58        // 设置已Base64解码ltpaSecret至workingBuffer 
59        System.arraycopy(ltpaSecret, 0, workingBuffer, preUserDataLength 
60                + usernameArray.length, ltpaSecret.length); 
61        // 创建Hash字符串 
62        byte[] hash = createHash(workingBuffer); 
63 
64        // ltpaToken版本+开始时间(16进制)+到期时间(16进制)+用户全名+SHA-1(ltpaToken版本+开始时间(16进制)+到期时间(16进制)+用户全名) 
65        byte[] outputBuffer = new byte[preUserDataLength + usernameArray.length 
66                + hashLength]; 
67        System.arraycopy(workingBuffer, 0, outputBuffer, 0, preUserDataLength 
68                + usernameArray.length); 
69        System.arraycopy(hash, 0, outputBuffer, preUserDataLength 
70                + usernameArray.length, hashLength); 
71        // 返回已Base64编码的outputBuffer 
72        return new String(Base64.encodeBase64(outputBuffer)); 
73    } 
74…... 
```

### # 6 通过F5 BIG-IP创建Domino LTPAToken

```java
when RULE_INIT {
 set cookie_name "LtpaToken"           # Don't change this
 set ltpa_version "\x00\x01\x02\x03"   # Don't change this
 set ltpa_secret "b64encodedsecretkey" # Set this to the LTPA secrey key from your Lotus Domino LTPA configuration
 set ltpa_timeout "1800"               # Set this to the timeout value from your Lotus Domino LTPA configuration
}

when HTTP_REQUEST {
 #
 # Do your usual F5 HTTP authentication here
 #

 # Initial values
 set creation_time_temp [clock seconds]
 set creation_time [format %X $creation_time_temp]
 set expr_time_temp [expr { $creation_time_temp + $::ltpa_timeout}]
 set expr_time [format %X $expr_time_temp]
 set username [HTTP::username]
 set ltpa_secret_decode [b64decode $::ltpa_secret]

 # First part of token
 set cookie_data_raw {}
 append cookie_data_raw $::ltpa_version
 append cookie_data_raw $creation_time
 append cookie_data_raw $expr_time
 append cookie_data_raw $username
 append cookie_data_raw $ltpa_secret_decode

 # SHA1 of first part of token
 set sha_cookie_raw [sha1 $cookie_data_raw]

 # Final not yet encoded token
 set ltpa_token_raw {}
 append ltpa_token_raw $::ltpa_version
 append ltpa_token_raw $creation_time
 append ltpa_token_raw $expr_time
 append ltpa_token_raw $username
 append ltpa_token_raw $sha_cookie_raw

 # Final Base64 encoded token
 set ltpa_token_final [b64encode $ltpa_token_raw]

 # Insert the cookie
 HTTP::cookie insert name $::cookie_name value $ltpa_token_final
 }

 # Remove Authorization HTTP header to avoid using basic authentication
 if { [HTTP::header exists "Authorization"] } {
 HTTP::header remove "Authorization"
 }
}
```