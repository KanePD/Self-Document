# 使用JDBC 获取相关的数据

## 什么是JDBC

Java Database Connectivity 是一种用于执行SQL语句的Java API，与数据库建立连接、发送 操作数据库的语句并处理结果。

## Spring Boot 使用 JDBC

### 增加依赖

- 修改pom.xml：将dependecies 修改为如下两个

```
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
        </dependency>
    </dependencies>
```

### 创建 Customer.java 类

```java
package com.example.kane.Model;

public class Customer {
	private long id;
    private String firstName, lastName;

    public Customer(long id, String firstName, String lastName) {
        this.id = id;
        this.firstName = firstName;
        this.lastName = lastName;
    }

    @Override
    public String toString() {
        return String.format(
                "Customer[id=%d, firstName='%s', lastName='%s']",
                id, firstName, lastName);
    }

    // getters & setters omitted for brevity
}
```

### 修改Application 类

```java
package com.example.kane;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.scheduling.annotation.EnableScheduling;

import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.client.RestTemplate;

import com.example.kane.Model.Customer;

@SpringBootApplication
//@EnableScheduling
public class RestfulWebService1Application implements CommandLineRunner{
	
	private static final Logger log = LoggerFactory.getLogger(RestfulWebService1Application.class);

    public static void main(String args[]) {
        SpringApplication.run(RestfulWebService1Application.class, args);
    }

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Override
    public void run(String... strings) throws Exception {

        log.info("Creating tables");

        jdbcTemplate.execute("DROP TABLE customers IF EXISTS");
        jdbcTemplate.execute("CREATE TABLE customers(" +
                "id SERIAL, first_name VARCHAR(255), last_name VARCHAR(255))");

        // Split up the array of whole names into an array of first/last names
        List<Object[]> splitUpNames = Arrays.asList("John Woo", "Jeff Dean", "Josh Bloch", "Josh Long").stream()
                .map(name -> name.split(" "))
                .collect(Collectors.toList());

        // Use a Java 8 stream to print out each tuple of the list
        splitUpNames.forEach(name -> log.info(String.format("Inserting customer record for %s %s", name[0], name[1])));

        // Uses JdbcTemplate's batchUpdate operation to bulk load data
        jdbcTemplate.batchUpdate("INSERT INTO customers(first_name, last_name) VALUES (?,?)", splitUpNames);

        log.info("Querying for customer records where first_name = 'Josh':");
        jdbcTemplate.query(
                "SELECT id, first_name, last_name FROM customers WHERE first_name = ?", new Object[] { "Josh" },
                (rs, rowNum) -> new Customer(rs.getLong("id"), rs.getString("first_name"), rs.getString("last_name"))
        ).forEach(customer -> log.info(customer.toString()));
    }
}
```

### 运行项目看结果

```bash&#39;
2019-03-01 14:19:52.078  INFO 7436 --- [  restartedMain] c.e.kane.RestfulWebService1Application   : Creating tables
2019-03-01 14:19:52.086  INFO 7436 --- [  restartedMain] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2019-03-01 14:19:52.392  INFO 7436 --- [  restartedMain] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2019-03-01 14:19:52.429  INFO 7436 --- [  restartedMain] c.e.kane.RestfulWebService1Application   : Inserting customer record for John Woo
2019-03-01 14:19:52.430  INFO 7436 --- [  restartedMain] c.e.kane.RestfulWebService1Application   : Inserting customer record for Jeff Dean
2019-03-01 14:19:52.430  INFO 7436 --- [  restartedMain] c.e.kane.RestfulWebService1Application   : Inserting customer record for Josh Bloch
2019-03-01 14:19:52.430  INFO 7436 --- [  restartedMain] c.e.kane.RestfulWebService1Application   : Inserting customer record for Josh Long
2019-03-01 14:19:52.461  INFO 7436 --- [  restartedMain] c.e.kane.RestfulWebService1Application   : Querying for customer records where first_name = 'Josh':
2019-03-01 14:19:52.480  INFO 7436 --- [  restartedMain] c.e.kane.RestfulWebService1Application   : Customer[id=3, firstName='Josh', lastName='Bloch']
2019-03-01 14:19:52.480  INFO 7436 --- [  restartedMain] c.e.kane.RestfulWebService1Application   : Customer[id=4, firstName='Josh', lastName='Long']
2019-03-01 14:20:01.122  INFO 7436 --- [nio-8080-exec-5] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2019-03-01 14:20:01.123  INFO 7436 --- [nio-8080-exec-5] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2019-03-01 14:20:01.146  INFO 7436 --- [nio-8080-exec-5] o.s.web.servlet.DispatcherServlet        : Completed initialization in 22 ms
```

### 说明

官网的例子，没有配置JDBC Template的Datasource，默认使用的是H2 的内存存储的数据库，只能当做测试使用。下面会有介绍更改DataSource的方法

## 介绍下 CommandLineRunner

### 功能

在项目启动后，执行执行功能，我们可以定一个类，去实现CommandLineRunner接口，重写run方法，执行一部分操作。**需要注意的是，定义类必须标记为Spring管理的组件**

### 测试类

```java
package com.example.kane.Model;

import org.springframework.boot.CommandLineRunner;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;
@Component
@Order(value=1) //因为可能有许多事情要做，Order 可以根据大小，判读执行的顺序
public class run_after_application implements CommandLineRunner{

	@Override
	public void run(String... args) throws Exception {
		// TODO Auto-generated method stub
		System.out.println("-----------------------");
	}
	
}
```

## 介绍下JdbcTempalte

在JDBC核心包中，JdbcTemplate是主要的类，简化了JDBC的使用，避免了一些常规错误。它能够执行JDBC核心流程，在应用代码之上提供SQL语句、导出结果。这个类执行SQL查询、更新、对结果集重复操作捕获JDBC的异常。并将它翻译成`org.springframework.dao` 包中定义的基本的、信息量更大的异常层次结构。

### JDBC构造方法

- JdbcTemplate()

```java
//为Bean创建一个JdbcTemplate以供使用
//再没配置DataSource的情况下 springboot提供了 一些嵌入式的数据库支持，上面的例子使用的就是H2数据库，是一个内存的数据库
```

- JdbcTemplate(javax.sql.DataSource dataSource)

```java
//构造的时候传入一个 DataSource，来获取链接
//JdbcTemplate Spring boot默认链接的是H2 database，
```

### 在spring boot中配置mysql 数据库

- 数据库配置类 **db_config**

```java
package com.example.kane.config;

import org.apache.commons.dbcp.BasicDataSource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import javax.sql.DataSource;
@Configuration
public class db_config {
	//这个类是一个Config类
		@Value("${db.driver}")
		private String DRIVER;

		@Value("${db.password}")
		private String PASSWORD;

		@Value("${db.url}")
		private String URL;

		@Value("${db.username}")
		private String USERNAME;
		@Bean
		public DataSource dataSource1() {
			BasicDataSource dataSource = new BasicDataSource();
			dataSource.setDriverClassName(DRIVER);
			dataSource.setUrl(URL);
			dataSource.setUsername(USERNAME);
			dataSource.setPassword(PASSWORD);
			return dataSource;
		}
}
```

- application.properties

```properties
# Database
# mysqljdbc连接驱动
db.driver:com.mysql.cj.jdbc.Driver
db.url:jdbc:mysql://localhost:3306/test
db.username:root
db.password:root
```

- pom.xml

```xml
<dependency>
    <groupId>commons-dbcp</groupId>
    <artifactId>commons-dbcp</artifactId>
    <version>1.4</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<!-- 需要用到commons-dbcp连接池，以及连接mysql使用的drver-->
```

- application 启动类修改

```java
    @Autowired
    JdbcTemplate jdbcTemplate;
	//下面是加载了数据库的配置。只需要增加这个
	@Autowired
	db_config db_config;
```

- 运行程序后会发现数据存储到本地数据库

```mysql
SELECT * from customers;
------------------------
1	John	Woo
2	Jeff	Dean
3	Josh	Bloch
4	Josh	Long
```

### 另一个简单的方法配置mysql数据库

- 直接修改application.properties

```properties
# database
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

- 将properties改成yml文件 application.yml

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
```

注：这两种方式又回归到配置文件的方式了，

### JDBC Template常用方法

- **execute方法：**可以用于执行任何SQL语句，一般用于执行DDL语句；
- **update方法及batchUpdate方法：**update方法用于执行新增、修改、删除等语句；batchUpdate方法用于执行批处理相关语句；
- **query方法及queryForXXX方法：**用于执行查询相关语句；
- **call方法：**用于执行存储过程、函数相关语句。
- 参考官网 https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html

### 关于连接池的一些内容

- 为什么要使用数据库连接池？

  因为建立数据库连接是一个非常耗时的过程，使用连接池可以预先同数据库建立连接，放在内存中。应用需要使用数据库的时候直接使用连接池中的连接即可。

- 当前三大主流连接池

  - DBCP：提供最大空闲连接数，超过连接全部自动断开连接，其他两个没有。
  - C3P0：提供最大空闲连接时间，这样可以做到自动收回空闲连接的机制
  - Druid：阿里出品的，同样提供最大的空闲连接时间



