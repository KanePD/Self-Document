[TOC]

# Spring Boot 上传文件

文件上传是一个基本需求，话不多说，我们直接演练

## 功能实现

### 增加Controller`FileUploadController`

#### 代码

```java
package com.example.kane.Controller;
import org.springframework.stereotype.Controller;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.web.bind.annotation.*;

import java.io.IOException;
import java.util.stream.Collectors;

import org.springframework.core.io.Resource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.servlet.mvc.method.annotation.MvcUriComponentsBuilder;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import com.example.kane.service.StorageException;
import com.example.kane.service.StorageFileNotFoundException;
import com.example.kane.service.StorageService;

import com.example.kane.Controller.FileUploadController;
@Controller
public class FileUploadController {
    @Autowired()
    //@Qualifier("FileSystemStorageService") 
    private  StorageService storageService;

    @Autowired
    public FileUploadController(StorageService storageService) {
        this.storageService = storageService;
        System.out.println(this.storageService);
    }
    @GetMapping("/")
    public String listUploadedFiles(Model model) throws IOException {
    	System.out.println(this.storageService);
        model.addAttribute("files", storageService.loadAll().map(
                path -> MvcUriComponentsBuilder.fromMethodName(FileUploadController.class,
                        "serveFile", path.getFileName().toString()).build().toString())
                .collect(Collectors.toList()));

        return "uploadForm";
    }
    @GetMapping("/files/{filename:.+}")
    @ResponseBody
    public ResponseEntity<Resource> serveFile(@PathVariable String filename) {

        Resource file = storageService.loadAsResource(filename);
        return ResponseEntity.ok().header(HttpHeaders.CONTENT_DISPOSITION,
                "attachment; filename=\"" + file.getFilename() + "\"").body(file);
    }

    @PostMapping("/")
    public String handleFileUpload(@RequestParam("file") MultipartFile file,
            RedirectAttributes redirectAttributes) {

        storageService.store(file);
        redirectAttributes.addFlashAttribute("message",
                "You successfully uploaded " + file.getOriginalFilename() + "!");

        return "redirect:/";
    }

    @ExceptionHandler(StorageFileNotFoundException.class)
    public ResponseEntity<?> handleStorageFileNotFound(StorageFileNotFoundException exc) {
    	System.out.println(11);
        return ResponseEntity.notFound().build();
    }
}
```

#### 逻辑分析

- 控制类汇总有一个私有的`StorageService`的类，做逻辑处理。
- 发送**GET** 请求，URL 匹配到`/`时，进入的文件上传的页面，页面汇总包含已上传文件列表、上传按钮。此处使用了**Thymeleaf**模板引擎，后面会介绍
- 发送**GET**请求，URL匹配到`/files/{filename}`时进行下载文件功能。
- 发送**POST**请求，URL匹配到`/`时，进行上传文件的请求
- 当遇到 `StorageFileNotFoundException`的时候异常处理

### 增加Service`StorageService` 

#### 代码

```java
package com.example.kane.service;

import org.springframework.core.io.Resource;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.context.annotation.Bean;
import java.nio.file.Path;
import java.util.stream.Stream;
@Service
public interface StorageService {

    void init();

    void store(MultipartFile file);
    @Bean
    Stream<Path> loadAll();

    Path load(String filename);

    Resource loadAsResource(String filename);

    void deleteAll();

}
```

#### 逻辑分析

上面的Service，只是一个接口，本例中，官方是**面向接口编程**，实现了JAVA的多态。后面会有介绍。

### 增加一个Thymeleaf页面

注：Thymeleaf后面整体介绍，此处简单的HTML 页面，不多做说明。

```html
<html xmlns:th="http://www.thymeleaf.org">
<body>

	<div th:if="${message}">
		<h2 th:text="${message}"/>
	</div>

	<div>
		<form method="POST" enctype="multipart/form-data" action="/">
			<table>
				<tr><td>File to upload:</td><td><input type="file" name="file" /></td></tr>
				<tr><td></td><td><input type="submit" value="Upload" /></td></tr>
			</table>
		</form>
	</div>

	<div>
		<ul>
			<li th:each="file : ${files}">
				<a th:href="${file}" th:text="${file}" />
			</li>
		</ul>
	</div>

</body>
</html>
```

### 修改一些简单的配置`application.properties`

```properties
spring.servlet.multipart.max-file-size=128KB  # file size 
spring.servlet.multipart.max-request-size=128KB # request size
```

### 修改Spring Boot Application类

#### 代码

```java
package com.example.kane;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.scheduling.annotation.EnableScheduling;

import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

import org.slf4j.Logger;

import com.example.kane.config.db_config;
import com.example.kane.service.StorageService;

import com.example.kane.service.StorageProperties;

import org.slf4j.LoggerFactory;
import org.springframework.web.client.RestTemplate;

import com.example.kane.Model.Customer;

@SpringBootApplication
@EnableConfigurationProperties(StorageProperties.class)
//@EnableScheduling
public class RestfulWebService1Application{
	
	private static final Logger log = LoggerFactory.getLogger(RestfulWebService1Application.class);

    public static void main(String args[]) {
        SpringApplication.run(RestfulWebService1Application.class, args);
    }
    @Bean
    CommandLineRunner init(StorageService storageService) {
        return (args) -> {
            //storageService.deleteAll(); 
            storageService.init();
        };
    }
}
```

#### 逻辑分析

- 开启Spring Boot项目
- 定义了一个项目启动后需要运行删除所有文件的逻辑。`CommandLineRunner`之前有做介绍。

### 官网没有说明其他的Service类的定义

按照官网至此已经创建完成了上传文件的应用，但是少了一部分内容，就是其他的Service的定义情况。下面做补充。

#### 接口`StorageService`的实现类`FileSystemStorageService`

```java
package com.example.kane.service;

import java.util.stream.Stream;

import java.io.IOException;
import java.io.InputStream;
import java.net.MalformedURLException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;
import java.util.stream.Stream;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.Resource;
import org.springframework.core.io.UrlResource;
import org.springframework.stereotype.Service;
import org.springframework.util.FileSystemUtils;
import org.springframework.util.StringUtils;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Primary;
@Service(value="FileSystemStorageService")
@Primary()
public class FileSystemStorageService  implements StorageService {
    private final Path rootLocation;

    @Autowired
    public FileSystemStorageService(StorageProperties properties) {
    	System.out.println(properties.test);
        this.rootLocation = Paths.get(properties.getLocation());
    }

    @Override
    public void store(MultipartFile file) {
        String filename = StringUtils.cleanPath(file.getOriginalFilename());
        try {
            if (file.isEmpty()) {
                throw new StorageException("Failed to store empty file " + filename);
            }
            if (filename.contains("..")) {
                // This is a security check
                throw new StorageException(
                        "Cannot store file with relative path outside current directory "
                                + filename);
            }
            try (InputStream inputStream = file.getInputStream()) {
                Files.copy(inputStream, this.rootLocation.resolve(filename),
                    StandardCopyOption.REPLACE_EXISTING);
            }
        }
        catch (IOException e) {
            throw new StorageException("Failed to store file " + filename, e);
        }
    }

    @Override
    public Stream<Path> loadAll() {
        try {
            return Files.walk(this.rootLocation, 1)
                .filter(path -> !path.equals(this.rootLocation))
                .map(this.rootLocation::relativize);
        }
        catch (IOException e) {
            throw new StorageException("Failed to read stored files", e);
        }

    }

    @Override
    public Path load(String filename) {
        return rootLocation.resolve(filename);
    }

    @Override
    public Resource loadAsResource(String filename) {
        try {
            Path file = load(filename);
            Resource resource = new UrlResource(file.toUri());
            if (resource.exists() || resource.isReadable()) {
                return resource;
            }
            else {
                throw new StorageFileNotFoundException(
                        "Could not read file: " + filename);

            }
        }
        catch (MalformedURLException e) {
            throw new StorageFileNotFoundException("Could not read file: " + filename, e);
        }
    }

    @Override
    public void deleteAll() {
        FileSystemUtils.deleteRecursively(rootLocation.toFile());
    }

    @Override
    public void init() {
        try {
            Files.createDirectories(rootLocation);
        }
        catch (IOException e) {
            throw new StorageException("Could not initialize storage", e);
        }
    }
}
```

#### 属性类`StorageProperties`

```java
package com.example.kane.service;
import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("storage")
public class StorageProperties {
    /**
     * Folder location for storing files
     */
    private String location = "upload-dir";
	public String test;

    public String getLocation() {
        return location;
    }

    public void setLocation(String location) {
        this.location = location;
    }
    public void settest(String test) {
    	this.test=test;
    }
    public String gettest() {
    	return test;
    }

}
```

#### 异常类`StorageFileNotFoundException` `StorageException`定义

```java
//StorageFileNotFoundException
package com.example.kane.service;

public class StorageFileNotFoundException  extends StorageException{
    public StorageFileNotFoundException(String message) {
        super(message);
    }

    public StorageFileNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}
//StorageException
package com.example.kane.service;

public class StorageException extends RuntimeException{
    public StorageException(String message) {
        super(message);
    	System.out.println(111111111);
    }

    public StorageException(String message, Throwable cause) {
        super(message, cause);
    	System.out.println(111111111);

    }
}
```

### 至此，运行项目。可以再上传与下载文件

注：在`StorageProperties`中定义了文件在服务中的位置`upload-dir`

## 介绍`@ConfigurationProperties`的用法

### 上面例子的 `@ConfigurationProperties`

`@ConfigurationProperties`可以使用application.properties中的属性。在官网的例子中，application.properties可以任意定义`storage.test=123`然后在类`StorageProperties`中书写get、set方法之后，就可以使用了。官网的例子并没有使用，我们可以将location的默认赋值改成如下做法

- `application.properties`

```properties
storage.location= upload-dir #只需要加这一行就可以
```

- `Storage.Properties`

```java
@ConfigurationProperties("storage")
public class StorageProperties {
    private String location;
    public String getLocation() {
        return location;
    }

    public void setLocation(String location) {
        this.location = location;
    }
}
```

按照如上操作，定义的@ConfigurationProperties("storage")才是有意义的，否则根本没有使用到。

### 扩展：自定义一个Properties供自己使用

默认情况下Spring Boot 会使用 Application.properties，我们再同级目录下创建文件`storage.properties`

- `storage.properties`

```properties
storage.location=upload-dir #注意这里要不加任何引号
```

- `Storage.Properties`

```java
package com.example.kane.service;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;
@Primary //标注当前类为主要的bean类
@Configuration//让当前类能够被Spring识别
@ConfigurationProperties(prefix="storage")
@PropertySource("classpath:storage.properties") //配置路径
public class StorageProperties {
    /**
     * Folder location for storing files
     */
    //private String location = "upload-dir";
	private String location;

    public String getLocation() {
        return location;
    }

    public void setLocation(String location) {
        this.location = location;
    }
}
```

### 没解决的一个问题

1. @Primary这个注解是必须的。

我在这里遇到这个问题，报错信息如下，可以看到是spring没法识别使用哪个bean了，这块没弄明白，应为第一个properties是打包后的target目录下的，第二则是实际代码中的，但是确有冲突，我Primary加上之后，会让spring拿当前类为首要的当做configuration的类。如果是别的原因造成的望指正

```bash
Parameter 0 of constructor in com.example.kane.service.FileSystemStorageService required a single bean, but 2 were found:
	- storageProperties: defined in file [C:\Workflow\Eclipse WS\Restful-Web-Service-1\target\classes\com\example\kane\service\StorageProperties.class]
	- storage-com.example.kane.service.StorageProperties: defined in null
```

## 介绍面向本例中的面向接口编程实现的Java的多态

在官网的例子中，在`FileUploadController`中定义了私有变量是，接口`StorageService`，而由于当前项目中只有一个类`FileSystemStorageService`实现了这个接口，所以项目能够正常运行。而加入我们项目中存在第二个类去实现`StorageService`会怎么样呢。

### 创建第二个实现`StorageService`的类之后的错误

- 创建Service类`twostorageservice`，不需要做什么具体实现。

```java
package com.example.kane.service;

import java.nio.file.Path;
import java.util.stream.Stream;
import org.springframework.stereotype.Service;

import org.springframework.core.io.Resource;
import org.springframework.web.multipart.MultipartFile;
@Service(value="twostorageservice")
public class twostorageservice  implements StorageService {

	@Override
	public void init() {
		// TODO Auto-generated method stub
		
	}

	@Override
	public void store(MultipartFile file) {
		// TODO Auto-generated method stub
		
	}

	@Override
	public Stream<Path> loadAll() {
		// TODO Auto-generated method stub
		return null;
	}

	@Override
	public Path load(String filename) {
		// TODO Auto-generated method stub
		return null;
	}

	@Override
	public Resource loadAsResource(String filename) {
		// TODO Auto-generated method stub
		return null;
	}

	@Override
	public void deleteAll() {
		// TODO Auto-generated method stub
		
	}

}
```

- 结果报错如下

```bash
Parameter 0 of constructor in com.example.kane.Controller.FileUploadController required a single bean, but 2 were found:
	- FileSystemStorageService: defined in file [C:\Workflow\Eclipse WS\Restful-Web-Service-1\target\classes\com\example\kane\service\FileSystemStorageService.class]
	- twostorageservice: defined in file [C:\Workflow\Eclipse WS\Restful-Web-Service-1\target\classes\com\example\kane\service\twostorageservice.class]
	Action:

Consider marking one of the beans as @Primary, updating the consumer to accept multiple beans, or using @Qualifier to identify the bean that should be consumed
```

信息很明显，出现了两个bean Spring不知道选择哪一个了。

### 解决办法

根据上面的提示，我们去解决它。

1. 在冲突的两个Service中的某一个上，增加注解 `@Primary`

```java
//在主要的实现类 FileSystemStorageService 前面加@Primary
@Primary()
public class FileSystemStorageService  implements StorageService {
}
```

2. 对每个Service定义一个name，在使用时进行选择

```java
//FileSystemStorageService
@Service(value="FileSystemStorageService")
@Primary
public class FileSystemStorageService  implements StorageService {}
//twostorageservice
@Service(value="twostorageservice")
public class twostorageservice  implements StorageService {}
// --------------------------使用的位置
@Autowired()
@Qualifier("twostorageservice") 
private  StorageService storageService;
// ------------------------controller中 我们打印一下Service，不管功能实现
 @GetMapping("/")
    public String listUploadedFiles(Model model) throws IOException {
    	System.out.println(this.storageService);
    }
// 输出结果
com.example.kane.service.twostorageservice@1364c
```

<font color='red'>注：实现时发现一个问题，@Primary 与@Qualifier不是两个并行的解决办法。方法2中也需要指定一个Service为主要实现类，否则还是会报错。</font>

### 至此，可以在代码运行时，动态的指定实现类。

## 关于自定义异常处理

### @Exceptionhandler

@Exceptionhandler在Controller中定义，对不同的Exception定义不同的处理方法。官网的例子中对`StorageFileNotFoundException`定义了处理方法。

- 我们删除一个文件，然后点击其连接下载

查看控制台输出

```bash
2019-03-18 15:56:43.071  WARN 16004 --- [nio-8080-exec-2] .m.m.a.ExceptionHandlerExceptionResolver : Resolved [com.example.kane.service.StorageFileNotFoundException: Could not read file: Data Analize.xls]
```

查看页面输出

```html
找不到 localhost 的网页 找不到与以下网址对应的网页：http://localhost:8080/files/Data%20Analize.xls
HTTP ERROR 404
```

- 我们将Controller中@Exceptionhandler方法注释掉，在看控制台输出

是一大串很长的Exception

```bash
com.example.kane.service.StorageFileNotFoundException: Could not read file: Data Analize.xls
	at com.example.kane.service.FileSystemStorageService.loadAsResource(FileSystemStorageService.java:83) ~[classes/:na]
	at com.example.kane.Controller.FileUploadController.serveFile(FileUploadController.java:55) ~[classes/:na]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_172]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_172]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_172]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_172]
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:189) ~[spring-web-5.1.5.RELEASE.jar:5.1.5.RELEASE]
```

- 我们再Controller中增加对异常类`StorageException`的处理方法

```java
    @ExceptionHandler(StorageException.class)
    public ResponseEntity<?> storageException(StorageException exc) {
    	return new ResponseEntity<Object>("test",HttpStatus.GATEWAY_TIMEOUT);
    }
//对异常页面做到自定义
```

- 上传一个空文件，出发`StorageException`异常

查看控制台输出

```bash
2019-03-18 16:04:15.591  WARN 16004 --- [nio-8080-exec-1] .m.m.a.ExceptionHandlerExceptionResolver : Resolved [com.example.kane.service.StorageException: Failed to store empty file New Text Document.txt]
```

查看页面输出

```html
test
```

### ErrorController

我们还可以定一个一个类实现`ErrorController`来对Controller的异常进行处理。

## 关于模板引擎 `Thymeleaf`的用法

### POM.xml增加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

### 修改默认模板路径

默认的路径是`resources/templates`，我们可以再`application.properties`文件中配置如下属性`spring.thymeleaf.prefix= classpath:/templates/test/`进行修改

### 使用

```java
    @GetMapping("/")
    public String listUploadedFiles(Model model) throws IOException {
        //往模板文件中增加变量
        model.addAttribute("files", storageService.loadAll().map(
                path -> MvcUriComponentsBuilder.fromMethodName(FileUploadController.class,
                        "serveFile", path.getFileName().toString()).build().toString())
                .collect(Collectors.toList()));

        return "uploadForm"; //模板文件的名字不带html
    }
```

## 以上是总结Spring Boot上传文件例子的内容



