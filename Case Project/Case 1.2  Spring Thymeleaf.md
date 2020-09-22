# Case 1.2  Spring Thymeleaf

> 该文档包含 Spring Thymeleaf 的使用方式



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

