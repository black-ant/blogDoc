# Case 1.3.1 RestEasy 使用方式

> EasyRest 是一个比较早期的 Restful 框架 , 但是仍然有部分公司还在用 , 该文档用于描述该框架的基础点及常见用法

> https://github.com/black-ant/case/tree/master/case%202.3.%20easyrest



[TOC]



## 一 . 框架特性

```
> RESTEasy是JBoss的开源项目之一，是一个RESTful Web Services框架。

> 这个框架在一个产品里面用过 ,当时确实很疑惑 , 为什么要选择该框架 , 给出的结论是轻量级和效率 , 但是个人的想法是 , 现在常见的Rest 框架 , 都是基于 ws 包做的底层 ,轻量级只是意味着封装更少 , 功能更少而已 , 而效率在微服务的场景下 , 反而没那么大影响了.
```





## 二 . 框架用法

### 2.1 Maven

```
        <dependency>
            <groupId>org.jboss.resteasy</groupId>
            <artifactId>resteasy-spring-boot-starter</artifactId>
            <version>4.0.0.Final</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.jboss.resteasy</groupId>
            <artifactId>resteasy-validator-provider</artifactId>
            <version>4.1.0.Final</version>
        </dependency>
        <dependency>
            <groupId>org.jboss.resteasy</groupId>
            <artifactId>resteasy-servlet-initializer</artifactId>
            <version>4.1.0.Final</version>
        </dependency>
```

### 2.2  SpringBoot

```java
// Step 1 : RestEasy 基于一个独立的Exception

@Component
@ApplicationPath("/rest/")
public class EasyRestApplication extends Application {
}


// Step 2 : 准备配置文件
server.port=8033
resteasy.jaxrs.app.registration=property
resteasy.jaxrs.app.classes=com.gamg.easyrest.demo.EasyRestApplication
    
// Step 3 : 基于 javax.ws.rs 的 Controller 类
@Path("/user")
@Component
@Produces({
        MediaType.APPLICATION_JSON, RESTHeaders.APPLICATION_YAML, MediaType.APPLICATION_XML
})
@Consumes({
        MediaType.APPLICATION_JSON, RESTHeaders.APPLICATION_YAML, MediaType.APPLICATION_XML, MediaType.TEXT_PLAIN
})
public class UserController {

    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @GET
    @Path("/show")
    public void showUser() {
        logger.info("user is in");
    }


    @GET
    @Path("/getUser")
    public User getUserMsg() {
        return new User("gang", 1, "show user type");
    }
}


// Step 4 : 访问 
```

## 三 . 心得

```
> RestEasy 是早期使用 restful 想法的框架 , 相对于过去 view 模式的方式 , 更加简单 , 基于  javax.ws.rs 的注解 , 也让上手更加简单 , 不过相对于而言 , 这类框架资料更少 , 功能也少 , 使用成本较高 , 可以针对特定场景使用
```

