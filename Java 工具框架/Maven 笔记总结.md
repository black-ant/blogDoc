# Maven

[TOC]



## 基础知识

### 常用命令

```java
> 打包
// mvn clean install -Dmaven.test.skip=true -Dmaven.javadoc.skip=true
// clean install -Dmaven.javadoc.skip=true
// 跳过 mvn [target] -Dcheckstyle.skip
// --> 跳过 license : mvn clean install -DskipTests license:format
//
// mvn clean install -Dmaven.test.skip=true -Dmaven.javadoc.skip=true -Dcheckstyle.skip -DskipTests=true
 clean install -DskipTests=true
// ? clean -> 
// ?  -Dmaven.test.skip=tru ->  跳过 test 测试
// ?  -Dmaven.javadoc.skip=true -> 跳过 javadoc
mvn install -Dmaven.test.skip=true -Dmaven.javadoc.skip=true
mvn install -X -Dmaven.test.skip=true -Dmaven.javadoc.skip=true
    
mvn assembly:assembly

// --> 清空新的项目
mvn clean

// --> 编译源代码
mvn compile 

// --> 构建项目
mvn package

// --> 打包项目并且部署到本地资源库
mvn install
mvn install:install-file -Dfile=algs4.jar
mvn install:install-file -Dfile=algs4.jar -DgroupId=lib.algs4 -DartifactId=algs4 -Dversion=4.0 -Dpackaging=jar　　  
    
// --> 打包并且上传到私服
mvn deploy
mvn clean deploy -Dmaven.test.skip=true -Dmaven.javadoc.skip=true -Dcheckstyle.skip -DskipTests=true
    
// --> 本地加入jar
mvn install:install-file  
-DgroupId=包名  
-DartifactId=项目名  
-Dversion=版本号  
-Dpackaging=jar  
-Dfile=jar文件所在路径  
    
// --> 下载源码
mvn dependency:sources

// --> 下载注释 
mvn dependency:resolve -Dclassifier=javadoc

// --> 部署到Tomcat 
mvn tomcat:redeploy

// --> 跳过测试阶段
mvn package -DskipTests

// --> 临时性跳过测试代码的编译
mvn package -Dmaven.test.skip=true
    
// --> 指定测试类
mvn test -Dtest=RandomGeneratorTest

// --> 以Random开头，Test结尾的测试类
mvn test -Dtest=Random*Test

// --> 用逗号分隔指定多个测试用例 
mvn test -Dtest=ATest,BTest

// --> 运行 checkstyle
mvn checkstyle:checkstyle
    
    
// --> 打包本地
mvn install:install-file -Dfile=xxx.jar -DgroupId=aaa -DartifactId=bbb -Dversion=1.0.0 -Dpackaging=jar
    
// --> 打包后运行    

nohup java -jar para-sdk-test-0.0.1.jar --spring.profiles.active=dev --server.port=18002 > para-sdk-test.log &
  
ps -ef|grep test 
nohup java -jar para-sdk-test-0.0.1.jar --server.port=18002 > common-test.log 2>&1 &
    
nohup java -jar para-msg-server-5.2.0.jar --server.port=17002 > para-msg-test.log 2>&1 &    
// 查看 
cd /opt/paraview/cic/code    
ps -ef|grep para-msg-server 
    
// --> 查看依赖树
mvn dependency:tree    
    
// --> 更新版本 
mvn versions:set -DnewVersion=1.0.1-SNAPSHOT    

```

### 使用根路径

```
 <properties>
    <rootpom.basedir>${basedir}/../..</rootpom.basedir>
  </properties>
  
	<properties>
		<rootpom.basedir>${basedir}/..</rootpom.basedir>
	</properties>
	
```



### Maven pom.xml

```xml
<!-- 基本结构 -->
<!-- -->
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
</project>

${basedir} 项目根目录
${project.build.directory} 构建目录，缺省为target
${project.build.outputDirectory} 构建过程输出目录，缺省为target/classes
${project.build.finalName} 产出物名称，缺省为${project.artifactId}-${project.version}
${project.packaging} 打包类型，缺省为jar
${project.xxx} 当前pom文件的任意节点的内容


<!-- 典型的 依赖结构 -->
GroupId / artifactId / version 
packaging : Maven 项目的打包方式
classifier : 用于构建输出附属组件
scope : 依赖的范围
optional : 标记依赖是否可选
exclusions : 用来排除传递性依赖

<!-- dependency 依赖范围 -->
< SCOPE >
	1 complie : 会在编译 ,测试 ,允许三个阶段均有效
	2 test : 该范围在测试的时候才有效 
	3 provided : 编译器 , 测试有效 ,运行时无效
	4 runtime : 运行时有效 ,编译 测试时无效(例如JDBC)
	5 system : 等同于 provided , 仅用于手动编辑本地包
</SCOPE>
```



### 组件含义

```
> pluginManagement : 公用plugins ， 子组件可以使用
```

### 依赖关系

```
> 传递性依赖 : 当依赖没有声明依赖范围的时候 ,默认是 complie , 当作用域都处在 complie 时期时(即当项目处在编译器时) , 依赖可以传递
	- scope 同样可以控制依赖传递的范围 , 当 scope 为 test 或者 provide 时 ,只有这2个时期依赖才会传递
	
> 依赖调节 : 基于传递性依赖 ,导致同一个依赖回重复 , 此时存在依赖调节 , 该操作有以下几个原则:
	- 1 路径最近者优先
	- 2 第一声明者优先
	
> 可选依赖 : <optional> true
	- A - B -C(可选)
	- A 依赖 B , B 依赖于 C , 当C 为可选依赖的时候 ,并不是传递到 A
	? 可选依赖应该避免使用 , 只有当一个项目实现了多个特性的时候 ,才需要实现可选依赖

> 排除依赖 : <exclusion>

	
	
```

### Maven 仓库

```
> 路径的生成步骤:
	- 根据 GroupId 准备路径 , formatAsDirectory 将 groupid 的句点分隔符转换为路径
		: org/test/
	- 根据 artifactId 准备路径 ,在上述基础上加上 artifactId + "一个路径分隔符"
		: org/test/gang/
	- 使用版本信息 , 加入 version 和分隔符
		: org/test/gang/1.0.0
	- 构建artifactId全名
		: : org/test/gang/1.0.0/test.jar
	- 加入 classifier
    	: org/test/gang/1.0.0/test
    - 如果有extension :
    	
    	
 >  Maven 仓库类型
 	- 简单区分 : 远程仓库 - 本地仓库
 	- 特殊划分 :
 		- 中央仓库 : Maven 核心仓库
 		- 私服 : 局域网私有仓库 , 缓存核心仓库以及提交
 			: 节省外网宽带
 			: 加速Maven构建 
 			: 部署第三方控件
 			: 提高稳定性 , 增强控制
 			: 降低中央仓库负载
 			: 
 			
 	- 部署到远程仓库:
    	- 
```

### 生命周期

```
> Maven 的生命周期用于处理项目的状态
> Maven 提供了三套生命周期
	- clean : 
	- default
	- site
```



### Setting 解析

```java
> proxies : 代理列表
	> proxy : 代理元素基本信息
		- id : 唯一id
		- active : 代理是否激活
		- protocol : 代理服务协议
		- host : 代理主机
		- port : 代理端口空
		- username 
		- password
		- nonProxyHosts : 不被代理列表
		
> servers : 服务器列表
	// 当存在私服的时候 , 需要通过server 标签给出授权信息
	> server : 服务器连接信息
		- id :唯一主键
		- username
		- password
		- privateKey : 服务器鉴权私钥
		- passphrase : 鉴权私钥密码
		- filePermissions : 文件创建权限 , 以 Linux 权限为标准
		- directoryPermissions : 目录被创建时权限
		- configuration : 传输层额外配置
		
> mirrors : 镜像列表
	> mirror : 给定镜像下载信息
		- id : 镜像唯一标识
		- name : 镜像名称
		- url : 镜像URL , 优先级大于默认服务器URL
		- mirrorOf : 被镜像的服务器ID , 对应这中央仓库id
		
		
> profiles : 
? settings.xml中的profile元素是pom.xml中profile元素的裁剪版本
? 如果一个settings中的profile被激活，它的值会覆盖任何其它定义在POM中或者profile.xml中的带有相同id的profile
	> profile
		- id : 唯一 id
		- activation : 开启命令集
			- activeByDefault : 默认是否激活
			- jdk
			- os : 匹配匹配的操作系统
				- name : 操作系统名称
				- family : 操作系统家族
				- arch : 操作系统架构
				- version : 操作系统版本
			- property : Maven检测到某一个属性 , 拥有对应的名称和值 , 激活
				- name / version
			- file : 检测一个文件名判断是否激活	
		- properties : 激活后可以访问的属性
			- 属性列表
		- repositories : 远程仓库列表
			- repository : 远程仓库信息
        		- id : 远程仓库唯一 id
        		- name : 远程仓库名称
        		- release: 处理远程仓库的版本
        			- enabled : 
        			- updatePolicy : 更新频率
        			- checksumPolicy : 
```



#### maven 配置多节点下载的正确方式

```xml
<!-- step 1 : 配置 profile 多个节点 --> 

<profiles>
    <profile>
        <id>boundlessgeo</id>
        <repositories>
            <repository>
                <id>boundlessgeo</id>
                <url>https://repo.boundlessgeo.com/main/</url>
                <releases>
                    <enabled>true</enabled>
                </releases>
                <snapshots>
                    <enabled>true</enabled>
                    <updatePolicy>always</updatePolicy>
                </snapshots>
            </repository>
        </repositories>
    </profile>
    <profile>
        <id>aliyun</id>
        <repositories>
            <repository>
                <id>aliyun</id>
                <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
                <releases>
                    <enabled>true</enabled>
                </releases>
                <snapshots>
                    <enabled>true</enabled>
                    <updatePolicy>always</updatePolicy>
                </snapshots>
            </repository>
        </repositories>
    </profile>
    <profile>
        <id>maven-central</id>
        <repositories>
            <repository>
                <id>maven-central</id>
                <url>http://central.maven.org/maven2/</url>
                <releases>
                    <enabled>true</enabled>
                </releases>
                <snapshots>
                    <enabled>true</enabled>
                    <updatePolicy>always</updatePolicy>
                </snapshots>
            </repository>
        </repositories>
    </profile>
</profiles>
    
<!-- step 激活节点 -->
<activeProfiles>
	<activeProfile>boundlessgeo</activeProfile>
	<activeProfile>aliyun</activeProfile>
	<activeProfile>maven-central</activeProfile>
</activeProfiles>



```

### Maven 多仓库方案二 

```xml
    <repositories>
        <repository>
            <!-- id必须唯一 -->
            <id>jboss-repository</id>
            <!-- 见名知意即可 -->
            <name>jboss repository</name>
            <!-- 仓库的url地址 -->
            <url>http://repository.jboss.org/nexus/content/groups/public-jboss/</url>
        </repository>
        <repository>
            <!-- id必须唯一 -->
            <id>aliyun-repository</id>
            <!-- 见名知意即可 -->
            <name>aliyun repository</name>
            <!-- 仓库的url地址 -->
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        </repository>
    </repositories>
```



#### 一个 Maven 项目的执行过程

```
> clean : clean
> resource : resource
> compiler : compile
> resource : testResource
> compiler : testCompile
```



## Maven 知识点

```java
> maven 本地资源库 ：存储项目的依赖库

> Maven 中央储存库 ：网络上的下载所有项目依赖库的默认位置
	// http://search.maven.org/
	// http://repo1.maven.org/maven2/

> MAVEN 库的流程
	> 在 Maven 本地库搜索
	> 在中央仓库中搜索
	> 在 Maven 远程仓库搜索

> 定制本地 JAR 库包
	mvn install:install-file -Dfile=c:\kaptcha-2.3.jar -DgroupId=com.google.code 
-DartifactId=kaptcha -Dversion=2.3 -Dpackaging=jar

// 首先 通过 mvn install 安装库 ， 然后pom 调用即可
<dependency>
      <groupId>com.google.code</groupId>
      <artifactId>kaptcha</artifactId>
      <version>2.3</version>
 </dependency>


> Maven 外部依赖
      <dependency>
         <groupId>ldapjdk</groupId>
         <artifactId>ldapjdk</artifactId>
         <scope>system</scope>
         <version>1.0</version>
         <systemPath>${basedir}\src\lib\ldapjdk.jar</systemPath>
      </dependency>


> lastUpdated.properties
	// 该文件可能会导致jar 包在下载失败后不再下载

> 加载指定版本以上 
	// _maven.repositories
	// 类似于 ：flex-messaging-opt-4.0.0.pom>Nexus=
	
> maven 的 多个阶段
	validate - 验证项目是否正确，并提供所有必要信息
	compile - 编译项目的源代码
	test - 使用合适的单元测试框架测试编译的源代码。这些测试不应要求打包或部署代码
	package - 获取已编译的代码并将其打包为可分发的格式，例如JAR。
	verify - 对集成测试结果进行任何检查，以确保满足质量标准
	install - 将软件包安装到本地存储库中，以便在本地用作其他项目的依赖项
	deploy - 在构建环境中完成，将最终包复制到远程存储库以与其他开发人员和项目共享。
	
> _remote.properties
	maven 远程私服下载属性
	
```

#### Maven 仓库的匹配等级

```
1、在mirrorOf与repositoryId相同的时候优先是使用mirror的地址
2、mirrorOf等于*的时候覆盖所有repository配置
3、存在多个mirror配置的时候mirrorOf等于*放到最后
4、只配置mirrorOf为central的时候可以不用配置repository


proxy是服务器不能直接访问外网时需要设置的代理服务，不常用
server是服务器要打包上传到私服时，设置私服的鉴权信息
repository是服务器下载jar包的仓库地址
mirror是用于替代仓库地址的镜像地址
```

#### Mirror 分析

```xml
mirror 用于配置额外的镜像地址，用于快速下载

        <!--给定仓库的下载镜像。  -->  
        <mirror>  
            <!--该镜像的唯一标识符。id用来区分不同的mirror元素。  -->  
            <id>planetmirror.com</id>  
            <!--镜像名称  -->  
            <name>PlanetMirror Australia</name>  
            <!--该镜像的URL。构建系统会优先考虑使用该URL，而非使用默认的服务器URL。  -->  
            <url>http://downloads.planetmirror.com/pub/maven2</url>  
            <!--被镜像的服务器的id。例如，如果我们要设置了一个Maven中央仓库（http://repo1.maven.org/maven2）的镜像，-->  
            <!--就需要将该元素设置成central。这必须和中央仓库的id central完全一致。 -->  
            <mirrorOf>central</mirrorOf>  
        </mirror>  


>  <mirrorOf>*</mirrorOf>   :  匹配所有的库（repository）
>  <mirrorOf>repo3</mirrorOf>  : 匹配 id 为 repo3 的库，


mirror相当于一个拦截器，它会拦截maven对remote repository的相关请求，把请求里的remote repository地址，重定向到mirror里配置的地址

repository 是远程仓库，通过 mirror 对其 filter 并且 gateway
```

#### maven 查看依赖树

```
mvn -X dependency:tree>tree.txt
mvn dependency:tree
```

#### maven 查看已解析依赖

```
mvn dependency:list
```

#### maven 分析依赖关系

```
mvn dependency:analyze
```



#### maven 的依赖关系

```
> Maven 的依赖是可以传递的 ,当你导入一个Jar 包的时候 ,并不是说把其他的第三方的包直接导入 , 而是通过pom 去仓库查询 , 当远程仓库没有的时候 ,这个过程就会变得很复杂

> 当本地的 install 的时候 , 项目可能会不易导入依赖
```

#### maven  远程仓库部署 jar

```xml
<!-- step 1 : 链接到远程仓库 -->
<!-- settings -->
<?xml version="1.0" encoding="utf-8"?>

<servers> 
  <server> 
    <id>mycompany-repository</id>  
    <username>jvanzyl</username>  
    <!-- Default value is ~/.ssh/id_dsa -->  
    <privateKey>/path/to/identity</privateKey> (default is ~/.ssh/id_dsa) 
    <passphrase>my_key_passphrase</passphrase> 
  </server> 
</servers>

<!-- servers example 2 -->
<servers> 
    <server> 
        <id>snapshots</id> 
        <username>server account</username> 
        <password>server password</password> 
    </server>
</servers>


<!-- 要将jar部署到外部存储库，必须在pom.xml中配置存储库url，并在settings.xml中配置用于连接到存储库的身份验证信息。-->
<?xml version="1.0" encoding="utf-8"?>

<distributionManagement> 
  <repository> 
    <id>mycompany-repository</id>  
    <name>MyCompany Repository</name>  
    <url>scp://repository.mycompany.com/repository/maven2</url> 
  </repository> 
</distributionManagement>

<!-- example 2 -->
<distributionManagement>
    <repository>
        <id>snapshots</id>
        <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
    </repository>
    <snapshotRepository>
        <id>snapshots</id>
        <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
    </snapshotRepository>
</distributionManagement>




<!-- 提交 -->
mvn clean package deploy
```

#### relativePath 指定路径

```
<relativePath>../parent/pom.xml</relativePath>

 <parent>
    <groupId>com.mycompany.app</groupId>
    <artifactId>my-app</artifactId>
    <version>1</version>
    <relativePath>../parent/pom.xml</relativePath>
  </parent>

> 该路需要首先查找的路径 , 该路径的顺序为 :
relativePath -> 本地库 -> 远程库

> 此配置是为了解决 本地库

```

### Maven Springboot 打包包含 jar

```xml
<build>  

    <plugins>  

        <plugin>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-maven-plugin</artifactId>  
            <configuration>  
                <!-- 指定SpringBoot程序的main函数入口类 -->
                <mainClass>com.redsoft.epip.EPIPApplication</mainClass>  
            </configuration>  

            <executions>  
                <execution>  
                    <goals>  
                        <goal>repackage</goal>  
                    </goals>  
                </execution> 
            </executions>  
        </plugin>  

        <plugin>  
            <artifactId>maven-compiler-plugin</artifactId>  
            <configuration>  
                <source>1.8</source>  
                <target>1.8</target>  
                <encoding>UTF-8</encoding>  
                <compilerArguments>  
                    <!-- 打包本地jar包 -->
                    <extdirs>${project.basedir}/lib</extdirs>  
                </compilerArguments>  
            </configuration>  
        </plugin>  
    </plugins>  

    <!-- 打包所有jar包 -->
    <resources>  
        <resource>  
            <directory>lib</directory>  
            <targetPath>BOOT-INF/lib/</targetPath>  
            <includes>  
                <include>**/*.jar</include>  
            </includes>  
        </resource>  

    <!-- 某些情况下，打包后运行不起来需要打开注释 -->
    <!-- <resource>
        <directory>src/main/resources</directory>
        <targetPath>BOOT-INF/classes/</targetPath>
    </resource> -->
    </resources>  

</build>  
```



## Maven Plugins

#### maven-assembly-plugin

```xml
<?xml version="1.0" encoding="utf-8"?>

<plugin> 
  <groupId>org.apache.maven.plugins</groupId>  
  <artifactId>maven-assembly-plugin</artifactId>  
  <executions> 
    <execution> 
      <id>source-release-assembly</id>  
      <phase>none</phase> 
    </execution>  
    <execution> 
      <id>source-release-assembly-no-eclipse-libs</id>  
      <phase>package</phase>  
      <goals> 
        <goal>single</goal> 
      </goals>  
      <configuration> 
        <runOnlyAtExecutionRoot>true</runOnlyAtExecutionRoot>  
        <descriptors> 
          <descriptor>src/assemble/${sourceReleaseAssemblyDescriptor}.xml</descriptor> 
        </descriptors>  
        <tarLongFileMode>gnu</tarLongFileMode> 
      </configuration> 
    </execution> 
  </executions> 
</plugin>

```



#### maven-assembly-plugin : 打包时添加依赖

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 打包时添加依赖包 , 依赖包一同打包 -->
<plugin> 
  <artifactId>maven-assembly-plugin</artifactId>  
  <configuration> 
    <!--这部分可有可无,加上的话则直接生成可运行jar包-->  
    <!--<archive>-->  
    <!--<manifest>-->  
    <!--<mainClass>${exec.mainClass}</mainClass>-->  
    <!--</manifest>-->  
    <!--</archive>-->  
    <descriptorRefs> 
      <descriptorRef>jar-with-dependencies</descriptorRef> 
    </descriptorRefs> 
  </configuration> 
</plugin>

```

#### maven-jar-plugin : 打包成 Jar

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 打包成 Jar -->
<plugin> 
  <groupId>org.apache.maven.plugins</groupId>  
  <artifactId>maven-jar-plugin</artifactId> 
</plugin>


<?xml version="1.0" encoding="utf-8"?>

<plugin> 
  <groupId>org.apache.maven.plugins</groupId>  
  <artifactId>maven-jar-plugin</artifactId>  
  <version>3.1.0</version>  
  <configuration> 
    <archive> 
      <manifest> 
        <addClasspath>false</addClasspath> 
      </manifest>  
      <manifestEntries> 
        <commit>${idm.commit}</commit> 
      </manifestEntries> 
    </archive> 
  </configuration> 
</plugin>

```

#### maven-release-plugin : maven 版本管理插件

```xml
<?xml version="1.0" encoding="utf-8"?>

<plugin> 
  <groupId>org.apache.maven.plugins</groupId>  
  <artifactId>maven-release-plugin</artifactId>  
  <version>2.5.3</version>  
  <configuration> 
    <mavenExecutorId>forked-path</mavenExecutorId>  
    <autoVersionSubmodules>true</autoVersionSubmodules>  
    <tagNameFormat>para-idm-@{project.version}</tagNameFormat> 
  </configuration> 
</plugin>

```



#### modernizer-maven-plugin : 检测对旧版API的使用

```xml
<?xml version="1.0" encoding="utf-8"?>

<!-- Modernizer Maven插件可以检测对旧版API的使用，这些旧版API可以替代现代Java版本。这些现代的API通常比传统的API具有更高的性能，安全性和惯用性。-->

<plugin> 
  <groupId>org.gaul</groupId>  
  <artifactId>modernizer-maven-plugin</artifactId>  
  <version>1.6.0</version>  
  <configuration> 
    <javaVersion>${targetJdk}</javaVersion> 
  </configuration>  
  <executions> 
    <execution> 
      <id>modernizer-check</id>  
      <phase>verify</phase>  
      <goals> 
        <goal>modernizer</goal> 
      </goals> 
    </execution> 
  </executions> 
</plugin>

```



#### maven-enforcer-plugin jar 冲突解决工具

```xml
<?xml version="1.0" encoding="utf-8"?>

<plugin> 
  <groupId>org.apache.maven.plugins</groupId>  
  <artifactId>maven-enforcer-plugin</artifactId>  
  <version>1.4.1</version>  
  <executions> 
    <execution> 
      <id>enforce-maven</id>  
      <goals> 
        <goal>enforce</goal> 
      </goals>  
      <configuration> 
        <rules> 
          <requireMavenVersion> 
            <version>3.0</version> 
          </requireMavenVersion> 
        </rules> 
      </configuration> 
    </execution> 
  </executions> 
</plugin>

```



####  findbugs-maven-plugin : 代码静态分析工具

```xml
<?xml version="1.0" encoding="utf-8"?>

<plugin> 
  <groupId>org.codehaus.mojo</groupId>  
  <artifactId>findbugs-maven-plugin</artifactId> 
</plugin>

```

#### groovy-maven-plugin : groovy 脚本执行工具

```xml
<?xml version="1.0" encoding="utf-8"?>

<plugin> 
  <groupId>org.codehaus.gmaven</groupId>  
  <artifactId>gmaven-plugin</artifactId>  
  <version>1.5</version>  
  <dependencies> 
    <dependency> 
      <groupId>org.codehaus.groovy</groupId>  
      <artifactId>groovy-all</artifactId>  
      <version>1.5.8</version> 
    </dependency> 
  </dependencies>  
  <executions> 
    <execution> 
      <phase>generate-resources</phase>  
      <goals> 
        <goal>execute</goal> 
      </goals>  
      <configuration> 
        <source> <![CDATA[
						                import java.util.Date
						                import java.text.MessageFormat
						                def year = MessageFormat.format("{0,date,yyyy}", new Date()) 
						                project.properties['year'] = year
						                project.properties['snapshotOrRelease'] = project.version.endsWith("SNAPSHOT") ? "snapshot" : "release"
						                project.properties['licenseUrl'] = "http://esc.paraview.cn"
						                project.properties['site.deploymentBaseDir'] = 
						                project.properties['site.deploymentBaseUrl'] == null || !project.properties['site.deploymentBaseUrl'].startsWith('file:') ? project.properties['project.build.directory'] + "/generated-docs" : project.properties['site.deploymentBaseUrl'].substring(7)
						
						                if (!project.properties.containsKey('buildNumber'))
						                project.properties['buildNumber'] = ""
						           ]]> </source> 
      </configuration> 
    </execution> 
  </executions> 
</plugin>

```



####  maven-surefire-plugin :  单元测试工具 

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- surefire 插件用来在maven构建生命周期的test phase执行一个应用的单元测试。它会产生两种不同形式的测试结果报告：

1）.纯文本
2）.xml文件格式的

默认情况下，这些文件生成在工程的${basedir}/target/surefire-reports，目录下（basedir指的是pom文件所在的目录）。 它可以运行任何testNG,Junit,pojo写的单元测试 -->

<plugin> 
  <groupId>org.apache.maven.plugins</groupId>  
  <artifactId>maven-surefire-plugin</artifactId>  
  <version>2.22.1</version>  
  <configuration> 
    <redirectTestOutputToFile>true</redirectTestOutputToFile>  
    <encoding>utf-8</encoding>  
    <runOrder>alphabetical</runOrder>  
    <argLine>-Xms512m -Xmx1024m -Xss256k</argLine> 
  </configuration> 
</plugin>

```

#### maven-resources-plugin :  负责处理项目资源文件并拷贝到输出目录 

```xml
<?xml version="1.0" encoding="utf-8"?>


<plugin> 
  <groupId>org.apache.maven.plugins</groupId>  
  <artifactId>maven-resources-plugin</artifactId>  
  <version>3.1.0</version>  
  <configuration> 
    <useDefaultDelimiters>false</useDefaultDelimiters>  
    <delimiters> 
      <delimiter>${*}</delimiter> 
    </delimiters> 
  </configuration> 
</plugin>


<!-- Maven将main resources和test resources分开，一般main resources关联main source code，而test resources关联test source code -->

<!-- 可以指定resources插件读取和写入文件的字符编码，比如ASCII,UTF-8或UTF-16 --> 
```



#### build-helper-maven-plugin : 多源码路径

```xml
<?xml version="1.0" encoding="utf-8"?>

<plugin> 
  <groupId>org.codehaus.mojo</groupId>  
  <artifactId>build-helper-maven-plugin</artifactId>  
  <version>3.0.0</version> 
</plugin>

```

#### maven-project-info-reports-plugin  **项目总报告** 

```xml
<?xml version="1.0" encoding="utf-8"?>

<plugin> 
  <groupId>org.apache.maven.plugins</groupId>  
  <artifactId>maven-project-info-reports-plugin</artifactId>  
  <version>3.0.0</version>  
  <configuration> 
    <dependencyDetailsEnabled>false</dependencyDetailsEnabled>  
    <dependencyLocationsEnabled>false</dependencyLocationsEnabled> 
  </configuration>  
  <reportSets> 
    <reportSet> 
      <reports> 
        <report>index</report>  
        <report>team</report>  
        <report>issue-management</report> 
      </reports> 
    </reportSet> 
  </reportSets> 
</plugin>

```

#### apache-rat-plugin :  生成带有Rat输出的报告。 

```xml
<?xml version="1.0" encoding="utf-8"?>

<plugin> 
  <groupId>org.apache.rat</groupId>  
  <artifactId>apache-rat-plugin</artifactId>  
  <executions> 
    <execution> 
      <id>rat-check</id>  
      <phase>none</phase> 
    </execution> 
  </executions> 
</plugin>

```

#### license-maven-plugin : 检查三方库依赖的许可

```xml
<plugin>
    <groupId>com.mycila</groupId>
    <artifactId>license-maven-plugin</artifactId>
    <inherited>false</inherited>
    <configuration>
        <skip>true</skip>
    </configuration>
</plugin>
```

#### wagon-maven-plugin : 上传自动化部署到远程服务器

```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>wagon-maven-plugin</artifactId>
    <version>1.0</version>
    <configuration>
        <serverId>15.119.81.111</serverId>
        <fromFile>target/extractor-alm.jar</fromFile>
        <url>scp://hpba@15.119.81.111/home/hpba/HPBA-340-340-master-redhat-x64/ContentPacks/ALM/EXTRACTOR/extractor-alm/</url>
    </configuration>
</plugin>

```

#### maven-replacer-plugin : 打包时对文件进行处理

```xml
  
<plugin>
    <groupId>com.google.code.maven-replacer-plugin</groupId>
    <artifactId>replacer</artifactId>
    <version>1.5.3</version>
    <executions>
        <!-- 打包前进行替换 -->
        <execution>
            <phase>prepare-package</phase>
            <goals>
                <goal>replace</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <!-- 自动识别到项目target文件夹 -->
        <basedir>${build.directory}</basedir>
        <!-- 替换的文件所在目录规则 -->
        <includes>
            <include>${build.finalName}/WEB-INF/views/*.html</include>
            <include>${build.finalName}/WEB-INF/views/**/*.html</include>
        </includes>
        <replacements>
            <!-- 更改规则，在css/js文件末尾追加?v=时间戳，反斜杠表示字符转义 -->
            <replacement>
                <token>\.css\"</token>
                <value>.css?v=${maven.build.timestamp}\"</value>
            </replacement>
            <replacement>
                <token>\.css\'</token>
                <value>.css?v=${maven.build.timestamp}\'</value>
            </replacement>
            <replacement>
                <token>\.js\"</token>
                <value>.js?v=${maven.build.timestamp}\"</value>
            </replacement>
            <replacement>
                <token>\.js\'</token>
                <value>.js?v=${maven.build.timestamp}\'</value>
            </replacement>
        </replacements>
    </configuration>
</plugin>

```

#### versions-maven-plugin : 版本号管理

```xml
<!-- 一行命令同时修改maven项目中多个mudule的版本号 -->
</plugins>
    <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>versions-maven-plugin</artifactId>
        <version>2.7</version>
        <configuration>
            <generateBackupPoms>false</generateBackupPoms>
        </configuration>
    </plugin>
</plugins>


<!-- generateBackupPoms用于配置是否生成备份Pom，用于版本回滚 --> 
<!-- 用法 : mvn versions:set -DnewVersion=0.0.2 --> 
```



## 问题



#### Maven 中文 乱码

```
Key: MAVEN_OPTS

Value: -Dfile.encoding=UTF-8

暂时未解决！

```

#### Maven 本地有包而报错

```
可以考虑修改
_remote.properties 
lastupdate
```



#### Failed to execute goal org.codehaus.mojo:buildnumber-maven-plugin

[ERROR] Failed to execute goal org.codehaus.mojo:buildnumber-maven-plugin:1.4:create (default) on project para-idm-core-logic: Execution default of goal org.codehaus.mojo:buildnumber-maven-plugin:1.4:create failed: The scm url cannot be null. -> [Help 1]

```xml
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>buildnumber-maven-plugin</artifactId>
  <inherited>true</inherited>
  <configuration>
    <doCheck>false</doCheck>
    <doUpdate>false</doUpdate>
    <skip>${skipTests}</skip>
  </configuration>
  <executions>
    <execution>
      <phase>validate</phase>
      <goals>
        <goal>create-timestamp</goal>
      </goals>
    </execution>
  </executions>
</plugin>

# 修改  <goal>create-timestamp</goal> 语句 ，通常这里是 create


```



#### - Execution enhancer of goal org.apache.openjpa:openjpa-maven-plugin

```
 Execution enhancer of goal org.apache.openjpa:openjpa-maven-plugin:3.0.0:enhance failed: Field "private java.util.List para.idm.core.persistence.jpa.entity.JPAApp.auxClasses" cannot be annotated by two persistence strategy annotations
 
策略问题 ，例如ManyToMany等


"private java.util.List para.idm.core.persistence.jpa.entity.JPAApp.auxClasses
```



#### - org.apache.maven.plugins.checkstyle.exec.CheckstyleExecutorException

```
	

 org.apache.maven.plugins.checkstyle.exec.CheckstyleExecutorException: There is 1 error reported by Checkstyle 6.18 with  
 Failed during checkstyle execution: There is 1 error reported by Checkstyle 6.18 with D:\sourceCode\gitlab\idm\idm-core\ext\app\persistence-jpa/../../../checks/checkstyle.xml ruleset.
 
 格式出现错误 ，记得看上面的最后一个 info ,可以得到相关的信息
```



#### - Plugin execution not covered by lifecycle configuration: org.apache.openjpa:openjpa-maven-plugin:3.0.0:enhance (execution: enhancer, phase: process-classes)

```
pom 中报错 ：
Plugin execution not covered by lifecycle configuration: org.apache.openjpa:openjpa-maven-plugin:3.0.0:enhance (execution: enhancer, phase: process-classes)

解决方式 ：
<?xml version="1.0" encoding="UTF-8"?>
<lifecycleMappingMetadata>
	<pluginExecutions>
		<pluginExecution>
			<pluginExecutionFilter>
				<groupId>org.codehaus.gmaven</groupId>
				<artifactId>gmaven-plugin</artifactId>
				<versionRange>[1.5,)</versionRange>
				<goals>
					<goal>execute</goal>
				</goals>
			</pluginExecutionFilter>
			<action>
				<ignore />
			</action>
		</pluginExecution>
		<pluginExecution>
			<pluginExecutionFilter>
				<groupId>org.apache.openjpa</groupId>
				<artifactId>enhance</artifactId>
				<versionRange>[1.5,)</versionRange>
				<goals>
					<goal>execute</goal>
				</goals>
			</pluginExecutionFilter>
			<action>
				<ignore />
			</action>
		</pluginExecution>
	</pluginExecutions>
</lifecycleMappingMetadata>


```

#### -编码GBK的不可映射字符

```java
<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>


<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-resources-plugin</artifactId>  
    <version>2.4.3</version>  
    <configuration>  
        <encoding>${project.build.sourceEncoding}</encoding>  
    </configuration>  
</plugin>


// 方法2
properties 加入
<encoding>UTF-8</encoding>  
```

#### 打包时打包所有

```
         <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.1.0</version>
                <configuration>
                    <!-- get all project dependencies -->
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                    <!-- MainClass in mainfest make a executable jar -->
                    <archive>
                        <manifest>
                            <mainClass>util.Microseer</mainClass>
                        </manifest>
                    </archive>

                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <!-- bind to the packaging phase -->
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```



### 案例 : 本地 + aliyun + 私有

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <localRepository>D:\java\resource\repo</localRepository>

    <pluginGroups>
    </pluginGroups>

    <proxies>
    </proxies>

    <servers>
    </servers>

    <profiles>
        <profile>
            <activation>
                <activeByDefault>true</activeByDefault>
                <jdk>1.8</jdk>
            </activation>
            <properties>
                <maven.compiler.source>1.8</maven.compiler.source>
                <maven.compiler.target>1.8</maven.compiler.target>
                <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
            </properties>
            <id>ParaESC-Dev</id>
            <!-- 注意其中得三个 repository -->
            <repositories>
                <repository>
                    <id>ParaESC</id>
                    <url>http://192.168.2.21:8081/repository/all-blend-dev/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                    </snapshots>
                </repository>
                <repository>
                    <id>aliyun</id>
                    <url>hhttp://maven.aliyun.com/nexus/content/groups/public/</url>
                </repository>
				<repository>
                    <id>local</id>
                    <url>file://D:\java\resource\repo</url>
                </repository>
            </repositories>
        </profile>
        <profile>
            <id>aliyun</id>
            <repositories>
                <repository>
                    <id>aliyun</id>
                    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                    </snapshots>
                </repository>
            </repositories>
        </profile>
    </profiles>

    <!-- step 激活节点 -->
    <activeProfiles>
        <activeProfile>ParaESC-Dev</activeProfile>
        <activeProfile>aliyun</activeProfile>
    </activeProfiles>


</settings>

```

