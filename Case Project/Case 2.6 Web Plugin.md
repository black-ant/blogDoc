# Web Plugin 

> 该文总结了一些常见Web工具的用法 , 包括 文件上传 , 下载 , okHttp , swagger , HttpClient ,RestTemplate 等



> https://github.com/black-ant/case
>
>  http://www.antblack.xyz/

[TOC]



## 一 . Plugin 简介

```

```

## 二 . Plugin 使用

### 2.1 文件上传的三种方式 

> https://www.cnblogs.com/fjsnail/p/3491033.html 

#### 2.1.1 配置文件

```java
@Configuration
public class UploadConfig {

    @Bean
    public CommonsMultipartResolver multipartResolver() {
        CommonsMultipartResolver resolver = new CommonsMultipartResolver();
        resolver.setDefaultEncoding("utf-8");
        resolver.setMaxUploadSize(10485760);
        resolver.setMaxInMemorySize(40960);
        return resolver;
    }

}

```

#### 2.1.2 使用

```java
package com.fileupload.demo.controller;


import org.apache.commons.io.FileUtils;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.multipart.MultipartHttpServletRequest;
import org.springframework.web.multipart.commons.CommonsMultipartFile;
import org.springframework.web.multipart.commons.CommonsMultipartResolver;

import javax.servlet.http.HttpServletRequest;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.util.Date;
import java.util.Iterator;

/**
 * By https://www.cnblogs.com/fjsnail/p/3491033.html
 */
@RestController
public class UploadController {

    @RequestMapping(value = "/upload", method = RequestMethod.POST)
    public @ResponseBody
    String uploadMethod(MultipartFile file) {
        try {
            FileUtils.writeByteArrayToFile(new File("i:/uploadtest" + file.getOriginalFilename()), file.getBytes());
            return "ok";
        } catch (IOException e) {
            e.printStackTrace();
            return "not";
        }

    }

    /*
     * 通过流的方式上传文件 
     * @RequestParam("file") 将name=file控件得到的文件封装成CommonsMultipartFile 对象
        var form = new FormData();
        form.append("files", fileInput.files[0], "/D:/资料/pdf/SpringBoot.pdf");

        var settings = {
          "url": "127.0.0.1:8088/upload/fileUpload",
          "method": "POST",
          "mimeType": "multipart/form-data",
          "data": form
        };

        $.ajax(settings).done(function (response) {
          console.log(response);
        });
     */
    @PostMapping("fileUpload")
    public String  fileUpload(@RequestParam("files") CommonsMultipartFile file) throws IOException {


        //用来检测程序运行时间
        long  startTime=System.currentTimeMillis();
        System.out.println("fileName："+file.getOriginalFilename());

        try {
            //获取输出流
            OutputStream os=new FileOutputStream("E:/"+new Date().getTime()+file.getOriginalFilename());
            //获取输入流 CommonsMultipartFile 中可以直接得到文件的流
            InputStream is=file.getInputStream();
            int temp;
            //一个一个字节的读取并写入
            while((temp=is.read())!=(-1))
            {
                os.write(temp);
            }
            os.flush();
            os.close();
            is.close();

        } catch (FileNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        long  endTime=System.currentTimeMillis();
        System.out.println("方法一的运行时间："+String.valueOf(endTime-startTime)+"ms");
        return "/success";
    }

    /**
     * 采用file.Transto 来保存上传的文件
     * @param file
     * @return
     * @throws IOException
     */
    @PostMapping("fileUpload2")
    public String  fileUpload2(@RequestParam("file") CommonsMultipartFile file) throws IOException {
        long  startTime=System.currentTimeMillis();
        System.out.println("fileName："+file.getOriginalFilename());
        String path="E:/"+new Date().getTime()+file.getOriginalFilename();

        File newFile=new File(path);
        //通过CommonsMultipartFile的方法直接写文件（注意这个时候）
        file.transferTo(newFile);
        long  endTime=System.currentTimeMillis();
        System.out.println("方法二的运行时间："+String.valueOf(endTime-startTime)+"ms");
        return "/success";
    }

    /*
     *采用spring提供的上传文件的方法
     */
    @PostMapping("springUpload")
    public String  springUpload(HttpServletRequest request) throws IllegalStateException, IOException
    {
        long  startTime=System.currentTimeMillis();
        //将当前上下文初始化给  CommonsMutipartResolver （多部分解析器）
        CommonsMultipartResolver multipartResolver=new CommonsMultipartResolver(
                request.getSession().getServletContext());
        //检查form中是否有enctype="multipart/form-data"
        if(multipartResolver.isMultipart(request))
        {
            //将request变成多部分request
            MultipartHttpServletRequest multiRequest=(MultipartHttpServletRequest)request;
            //获取multiRequest 中所有的文件名
            Iterator iter=multiRequest.getFileNames();

            while(iter.hasNext())
            {
                //一次遍历所有文件
                MultipartFile file=multiRequest.getFile(iter.next().toString());
                if(file!=null)
                {
                    String path="E:/springUpload"+file.getOriginalFilename();
                    //上传
                    file.transferTo(new File(path));
                }

            }

        }
        long  endTime=System.currentTimeMillis();
        System.out.println("方法三的运行时间："+String.valueOf(endTime-startTime)+"ms");
        return "/success";
    }
}

```

### 2.2 文件下载的方式

> ```
> https://www.cnblogs.com/lucas1024/p/9533220.html
> ```

```java
package com.fileupload.demo.controller;

import com.fileupload.demo.to.DownloadTO;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletResponse;
import java.io.BufferedInputStream;
import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLConnection;

/**
 * @Classname DownLoadController
 * @Description By https://www.cnblogs.com/lucas1024/p/9533220.html
 * @Date 2021/1/9 23:05
 * @Created by
 */
@RestController
@RequestMapping("/download")
public class DownLoadController {

    private Logger logger = LoggerFactory.getLogger(this.getClass());

    /**
     * 以流的方式下载.
     * Invalid character found in the request target. The valid characters are defined in RFC 7230 and RFC 3986
     * 注意编码 :: D:/SpringBoot.pdf
     *
     * @param path
     * @param response
     * @return
     */
    @GetMapping("stream")
    public HttpServletResponse download(@RequestParam(name = "path") String path, HttpServletResponse response) {

        logger.info("------> this is in stream <-------");
        try {
            // path是指欲下载的文件的路径。
            File file = new File(path);
            // 取得文件名。
            String filename = file.getName();
            // 取得文件的后缀名。
            String ext = filename.substring(filename.lastIndexOf(".") + 1).toUpperCase();

            // 以流的形式下载文件。
            InputStream fis = new BufferedInputStream(new FileInputStream(path));
            byte[] buffer = new byte[fis.available()];
            fis.read(buffer);
            fis.close();
            // 清空response
            response.reset();
            // 设置response的Header
            response.addHeader("Content-Disposition", "attachment;filename=" + new String(filename.getBytes()));
            response.addHeader("Content-Length", "" + file.length());
            OutputStream toClient = new BufferedOutputStream(response.getOutputStream());
            response.setContentType("application/octet-stream");
            toClient.write(buffer);
            toClient.flush();
            toClient.close();
        } catch (IOException ex) {
            ex.printStackTrace();
        }
        return response;
    }

    /**
     * 保存到本地
     *
     * @param path
     * @param response
     * @throws FileNotFoundException
     */
    @GetMapping("download2")
    public void downloadLocal(@RequestParam(name = "path") String path, HttpServletResponse response) throws FileNotFoundException {

        // 下载本地文件
        String fileName = "Operator.doc".toString(); // 文件的默认保存名
        // 读到流中
        InputStream inStream = new FileInputStream(path);// 文件的存放路径
        // 设置输出的格式
        response.reset();
        response.setContentType("bin");
        response.addHeader("Content-Disposition", "attachment; filename=\"" + fileName + "\"");
        // 循环取出流中的数据
        byte[] b = new byte[100];
        int len;
        try {
            while ((len = inStream.read(b)) > 0)
                response.getOutputStream().write(b, 0, len);
            inStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 下载网络图片
     *
     * @param pathUrl
     * @param outPutPath
     * @param response
     * @throws MalformedURLException
     */
    @GetMapping("download3")
    public void downloadNet(@RequestParam(name = "pathUrl") String pathUrl,
                            @RequestParam(name = "outPutPath") String outPutPath,
                            HttpServletResponse response) throws MalformedURLException {
        // 下载网络文件
        int bytesum = 0;
        int byteread = 0;

        URL url = new URL(pathUrl);

        try {
            URLConnection conn = url.openConnection();
            InputStream inStream = conn.getInputStream();
            FileOutputStream fs = new FileOutputStream(outPutPath);

            byte[] buffer = new byte[1204];
            int length;
            while ((byteread = inStream.read(buffer)) != -1) {
                bytesum += byteread;
                System.out.println(bytesum);
                fs.write(buffer, 0, byteread);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 127.0.0.1:8088/download/download4?path=D:/avataaars.png&isOnLine=true
     * @param filePath
     * @param response
     * @param isOnLine
     * @throws Exception
     */
    @GetMapping("download4")
    public void downLoad(@RequestParam(name = "path") String filePath, HttpServletResponse response,
                         @RequestParam(name = "isOnLine") boolean isOnLine) throws Exception {
        File f = new File(filePath);
        if (!f.exists()) {
            response.sendError(404, "File not found!");
            return;
        }
        BufferedInputStream br = new BufferedInputStream(new FileInputStream(f));
        byte[] buf = new byte[1024];
        int len = 0;

        response.reset(); // 非常重要
        if (isOnLine) { // 在线打开方式
            URL u = new URL("file:///" + filePath);
            response.setContentType(u.openConnection().getContentType());
            response.setHeader("Content-Disposition", "inline; filename=" + f.getName());
            // 文件名应该编码成UTF-8
        } else { // 纯下载方式
            response.setContentType("application/x-msdownload");
            response.setHeader("Content-Disposition", "attachment; filename=" + f.getName());
        }
        OutputStream out = response.getOutputStream();
        while ((len = br.read(buf)) > 0)
            out.write(buf, 0, len);
        br.close();
        out.close();
    }

}

```

### 2.3 Swagger 

```
> 官网 : https://swagger.io/
// Swagger V3 参考地址 : https://github.com/black-ant/case/tree/master/case%202.6.4%20swaggerv3
```



#### 2.3.1 Maven 配置

```
// Maven
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>
```

#### 2.3.2 Config 配置

```java
@Configuration
@EnableSwagger2
public class Swagger2 {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.gang.swagger"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Spring Boot中使用Swagger2构建RESTful APIs")
                .description("swagger api 描述")
                .termsOfServiceUrl("www.baidu.com")
                .contact("gang")
                .version("1.0")
                .build();
    }

}


// http://localhost:8080/swagger-ui.html
```



### 2.4 OKHttp 

```
okHttp 是一个类似于 HttpClient 的 请求辅助框架 ,但是 OKHttp 提供了很强大的功能 ,让你轻松的处理Http 请求 , okhttp 采用链式处理 ,同时提供event 功能以及 Interceptors 功能 , 同时提供了很好用异步处理操作 , 是一个百宝箱一样的框架

// 具体可以参考 OkHTTP 官方文档
https://square.github.io/okhttp/recipes/

```

#### 2.4.1 基本使用

```java
// 构建对象
OkHttpClient client = new OkHttpClient();

// 构建 Event 监听对象
// private static final class PrintingEventListener extends EventListener
OkHttpClient client = new OkHttpClient.Builder()
            .eventListener(new PrintingEventListener())
            .build();

// 构建 Interceptor 对象
// class LoggingInterceptor implements Interceptor 
OkHttpClient client = new OkHttpClient.Builder()
            .addInterceptor(new LoggingInterceptor())
            .build();

// Get 请求
Request request = new Request.Builder().url(url).build();
try (Response response = client.newCall(request).execute()) {
	 logger.info("------> this is response :{} <-------", response.body().string());
}

// Post 请求
JSONObject json = new JSONObject();
json.put("username", "ant");
json.put("age", 18);

RequestBody body = RequestBody.create(json.toJSONString(), JSON);
Request request = new Request.Builder().url(url).post(body).build();
try (Response response = client.newCall(request).execute()) {
	logger.info("------> this is Post response :{} <-------", response.body().string());
}

// Https 证书访问
private final OkHttpClient client = new OkHttpClient.Builder()
            .certificatePinner(
                    new CertificatePinner.Builder()
                            .add("publicobject.com", "sha256/afwiKY3RxoMmLkuRW1l7QsPZTJPwDS2pdDROQjXw8ig=")
                            .build())
            .build();
```

