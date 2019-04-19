---
title: SSM整合（MAVEN）
date: 2019-03-29 13:15:00
tegs: 
    - Maven
    - SSH整合
categories:
    - Java
    - 框架整合
---
# 项目环境
- JDK1.8
- Tomcat 8.5
- MySql 5.7
- Spring 4.x
- MyBatis 3.x

# 创建MAVEN项目
## 准备工作
IDEA中设置MAVEN相关配置   
![0.png](0.png)
## 创建父工程
1. 新建MAVEN项目 （new project）  
    ![1.png](1.png)   
    ![2.png](2.png)   
    ![3.png](3.png)   

2. pom.xml 中定义父工程打包方式
    ``` xml
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>

        <groupId>com.chen.edu</groupId>
        <artifactId>common-parent</artifactId>
        <version>1.0-SNAPSHOT</version>

        <!--定义父工程打包方式 pom（父工程要用pom方式）-->
        <packaging>pom</packaging>

    </project>
    ```
3. MAVEN父工程不写代码所以 删除src文件夹

## 创建子工程
1. 创建 model、dao、service、manager后台管理、api等子工程模块（new model），在父工程项目上面右键选择new>>model  
    ![4.png](4.png)   
    ![5.png](5.png)

2. 除manager、api模块使用maven web模板创建，上面其余模块使用普通maven工程创建

3. 在所有子工程pom.xml配置文件中定义子工程打包方式为jar，这里manager、api模块默认已经为我们定义了打包方式为war，所以这两个模块不用定义

## 创建公共依赖和统一管理版本号范例
在父工程中pom.xml文件中
``` xml
    <!--公共版本号管理-->
    <properties>
        <junit.version>4.11</junit.version>
    </properties>

    <!--创建公共依赖-->
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

# ssm整合
## manager模块编写
1. manager模块pom配置文件中添加spring/springmvc依赖
    ``` xml
    <dependencies>
            <!--添加Spring/SpringMVC 依赖-->
            <!--aop-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-aop</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!--aspects-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-aspects</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!--beans-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-beans</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!--context-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!--core-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!--expression-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-expression</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!--jdbc-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-jdbc</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!--orm-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-orm</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!--test-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-test</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!--tx-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-tx</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!--web-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-web</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <!--webmvc-->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-webmvc</artifactId>
                <version>${spring.version}</version>
            </dependency>
        </dependencies>
    ```
2. 配置tomcat   
![6.png](6.png)   
![7.png](7.png)   

3. 执行tomcat 测试   
![8.png](8.png)   

4. 添加SpringMVC配置文件   
首先在manager模块中src/main文件夹下创建java（idea中设置为Sources root）文件夹 和resources（idea中设置为Resources root）文件夹，创建配置文件中需要扫描的包`com.chen.edu.web.controller`，之后创建springmvc.xml配置文件
   ``` xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:mvc="http://www.springframework.org/schema/mvc"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xmlns:tx="http://www.springframework.org/schema/tx"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-4.3.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx-4.3.xsd">

        <!-- 1.注解扫描位置-->
        <context:component-scan base-package="com.chen.edu.web.controller" />

        <!-- 2.配置映射处理和适配器-->
        <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>
        <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>

        <!-- 3.视图的解析器-->
        <bean
                class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <property name="prefix" value="/WEB-INF/Modules/" />
            <property name="suffix" value=".jsp" />
        </bean>
    </beans>
   ```

5. 修改web.xml为3.0约束和添加SpringMVC配置   
   在manager模块的web.xml中添加SpringMVC拦截相关设置
    ``` xml
    <!-- 配置springmvc-->
    <?xml version="1.0" encoding="UTF-8"?>
    <web-app
        version="3.0"
        xmlns="http://java.sun.com/xml/ns/javaee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">

    <display-name>Archetype Created Web Application</display-name>

    <!--配置SpringMVC拦截-->
    <servlet>
        <servlet-name>DispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>DispatcherServlet</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
    </web-app>
    ```
6. 创建测试类
    ``` java
    package com.chen.edu.web.controller;

    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.RequestMapping;

    @Controller
    @RequestMapping("user")
    public class UserContorller {

        @RequestMapping("login")
        public String addUser(){
            System.out.println("login执行了");
            return null;
        }
    }
    ```

## model模块编写
1. 创建dao所需要的User模型
    ``` java
    package com.chen.edu.model;

    public class User {
        private String username;
        private String password;

        @Override
        public String toString() {
            return "User{" +
                    "username='" + username + '\'' +
                    ", password='" + password + '\'' +
                    '}';
        }

        public String getUsername() {
            return username;
        }

        public void setUsername(String username) {
            this.username = username;
        }

        public String getPassword() {
            return password;
        }

        public void setPassword(String password) {
            this.password = password;
        }
    }

    ```

## dao模块编写 

1. pom.xml文件添加model模块依赖 和MyBatis依赖 
    ``` xml
        <dependencies>
            <!--依赖model模块-->
            <dependency>
                <artifactId>edu-model</artifactId>
                <groupId>com.chen.edu</groupId>
                <version>1.0-SNAPSHOT</version>
            </dependency>
            <!-- MyBatis依赖 -->
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
                <version>${mybatis.version}</version>
            </dependency>
        </dependencies>
    ```
2. 在主工程项目pom.xml文件中定义mybatis版本
    ``` xml
    <mybatis.version>3.4.1</mybatis.version>
    ```

3. 创建抽取接口BaseMapper
    ``` java
    package com.chen.edu.mapper.base;

    public interface BaseMapper<T> {

        public T findByID(Integer id);
        public T findByUUID(String UUID);

        public void deleteByID(Integer id);
        public void deleteByUUID(String UUID);

        public void update(T t);

        public void insert(T t);
    }
    ```

4. 创建User接口 继承BaseMapper 接口
    ``` java
    package com.chen.edu.mapper;


    import com.chen.edu.model.User;
    import com.chen.edu.mapper.base.BaseMapper;

    public interface UserMapper extends BaseMapper<User> {

    }
    ```

5. 创建UserMapper.xml 用于测试
    ``` xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
    <mapper namespace="com.gyf.edu.mapper.UserMapper" >

        <select id="findByID" resultType="user" parameterType="int">
            SELECT * FROM t_user WHERE id = #{id}
        </select>
    </mapper>
    ```

## service模块编写
1. pom.xml添加dao依赖 和Spring包依赖（用于使用@Autowired、@Service和@Transactional）
    ``` xml
        <dependency>
            <artifactId>edu-dao</artifactId>
            <groupId>com.chen.edu</groupId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>${spring.version}</version>
        </dependency>
    ```

2. 创建抽取接口 IBaseService
    ``` java
    package com.chen.edu.service.base;

    public interface IBaseService<T> {
        public T findByID(Integer id);
        public T findByUUID(String UUID);

        public void deleteByID(Integer id);
        public void deleteByUUID(String UUID);

        public void update(T t);

        public void insert(T t);
    }
    ```

3. 创建抽象类 BaseServiceImpl 实现 IBaseService 接口
    ``` java
    package com.chen.edu.service.base;

    import com.chen.edu.mapper.UserMapper;
    import org.springframework.beans.factory.annotation.Autowired;

    public abstract class BaseServiceImpl<T> implements IBaseService<T>{

        //统一管理dao
        @Autowired
        protected UserMapper userMapper;
    }
    ```

4. 创建IUserService 继承 IBaseService
    ``` java
    package com.chen.edu.service;

    import com.chen.edu.User;
    import com.chen.edu.service.base.IBaseService;

    public interface IUserService extends IBaseService {
        //特有的方法 例如登陆
        public User login(String username,String password);
    }
    ```

5. 创建 UserServiceImpl 继承 IUserService   
    ``` java
    package com.chen.edu.service.impl;

    import com.chen.edu.model.User;
    import com.chen.edu.service.IUserService;
    import com.chen.edu.service.base.BaseServiceImpl;
    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;

    @Service
    @Transactional
    public class UserServiceImpl extends BaseServiceImpl<User> implements IUserService {

        @Override
        public User login(String username, String password) {
            return null;
        }

        @Override
        public User findByID(Integer id) {
            return userMapper.findByID(id);
        }

        @Override
        public User findByUUID(String UUID) {
            return null;
        }

        @Override
        public void deleteByID(Integer id) {

        }

        @Override
        public void deleteByUUID(String UUID) {

        }

        @Override
        public void update(User user) {

        }

        @Override
        public void insert(User user) {

        }
    }   
    ```


## 完善manager模块依赖
1. pom.xml添加service依赖
    ``` xml
        <dependency>
            <groupId>com.chen.edu</groupId>
            <artifactId>edu-service</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    ```
2. 添加其他依赖   

    依赖|描述
    ---|---
    Spring+ SpringMVC |之前已经导入
    MyBatis |
    Mybatis-spring整合包 |
    AOP联盟+织入 |之前导入jar包中自动导入了相关依赖
    c3p0 数据库连接池 |
    MySQL连接驱动 + jstl |

   主工程pom.xml 抽取相关依赖版本号（前三个之前已经定义）
   ``` xml
    <properties>
    <junit.version>4.11</junit.version>
    <spring.version>4.3.14.RELEASE</spring.version>
    <mybatis.version>3.4.1</mybatis.version>
    <mybatis-spring.version>1.3.1</mybatis-spring.version>
    <log4j.version>1.2.17</log4j.version>
    <slf4j.version>1.7.12</slf4j.version>
    <jstl.version>1.2</jstl.version>
    <taglibs.version>1.1.2</taglibs.version>
    <c3p0.version>0.9.5.2</c3p0.version>
    <mysql.version>5.1.35</mysql.version>
    </properties>
   ```

   manager pom.xml加入相关依赖
   ```xml
    <!--mybatis 和Spring整合包-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>${mybatis-spring.version}</version>
    </dependency>    
   <!-- 数据库驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.version}</version>
    </dependency>
    <!-- 数据连接池 -->
    <dependency>
        <groupId>com.mchange</groupId>
        <artifactId>c3p0</artifactId>
        <version>${c3p0.version}</version>
    </dependency>

    <!-- jstl标签类 -->
    <dependency>
        <groupId>jstl</groupId>
        <artifactId>jstl</artifactId>
        <version>${jstl.version}</version>
    </dependency>
    <dependency>
        <groupId>taglibs</groupId>
        <artifactId>standard</artifactId>
        <version>${taglibs.version}</version>
    </dependency>
    <!-- log start -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>${log4j.version}</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>${slf4j.version}</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>${slf4j.version}</version>
    </dependency>
    ```
3. 其他配置文件   
   在manager模块 src/main/resource文件夹创建相关配置文件   
   applicationContext.xml   
   ``` xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xmlns:tx="http://www.springframework.org/schema/tx"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context-4.3.xsd
            http://www.springframework.org/schema/aop
            http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
            http://www.springframework.org/schema/tx
            http://www.springframework.org/schema/tx/spring-tx-4.3.xsd">

        <context:property-placeholder location="classpath:db.properties"/>

        <!-- 配置数据源 -->
        <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
            <property name="driverClass" value="${jdbc.driver}"/>
            <property name="jdbcUrl" value="${jdbc.url}"/>
            <property name="user" value="${jdbc.username}"/>
            <property name="password" value="${jdbc.password}"/>
            <property name="maxPoolSize" value="30"/>
            <property name="minPoolSize" value="2"/>
        </bean>

        <!-- 配置sessionFactory -->
        <bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sqlSessionFactoryBean">
            <property name="dataSource" ref="dataSource"/>
            <!-- 指定配置文件位置 -->
            <!--<property name="configLocation" value="classpath:mybatis.xml"/>-->
            <!--配置别名-->
            <property name="typeAliasesPackage" value="com.chen.edu.model"></property>

            <!--配置加载映射文件 UserMapper.xml-->
            <property name="mapperLocations" value="classpath:com/chen/edu/mapper/*Mapper.xml"/>
        </bean>

        <!-- 自动生成dao,mapper-->
        <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
            <property name="basePackage" value="com.chen.edu.mapper"/>
            <property name="sqlSessionFactoryBeanName" value="sqlSessionFactoryBean"/>
        </bean>

        <!--自动扫描Service-->
        <context:component-scan base-package="com.chen.edu"/>


        <!-- 配置事务-->
        <!-- 5.配置事务管理器 -->
        <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource" ref="dataSource"/>
        </bean>
        <!-- 6.开启事务注解-->
        <tx:annotation-driven></tx:annotation-driven>
    </beans>
   ```
   db.properties
   ``` properties
    jdbc.driver=com.mysql.jdbc.Driver
    jdbc.url=jdbc:mysql://localhost:3306/edu1
    jdbc.username=root
    jdbc.password=123456
   ```

   log4j.properties 
   ``` properties
    # Global logging configuration
    log4j.rootLogger=DEBUG, stdout
    # Console output...
    log4j.appender.stdout=org.apache.log4j.ConsoleAppender
    log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
    log4j.appender.stdout.layout.
   ```

   ~~mybatis.xml~~ **这个整合到了spring配置文件中 无需创建**   
   ``` xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <!--配置别名-->
        <typeAliases>
            <!--批量配置别名:指定批量定义别名的类包,别名为类名(首字母大小写都可以)-->
            <package name="com.chen.edu.model"/>
        </typeAliases>
        <mappers>
            <!--批量加载映射文件-->
            <package name="com.chen.edu.mapper" />
        </mappers>
    </configuration>
   ```

4. web.xml中配置加载spring配置文件
   ``` xml 
    <!--配置加载spring applicationContext.xml -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
   ```
5. 添加xml打包插件   
   在idea中默认不会对mapper.xml进行打包，我们需要在**dao模块**pom.xml使用xml打包插件（使用命令行maven命令可以正常打包xml）
   ``` xml
        <!--在<build>标签中加入-->
        <resources>
            <resource>
            <directory>src/main/java</directory>
                <includes>**.*xml</includes>
            </resource>
        </resources>
   ```
6. 完善contorller方法用于测试   
    ``` java
    package com.chen.edu.web.controller;

            import com.chen.edu.model.User;
            import com.chen.edu.service.IUserService;
            import org.springframework.beans.factory.annotation.Autowired;
            import org.springframework.stereotype.Controller;
            import org.springframework.web.bind.annotation.RequestMapping;

    @Controller
    @RequestMapping("user")
    public class UserContorller {

        @Autowired
        private IUserService userService;

        @RequestMapping("login")
        public String login(){
            System.out.println("login执行了");
            return null;
        }
        @RequestMapping("find")
        public User find(Integer id){
            System.out.println("find执行了");
            User user = userService.findByID(id);
            System.out.println(user);
            return user;
        }
    }
    ```


### 关闭idea自动检测配置Spring中Bean是否配置   
   ![9.png](9.png)   
   ![10.png](10.png)

## 整合后目录结构
   ![common-parent.png](common-parent.png)   
   ![edu-model.png](edu-model.png)   
   ![edu-dao.png](edu-dao.png)   
   ![edu-service.png](edu-service.png)   
   ![edu-manager.png](edu-manager.png)   


# web资源整合   
1. 将js、图片、css等资源放入manager模块webapp文件夹下，页面等资源放入WEB-INF下   
   ![11.png](11.png)   

2. 将所有.html 修改为.jsp   
3. pom.xml中加入servlet-api/jsp 依赖 （注意：`<scope>` 应为provided 不打如war包，tomcat带这两个包，这里只是为了jsp页面有提示）
   ``` xml
           <!--添加servlet-api/jsp 依赖-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>javax.servlet.jsp-api</artifactId>
            <version>2.3.1</version>
            <scope>provided</scope>
        </dependency>
   ```
4. 修改jsp文件中所有路径 路径前加上`${pageContext.request.contextPath}` 并修改为正确路径   
5. 修改contorller中retrun跳转为相应的jsp页面
6. 查看springMVC.xml配置文件中 视图解析器是否为正确地址
7. 启动tomcat测试页面是否正常跳转
8. 编写所有用于跳转的contorller，修改jsp页面中所有跳转的路径




# 抽取contorller
1. 创建BaseContorller 抽象类
   ``` java 
    package com.chen.edu.web.controller.base;

    import java.lang.reflect.ParameterizedType;

    public abstract class BaseContorller<T> {
        //定义静态变量,用于页面跳转
        public static String MANAGE_PAGE;
        public static String INFO_PAGE;
        public static String EDIT_PAGE;
        //定义静态常量,用于页面访问
        public final static String MANAGE = "manage";
        public final static String INFO = "info";
        public final static String EDIT = "edit";

        public BaseContorller() {
            //1.获取泛型的真实类型
            ParameterizedType type = (ParameterizedType) this.getClass().getGenericSuperclass();
            Class<?> modelClass = (Class<?>) type.getActualTypeArguments()[0];

            //2.获取模块名
            String modelName = modelClass.getSimpleName();

            //3.给静态变量赋值
            MANAGE_PAGE = modelName + "/" + MANAGE;
            INFO_PAGE = modelName + "/" + INFO;
            EDIT_PAGE = modelName + "/" + EDIT;
        }
    }
   ```

2. 修改相关ModelContorller 继承抽象类，使用静态变量和常量
   ``` java
    package com.chen.edu.web.controller;


            import com.chen.edu.model.User;
            import com.chen.edu.service.IUserService;
            import com.chen.edu.web.controller.base.BaseContorller;
            import org.springframework.beans.factory.annotation.Autowired;
            import org.springframework.stereotype.Controller;
            import org.springframework.web.bind.annotation.RequestMapping;

    @Controller
    @RequestMapping("user")
    public class UserContorller extends BaseContorller<User> {

        @Autowired
        private IUserService userService;

        @RequestMapping(MANAGE)
        public String manage(){
            return MANAGE_PAGE;
        }
        @RequestMapping(INFO)
        public String info(){
            return INFO_PAGE;
        }

        @RequestMapping(EDIT)
        public String edit(){
            return EDIT_PAGE;
        }

    }
   ```