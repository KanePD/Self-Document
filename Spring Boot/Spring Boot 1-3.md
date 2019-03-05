# Spring Boot Guides Examples(1~3)

参考网址：https://spring.io/guides

## 创建一个RESTful Web Service

- 使用Eclipse 创建一个 Spring Boot项目

```
Project -> Other -> Spring Boot -> Spring Starter Project
```

- 直接找到，spring boot自动创建的application类，

```bash
#我的是 RestfulWebService1Application.java
右键 -> Run As -> Java Application 运行
------------------console result---------------------
#看见下面两行，则开启成功
2019-02-24 10:38:42.935  INFO 6724 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-02-24 10:38:42.940  INFO 6724 --- [           main] c.e.kane.RestfulWebService1Application   : Started RestfulWebService1Application in 4.108 seconds (JVM running for 5.631)
```

- 打开浏览器，键入 localhost:8080会得到下面的报错信息，因为我们还没有定义Controller

```bash
Whitelabel Error Page
This application has no explicit mapping for /error, so you are seeing this as a fallback.

Sun Feb 24 10:39:50 CST 2019
There was an unexpected error (type=Not Found, status=404).
No message available
## 可以参考下面的网址，对Whitelabel Error Page 自定义
https://www.baeldung.com/spring-boot-custom-error-page
```

注：可以自定义该页面，**到官网查看**

- 开始写如一些java类

```java
// 在Application同级目录下创建Model 文件夹，增加Greeting 类
// Greeting.java
package com.example.kane.Model;
public class Greeting{
	private final long id;
	private final String content;
	public Greeting (long id , String content){
		
		this.id = id ;
		this.content = content;
	}
	public long getId(){
		return id;
	}
	public String getContent(){
		return content;
	}
}
// 在Application同级目录下创建 Controller文件夹，增加GreetingController
// GreetingController.java
package com.example.kane.Controller;

import java.util.concurrent.atomic.AtomicLong;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
	// RestController是Spring4的新的注解，被注解的controller返回域对象，不是view
	// 它是@Controller和@ResponseBody的缩写
import org.springframework.web.bind.annotation.RestController;
import com.example.kane.Model.*;
@RestController
public class GreetingController {

    private static final String template = "Hello, %s!";
    private final AtomicLong counter = new AtomicLong();

    @RequestMapping("/greeting")
    public Greeting greeting(@RequestParam(value="name", defaultValue="World") String name) {
        return new Greeting(counter.incrementAndGet(),
                            String.format(template, name));
    }
}
```

- 访问 localhost:8080/greeting

```bash
# 页面结果
{"id":1,"content":"Hello, World!"}
```

- 至此一个简单的Restful Web Service 搭好了

注：官方的关于Restful Web Service 网址 https://spring.io/guides/gs/rest-service/

## 创建一个计划的任务 Scheduling Tasks

- 创建schdule task类

```java
package com.example.kane;
import java.text.SimpleDateFormat;
import java.util.Date;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
@Component

public class SchduleTask {
	private static final Logger log = LoggerFactory.getLogger(SchduleTask.class);

    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");

    @Scheduled(fixedRate = 5000)
    public void reportCurrentTime() {
        log.info("The time is now {}", dateFormat.format(new Date()));
    }
}
```

- 更改Application 类，增加@EnableScheduling

```java
package com.example.kane;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;
@SpringBootApplication
@EnableScheduling
public class RestfulWebService1Application {

	public static void main(String[] args) {
		SpringApplication.run(RestfulWebService1Application.class, args);
	}

}

```

- 结果，会在console中看到如下输出

```bash
2019-02-24 12:00:14.770  INFO 21096 --- [   scheduling-1] com.example.kane.SchduleTask             : The time is now 12:00:14
2019-02-24 12:00:19.770  INFO 21096 --- [   scheduling-1] com.example.kane.SchduleTask             : The time is now 12:00:19
2019-02-24 12:00:24.770  INFO 21096 --- [   scheduling-1] com.example.kane.SchduleTask             : The time is now 12:00:24

## 不停地运行Schdule函数
```

- 其他的Schdule选项

https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#scheduling

```java
// 等待上一个执行之后5秒执行下一次
@Scheduled(fixedDelay=5000)
//-------------------------------------
//上一次开启之后5秒执行下一次，无论第一次是否执行完成
@Scheduled(fixedRate = 5000)
//-------------------------------------
// 使用cron 进行定时的安排，语法与linux cronjob相同
@Scheduled(cron="*/5 * * * * MON-FRI")
```

**注：cron表达式规则**

```
Seconds Minutes Hours DayofMonth Month DayofWeek Year 
或
Seconds Minutes Hours DayofMonth Month DayofWeek
```

| 字段                     | 允许值                                 | 允许的特殊字符               |
| ------------------------ | -------------------------------------- | ---------------------------- |
| 秒（Seconds）            | 0~59的整数                             | , - * /    四个字符          |
| 分（*Minutes*）          | 0~59的整数                             | , - * /    四个字符          |
| 小时（*Hours*）          | 0~23的整数                             | , - * /    四个字符          |
| 日期（*DayofMonth*）     | 1~31的整数（但是你需要考虑你月的天数） | ,- * ? / L W C     八个字符  |
| 月份（*Month*）          | 1~12的整数或者 JAN-DEC                 | , - * /    四个字符          |
| 星期（*DayofWeek*）      | 1~7的整数或者 SUN-SAT （1=SUN）        | , - * ? / L C #     八个字符 |
| 年(可选，留空)（*Year*） | 1970~2099                              | , - * /    四个字符          |

 ## 消费一个Restful Web Service

这里的消费意为在java中访问一些Restful API 并获得想要的内容。

- 更改 Application.java文件

```java
package com.example.kane;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.scheduling.annotation.EnableScheduling;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
public class RestfulWebService1Application {
	private static final Logger log = LoggerFactory.getLogger(RestfulWebService1Application.class);
	public static void main(String[] args) {
//		不启动Spring boot的application，直接运行RestTemplate
//		SpringApplication.run(RestfulWebService1Application.class, args);
		RestTemplate restTemplate = new RestTemplate();
		System.out.println(Quote.class);
        Quote quote = restTemplate.getForObject("http://gturnquist-quoters.cfapps.io/api/random", Quote.class);
        log.info(quote.toString());
	}
    /*  下面的代码官方说明是要管理RestTemplate，不明白什么意思
	@Bean
	public RestTemplate restTemplate(RestTemplateBuilder builder) {
		return builder.build();
	}

	@Bean
	public CommandLineRunner run(RestTemplate restTemplate) throws Exception {
		return args -> {
			Quote quote = restTemplate.getForObject(
					"http://gturnquist-quoters.cfapps.io/api/random", Quote.class);
			log.info(quote.toString());
		};
	}*/
}
```

- 定义一个Quote.java类

```java
// Quote 定义的时候需要知道 restful api 返回的json对象都有什么属性，如果Quote中定义的属性在api中不存在那么将被复制为null，
//我将官网的type属性更改为 type1属性后，结构显示type1:null
package com.example.kane;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class Quote {
	private String type1;
	private Value value;
	
	public Quote() {
	}
	
	public String getType() {
	    return type1;
	}
	
	public void setType(String type) {
	    this.type1 = type;
	}
	
	public Value getValue() {
	    return value;
	}
	
	public void setValue(Value value) {
	    this.value = value;
	}
	
	@Override
	public String toString() {
	    return "Quote{" +
	            "type='" + type1 + '\'' +
	            ", value=" + value +
	            '}';
	}
}

```

- 定义Value.java

```java
//value 与 quote相同，定义了这两个类 主要是http://gturnquist-quoters.cfapps.io/api/random返回的json对象中有这两个类
package com.example.kane;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class Value {

    private Long id;
    private String quote;

    public Value() {
    }

    public Long getId() {
        return this.id;
    }

    public String getQuote() {
        return this.quote;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public void setQuote(String quote) {
        this.quote = quote;
    }

    @Override    
    public String toString() {
        return "Value{" +
                "id=" + id +
                ", quote='" + quote + '\'' +
                '}';
    }
}

```

- 上结果

```bash
20:59:39.613 [main] INFO com.example.kane.RestfulWebService1Application - Quote{type='null', value=Value{id=4, quote='Previous to Spring Boot, I remember XML hell, confusing set up, and many hours of frustration.'}}
```

# Spring Boot Guides Examples(4~6)

## Building Java Projects with Gradle

为什么选择Gradle

- Build Anything：各种语言、各种平台
- Automate Everything：