---
title: SpringBoot.1.基础操作
ate: 2019-03-04 11:32:29
tags:
    - SpringBoot
categories:
    - Java
    - 框架
---
## Spring Boot 入门案例
1. 新建普通maven工程
2. pom.xml中加入springboot `spring-boot-starter-web`依赖
   ``` xml
   <parent>
      <!-- 父依赖 提供dependency management依赖管理，引入这个依赖后声明其他依赖就不需要定义版本号了 -->
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.9.RELEASE</version>
	</parent>
	<dependencies>
      <!-- springboot核心组件 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
	</dependencies>
   ```

3. 编写测试控制器
   ``` java
   package com.chen.springboot.web.contorller;

   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.ResponseBody;
   import org.springframework.web.bind.annotation.RestController;

   @RestController //声明Rest风格的控制器
   @EnableAutoConfiguration //自动配置,相当于写了Spring的配置文件
   public class HelloContorller {

      @RequestMapping("hello")
      @ResponseBody
      public String hello(){
         return "Hello,Spring Boot";
      }

      public static void main(String[] args) {
         //启动springboot项目
         SpringApplication.run(HelloContorller.class,args);
      }
   }
   ```

4. 运行项目测试 访问http://localhost:8080/hello

### 解决maven项目报错 
   解决决思路： 
   1.查看problems选项卡，里面有对问题的描述 
   2.工程右键->maven->update project(勾选Force Update) 



## 启动SpringBoot项目的方式
1. mian方法 缺点只能启动一个控制器
   ``` java 
       public static void main(String[] args) {
         //启动springboot项目
         SpringApplication.run(HelloContorller.class,args);
      }
   ```
2. 编写个用于启动的类 使用`@ComponentScan` 注解设置要扫描的控制器包名
   ``` java
   package com.chen.springboot;

   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
   import org.springframework.context.annotation.ComponentScan;

   @ComponentScan(basePackages = "com.chen.springboot.web")
   @EnableAutoConfiguration
   public class App {
      public static void main(String[] args) {
         //启动SpringBoot项目
         SpringApplication.run(App.class,args);
      }
   }
   ```

## SpringBoot 集成依赖
依赖|描述
---|---
**spring-boot-starter**|核心 POM，包含自动配置支持、日志库和对 YAML 配置文件的支持。
spring-boot-starter-amqp|通过 spring-rabbit 支持 AMQP
spring-boot-starter-aop|包含 spring-aop 和 AspectJ 来支持面向切面编程（AOP）。
spring-boot-starter-batch|支持 Spring Batch，包含 HSQLDB。
spring-boot-starter-data-jpa|包含 spring-data-jpa、spring-orm 和 Hibernate 来支持 JPA。
spring-boot-starter-data-mongodb|包含 spring-data-mongodb 来支持 MongoDB。
spring-boot-starter-data-rest|通过 spring-data-rest-webmvc 支持以 REST 方式暴露 Spring Data 仓库。
spring-boot-starter-jdbc|支持使用 JDBC 访问数据库
spring-boot-starter-security|包含 spring-security。
spring-boot-starter-test|包含常用的测试所需的依赖，如 JUnit、Hamcrest、Mockito 和 spring-test 等。
spring-boot-starter-velocity|支持使用 Velocity 作为模板引擎。
**spring-boot-starter-web**|支持 Web 应用开发，包含 Tomcat 和 spring-mvc。
spring-boot-starter-websocket|支持使用 Tomcat 开发 WebSocket 应用。
spring-boot-starter-ws|支持 Spring Web Services
spring-boot-starter-actuator|添加适用于生产环境的功能，如性能指标和监测等功能。
spring-boot-starter-remote-shell|添加远程 SSH 支持
spring-boot-starter-jetty|使用 Jetty 而不是默认的 Tomcat 作为应用服务器。
spring-boot-starter-log4j|添加 Log4j 的支持
spring-boot-starter-logging|使用 Spring Boot 默认的日志框架 Logback
spring-boot-starter-tomcat|使用 Spring Boot 默认的 Tomcat 作为应用服务器。

### spring-boot-starter-web
- POM 文件中可以看到，应用所声明的依赖很少
- 只有一个“org.springframework.boot:spring-boot-starter-web”
- 而不是像其他 Spring 项目一样需要声明很多的依赖。
- 当使用 Maven 命令“mvn dependency:tree”来查看项目实际的依赖时
- 发现其中包含SpringMVC框架、SLF4J、Jackson、Hibernate Validator 和 Tomcat 等依赖。
- 这实际上 Spring 推荐的 Web 应用中使用的开源库的组合。

### EnableAutoConfiguration
- EnableAutoConfiguration”注解的作用在于让 Spring Boot 根据应用所声明的依赖来对 Spring 框架进行自动配置，这就减少了开发人员的工作量。
- Spring Boot 推荐采用基于 Java 注解的配置方式，而不是传统的 XML。只需要在主配置 Java 类上添加“@EnableAutoConfiguration”注解就可以启用自动配置。
- 注解“@RestController”和”@RequestMapping”由 Spring MVC 提供，用来创建 REST 服务。这两个注解和 Spring Boot 本身并没有关系。

### 查看mvn依赖结构
idea 中点击项目文件夹后点击idea界面左下角Terminal(相当于dos) 输入`mvn dependency:tree` 可以查看mvn项目依赖结构
``` dos
[INFO] com.chen.springboot:springboottext1:jar:1.0-SNAPSHOT
[INFO] +- org.springframework.boot:spring-boot-starter-web:jar:1.5.9.RELEASE:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter:jar:1.5.9.RELEASE:compile
[INFO] |  |  +- org.springframework.boot:spring-boot:jar:1.5.9.RELEASE:compile
[INFO] |  |  +- org.springframework.boot:spring-boot-autoconfigure:jar:1.5.9.RELEASE:compile
[INFO] |  |  +- org.springframework.boot:spring-boot-starter-logging:jar:1.5.9.RELEASE:compile
[INFO] |  |  |  +- ch.qos.logback:logback-classic:jar:1.1.11:compile
[INFO] |  |  |  |  +- ch.qos.logback:logback-core:jar:1.1.11:compile
[INFO] |  |  |  |  \- org.slf4j:slf4j-api:jar:1.7.25:compile
[INFO] |  |  |  +- org.slf4j:jcl-over-slf4j:jar:1.7.25:compile
[INFO] |  |  |  +- org.slf4j:jul-to-slf4j:jar:1.7.25:compile
[INFO] |  |  |  \- org.slf4j:log4j-over-slf4j:jar:1.7.25:compile
[INFO] |  |  +- org.springframework:spring-core:jar:4.3.13.RELEASE:compile
[INFO] |  |  \- org.yaml:snakeyaml:jar:1.17:runtime
[INFO] |  +- org.springframework.boot:spring-boot-starter-tomcat:jar:1.5.9.RELEASE:compile
[INFO] |  |  +- org.apache.tomcat.embed:tomcat-embed-core:jar:8.5.23:compile
[INFO] |  |  |  \- org.apache.tomcat:tomcat-annotations-api:jar:8.5.23:compile
[INFO] |  |  +- org.apache.tomcat.embed:tomcat-embed-el:jar:8.5.23:compile
[INFO] |  |  \- org.apache.tomcat.embed:tomcat-embed-websocket:jar:8.5.23:compile
[INFO] |  +- org.hibernate:hibernate-validator:jar:5.3.6.Final:compile
[INFO] |  |  +- javax.validation:validation-api:jar:1.1.0.Final:compile
[INFO] |  |  +- org.jboss.logging:jboss-logging:jar:3.3.1.Final:compile
[INFO] |  |  \- com.fasterxml:classmate:jar:1.3.4:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-databind:jar:2.8.10:compile
[INFO] |  |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.8.0:compile
[INFO] |  |  \- com.fasterxml.jackson.core:jackson-core:jar:2.8.10:compile
[INFO] |  +- org.springframework:spring-web:jar:4.3.13.RELEASE:compile
[INFO] |  |  +- org.springframework:spring-aop:jar:4.3.13.RELEASE:compile
[INFO] |  |  +- org.springframework:spring-beans:jar:4.3.13.RELEASE:compile
[INFO] |  |  \- org.springframework:spring-context:jar:4.3.13.RELEASE:compile
[INFO] |  \- org.springframework:spring-webmvc:jar:4.3.13.RELEASE:compile
[INFO] |     \- org.springframework:spring-expression:jar:4.3.13.RELEASE:compile
[INFO] \- junit:junit:jar:4.11:test
[INFO]    \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 45.779 s
[INFO] Finished at: 2019-04-04T10:54:36+08:00
[INFO] Final Memory: 23M/99M
[INFO] ------------------------------------------------------------------------
```

## SpringBoot WEB开发 
SpringBoot 使jar包也可以做web开发。   
在我们开发Web应用的时候，需要引用大量的js、css、图片等静态资源。
Spring Boot默认提供静态资源目录（位置需置于classpath下），目录名需符合如下规则：   
   **/static**   
   **/public**   
   **/resources**   
   **/META-INF/resources**   
![1.png](1.png)

目录优先级：META/resources > resources > static > public

举例：我们在上面四个资源路径下同时放入了4个同名的图片文件text.jpg，访问图片请求地址`http://localhost:8080/test.jpg` 会按照优先级显示


## 自动返回json数据
   @RestController 一般用于写API,给移动客户端提供数据，移动客户端一般接受数据类型为josn，@Controller 一般用于写后台代码编写（有页面）   
   使用@RestController 注解返回json数据   
   ``` java
   package com.chen.springboot.web.contorller;

   import com.chen.springboot.model.User;
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.ResponseBody;
   import org.springframework.web.bind.annotation.RestController;

   @RestController //声明Rest风格的控制器
   @EnableAutoConfiguration //自动配置,相当于写了Spring的配置文件
   @RequestMapping("user")
   public class UserContorller {

      @RequestMapping("{id}")
      @ResponseBody  //使用@ResponseBody会返回json数据
      /**
      * 通过id查询用户信息
      */
      public User hello(@PathVariable() Integer id){
         User user = new User("chen","123");
         user.setId(id);
         return user;
      }
   }
   ```

## 全局捕获异常
注释|说明
---|---
@ControllerAdvice|@Contorller的一个辅助类，最常用的就是作为全局异常处理的切面类，约定了几种可行的返回值，如果是直接返回 model 类的话，需要使用@ResponseBody 进行 json 转换     
@ExceptionHandler()|表示拦截异常
   ``` java 
   package com.chen.springboot.web.exception;

   import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
   import org.springframework.context.annotation.ComponentScan;
   import org.springframework.stereotype.Controller;
   import org.springframework.web.bind.annotation.ControllerAdvice;
   import org.springframework.web.bind.annotation.ExceptionHandler;
   import org.springframework.web.bind.annotation.ResponseBody;

   import java.util.HashMap;
   import java.util.Map;

   /**
   * 用于捕获全局异常
   */
   @ControllerAdvice  //声明控制器切面，用于捕获全局异常
   public class GlobalExceptionHandler {

      @ExceptionHandler(RuntimeException.class) //使用@ExceptionHandler 标记异常类型
      @ResponseBody  //返回json数据
      public Map<String,Object> exception(){  //处理异常的方法
         Map<String,Object> map = new HashMap<String,Object>();
         map.put("errorCode","101");
         map.put("errorMsg","系统错误!");
         return map;

      }
   }
   ```

## 渲染Web页面
### 模板引擎
SpringBoot 提供了多种模板引擎的默认配置支持，所以使用推荐的模板引擎下，我们可以快速的开发动态页面。
SptingBoot 提供了默认配置的模板引擎主要有一下几种：
- Thymeleaf
- FreeMarker
- Velocity
- Groovy
- Mustache

**Spring Boot 建议使用上述模板引擎，避免使用jsp，若一定使用jsp会无法完全实现SpringBoot的多种特性**
默认模板路径为：ser/mian/resources/templates


### 整合FreeMarker
1. 引入freemarker 依赖
   ``` xml 
   <!-- 引入freeMarker的依赖包. -->
   <dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-freemarker</artifactId>
   </dependency>
   ```

2. 创建FreeMarker模板文件
    
3. 编写控制器
   ``` java 
    
   ``` 




### 整合jsp
1. 新建war maven项目，导入依赖
   ``` xml
   <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>1.5.9.RELEASE</version>
   </parent>

   <dependencies>
      <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      </dependency>
      <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      </dependency>
      <!--jsp解析-->
      <dependency>
      <groupId>org.apache.tomcat.embed</groupId>
      <artifactId>tomcat-embed-jasper</artifactId>
      </dependency>
   </dependencies>
   ```

2. 创建resources资源文件夹，之后创建application.porperties 配置文件
   ``` properties
   spring.mvc.view.prefix=/WEB-INF/view/
   spring.mvc.view.suffix=.jsp
   ```

4. 创建controller
   ``` java
   package com.chen.controller;

   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
   import org.springframework.stereotype.Controller;
   import org.springframework.web.bind.annotation.RequestMapping;

   @Controller
   @EnableAutoConfiguration
   @RequestMapping("user")
   public class UserController {
      @RequestMapping("list")
      public String list() {
         return "list";
      }

      public static void main(String[] args) {
         SpringApplication.run(UserController.class,args);
      }

   }
   ```
5. 创建list.jsp页面 运行并且测试是否跳转



## 数据库访问
### 使用JDBC
1. 添加jdbc依赖
   ``` xml
      <!-- Spirng JDBC 依赖 -->
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-jdbc</artifactId>
      </dependency>
      <!-- 数据库驱动 -->
      <dependency>
         <groupId>mysql</groupId>
         <artifactId>mysql-connector-java</artifactId>
      </dependency>
      <!--单元测试,可不添加-->
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-test</artifactId>
         <scope>test</scope>
      </dependency>
   ```
2. application.properties中加入数据库配置信息
   ``` properties
   #数据库配置
   spring.datasource.url=jdbc:mysql://localhost:3306/day12
   spring.datasource.username=root
   spring.datasource.password=123456
   spring.datasource.driver-class-name=com.mysql.jdbc.Driver
   ```

3. 创建service
   ``` java
   package com.chen.springboot.service.impl;

   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.jdbc.core.JdbcTemplate;
   import org.springframework.stereotype.Service;

   @Service
   public class UserServiceImpl implements IUserService {

      @Autowired
      private JdbcTemplate jdbcTemplate;

      @Override
      public void register(String username, String password) {
         String sql = "insert into user(username,password) values(?,?)";
         jdbcTemplate.update(sql, username, password);
      }
   }
   ```

4. contorller中添加方法
   ``` java
      @Autowired
      private IUserService userService;
      @RequestMapping("register")
      @ResponseBody
      public String register(String username,String password){
         userService.register(username,password);
         return "success";
      }
   ```

5. SpringBoot启动类中设置扫描的包（加入service），并运行测试

### 使用Mybatis
1. 导入依赖
   ``` xml
   <parent>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-parent</artifactId>
         <version>1.3.2.RELEASE</version>
         <relativePath /> <!-- lookup parent from repository -->
      </parent>
      <dependencies>
         <!-- 这个导入spring-boot-starter-web时已经自动导入，可以不导 -->
         <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
         </dependency>
         <!-- 单元测试 -->
         <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
         </dependency>
         <!-- mybaties 主要导入这个-->
         <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.1.1</version>
         </dependency>
         <!-- mysql驱动 -->
         <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
         </dependency>
         <dependency> <groupId>org.springframework.boot</groupId> <artifactId>spring-boot-starter-web</artifactId> 
            </dependency>
      </dependencies>
   ```
2. application.properties中加入数据库配置信息
   ``` properties
   #数据库配置
   spring.datasource.url=jdbc:mysql://localhost:3306/day12
   spring.datasource.username=root
   spring.datasource.password=123456
   spring.datasource.driver-class-name=com.mysql.jdbc.Driver
   ```

3. 创建Mapper
   - 方式一：注释
   ``` java
   package com.chen.springboot.mapper;

   import com.chen.springboot.model.User;
   import org.apache.ibatis.annotations.Insert;
   import org.apache.ibatis.annotations.Param;
   import org.apache.ibatis.annotations.Select;

   public interface UserMapper {
      @Insert("insert into user(username,password) values(#{username},#{password})")
      public int save(@Param("username") String username, @Param("password") String password);
      @Select("select * from user where username = #{username}")
      public User findByUsername(@Param("username") String username);
   }
   ```
   - 方法二：xml映射
   ``` xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
   <mapper namespace="com.chen.springboot.mapper.UserMapper" >
      <!-- 接口中使用@param("username")所以取值使用#{username} 若没使用@param则用#{0} 取值 -->
      <insert id="save">
         insert into user (username,password) VALUES(#{username},#{password})
      </insert>
      <select id="findByUsername" resultType="com.chen.springboot.model.User" parameterType="string">
         select * from user where us ername = #{username,jdbcType=VARCHAR}
      </select>
   </mapper>
   ```

4. pom.xml配置文件中加入下面代码，解决打包时不自动打包xml文件
   ``` xml
   <build>
   <resources>
      <resource>
         <directory>src/main/java</directory>
         <includes>
         <include>**/*.xml</include>
         </includes>
      </resource>
   </resources>
   </build>
   ```
5. 创建service
   ``` java
   package com.chen.springboot.service.impl;

   import com.chen.springboot.mapper.UserMapper;
   import com.chen.springboot.service.IUserService;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Service;

   @Service
   public class UserServiceImpl implements IUserService {

      @Autowired
      private UserMapper userMapper;

      @Override
      public void register(String username, String password) {
         userMapper.save(username, password);
      }
   }
   ```
6. Controller 加入测试方法
   ``` java
   @Autowired
   private IUserService userService;
   @RequestMapping("register")
   @ResponseBody
   public String register(String username,String password){
      userService.register(username,password);
      return "success";
   }
   ```
6. App.class SpringBoot启动类中定义扫描mapper包,执行并测试
   ``` java
   package com.chen.springboot;

   import org.mybatis.spring.annotation.MapperScan;
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
   import org.springframework.context.annotation.ComponentScan;

   @ComponentScan(basePackages = {"com.chen.springboot.web","com.chen.springboot.service"})
   @MapperScan(basePackages = "com.chen.springboot.mapper")
   @EnableAutoConfiguration
   public class App {
      public static void main(String[] args) {
         //启动SpringBoot项目
         SpringApplication.run(App.class,args);
      }
   }
   ```

### 使用事务
在需要使用事务的service类上面使用`@Transactional`注解即可，可不配置事务管理器

### 配置多数据源
具体请查看另一篇博客 SpringBoot 多数据源以及多数据源事务管理


## 配置文件两种方式
在springBoot中使用application.xml和application.yml (Spring推荐) 两种形式的配置文件都可以起作用
1. application.xml 配置格式
   ``` properties
   #修改tomcat端口号
   server.port=8888
   #访问路径添加项目名字
   server.context-path=/test1
   ```
2. application.yml 配置格式
   ``` yml
   server:
   port:  8090
   context-path: /test-yml
   ```

## 整合log4j
1. 导入log4j.properties配置文件
   ``` properties
   log4j.rootLogger=INFO,Console,File   
   log4j.appender.Console=org.apache.log4j.ConsoleAppender  
   log4j.appender.Console.Target=System.out
   log4j.appender.Console.layout = org.apache.log4j.PatternLayout  
   log4j.appender.Console.layout.ConversionPattern=[%p] [%d{yyyy-MM-dd HH\:mm\:ss}][%c - %L]%m%n
   
   log4j.appender.File = org.apache.log4j.RollingFileAppender  
   log4j.appender.File.File = C:/Users/10301/Desktop/test/logs/info/info.log 
   log4j.appender.File.MaxFileSize = 10MB  
   
   log4j.appender.File.Threshold = ALL  
   log4j.appender.File.layout = org.apache.log4j.PatternLayout  
   log4j.appender.File.layout.ConversionPattern =[%p] [%d{yyyy-MM-dd HH\:mm\:ss}][%c - %L]%m%n  
   ```

2. pom.xml中排除logging依赖 加入log4j依赖
   这里排除springboot logging 是因为log4j没有起作用，所以我们排除后自己添加log4j依赖，若maven提示没有找到log4j 依赖我们需要定义版本号(Springboot父依赖版本号过高)
   ``` xml
    <!--排除SptingBoot自带的logging日志-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
      <exclusions>
        <exclusion>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
      </exclusions>
   </dependency>

    <!--重新导入log4j-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-log4j</artifactId>
      <version>1.3.8.RELEASE</version>
    </dependency>
   ```

3. 编写controller用于测试
   ``` java
   package com.chen.springboot.web.contorller;

   import com.chen.springboot.db1.service.UserService;
   import org.apache.log4j.Logger;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.ResponseBody;
   import org.springframework.web.bind.annotation.RestController;


   @RestController //声明Rest风格的控制器
   @EnableAutoConfiguration //自动配置,相当于写了Spring的配置文件
   @RequestMapping("user")
   public class UserController {

      @Autowired
      private UserService userService;

      Logger logger = Logger.getLogger(UserController.class);

      @RequestMapping("register")
      @ResponseBody
      public String register(String username, String password, String tel) {
         //log4j记录客户端请求参数
         logger.info("ip:" + "省略" + "username:" + username + "password:" + "tel:" + tel);
         //数据保存到数据库
         userService.register(username, password, tel);
         return "success";
      }
   }
   ```
## aop统一管理日志
1. 导入aop依赖
   ``` xml 
   <!--aop依赖-->
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-aop</artifactId>
   </dependency>
   ```
2. 创建用于web日志的切面类
   ``` java 
   package com.chen.springboot.aspect;

   import java.util.Enumeration;

   import javax.servlet.http.HttpServletRequest;

   import org.apache.log4j.Logger;

   import org.aspectj.lang.JoinPoint;
   import org.aspectj.lang.annotation.AfterReturning;
   import org.aspectj.lang.annotation.Aspect;
   import org.aspectj.lang.annotation.Before;
   import org.aspectj.lang.annotation.Pointcut;
   import org.springframework.stereotype.Component;
   import org.springframework.web.context.request.RequestContextHolder;
   import org.springframework.web.context.request.ServletRequestAttributes;
   @Aspect
   @Component
   public class WebLogAspect {
      private Logger logger = Logger.getLogger(getClass());

      @Pointcut("execution(public * com.chen.springboot.web.contorller.*.*(..))")
      public void webLog() {

      }

      @Before("webLog()")
      public void doBefore(JoinPoint joinPoint) throws Throwable {
         // 接收到请求，记录请求内容
         ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
         HttpServletRequest request = attributes.getRequest();
         // 记录下请求内容
         logger.info("---------------request----------------");
         logger.info("URL : " + request.getRequestURL().toString());
         logger.info("HTTP_METHOD : " + request.getMethod());
         logger.info("IP : " + request.getRemoteAddr());
         Enumeration<String> enu = request.getParameterNames();
         while (enu.hasMoreElements()) {
               String name = (String) enu.nextElement();
               logger.info("name:" + name + "value" + request.getParameter(name));
         }
      }
      @AfterReturning(returning = "ret", pointcut = "webLog()")
      public void doAfterReturning(Object ret) throws Throwable {
         logger.info("---------------response----------------");
         // 处理完请求，返回内容
         logger.info("RESPONSE : " + ret);
      }
   }
   ```
3. 在启动类中加入切入类的扫描位置，并测试
