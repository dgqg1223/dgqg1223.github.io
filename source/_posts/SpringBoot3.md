---
title: SpringBoot.3.SpringBoot+MyBatis+FreeMarker整合和打包部署
ate: 2019-03-04 11:32:29
tags:
    - SpringBoot
categories:
    - Java
    - 框架,框架整合
---
# SpringBoot+MyBatis+FreeMarker整合
1. pom.xml依赖
   ``` xml 
    <?xml version="1.0" encoding="UTF-8"?>

    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>

        <groupId>com.chen.springboot</groupId>
        <artifactId>springboottext3</artifactId>
        <version>1.0-SNAPSHOT</version>

        <properties>
            <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
            <maven.compiler.source>1.7</maven.compiler.source>
            <maven.compiler.target>1.7</maven.compiler.target>
        </properties>

        <parent>
            <!-- 父依赖 提供dependency management依赖管理，引入这个依赖后声明其他依赖就不需要定义版本号了 -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>1.5.9.RELEASE</version>
        </parent>
        <dependencies>
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


            <!-- springboot核心组件 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>

            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.11</version>
                <scope>test</scope>
            </dependency>

            <!-- 引入freeMarker的依赖包. -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-freemarker</artifactId>
            </dependency>
            <!-- MyBatis -->
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>1.1.1</version>
            </dependency>
            <!-- 数据库驱动 -->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
            </dependency>
            <!--单元测试-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <scope>test</scope>
            </dependency>
            <!--多事物统一管理-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-jta-atomikos</artifactId>
            </dependency>
            <!--aop依赖-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-aop</artifactId>
            </dependency>
            <!--druid连接池-->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>1.0.25</version>
            </dependency>


        </dependencies>

        <build>
            <!--解决打包时候不打入xml的问题-->
            <resources>
                <resource>
                    <directory>src/main/java</directory>
                    <includes>
                        <include>**/*.xml</include>
                    </includes>
                </resource>
            </resources>
        </build>
    </project>
   ```

2. appliction.properties配置druid连接池   
   SpringBoot 会为我们导入它默认的连接池，这里我们使用自定义的连接池druid连接池
   ``` properties
    spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
    spring.datasource.url=jdbc:mysql://localhost:3306/edu1?useUnicode=true&characterEncoding=utf-8
    spring.datasource.username=root
    spring.datasource.password=123456
    spring.datasource.driverClassName=com.mysql.jdbc.Driver
    #连接池的配置信息
    spring.datasource.initialSize=5
    spring.datasource.minIdle=5
    spring.datasource.maxActive=20
    spring.datasource.maxWait=60000
    spring.datasource.timeBetweenEvictionRunsMillis=60000
    spring.datasource.minEvictableIdleTimeMillis=300000
    spring.datasource.validationQuery=SELECT 1 FROM DUAL
    spring.datasource.testWhileIdle=true
    spring.datasource.testOnBorrow=false
    spring.datasource.testOnReturn=false
    spring.datasource.poolPreparedStatements=true
    spring.datasource.maxPoolPreparedStatementPerConnectionSize=20
    spring.datasource.filters=stat,wall,log4j
    spring.datasource.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
   ```

3. 通过注解配置数据源
   ``` java
    package com.chen.springboot.dbconfig;


    import com.alibaba.druid.pool.DruidDataSource;
    import org.apache.log4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.Primary;

    import javax.sql.DataSource;
    import java.sql.SQLException;

    @Configuration
    public class DruidDBConfig {
        private Logger logger = Logger.getLogger(DruidDBConfig.class);

        @Value("${spring.datasource.url}")
        private String dbUrl;

        @Value("${spring.datasource.username}")
        private String username;

        @Value("${spring.datasource.password}")
        private String password;

        @Value("${spring.datasource.driverClassName}")
        private String driverClassName;

        @Value("${spring.datasource.initialSize}")
        private int initialSize;

        @Value("${spring.datasource.minIdle}")
        private int minIdle;

        @Value("${spring.datasource.maxActive}")
        private int maxActive;

        @Value("${spring.datasource.maxWait}")
        private int maxWait;

        @Value("${spring.datasource.timeBetweenEvictionRunsMillis}")
        private int timeBetweenEvictionRunsMillis;

        @Value("${spring.datasource.minEvictableIdleTimeMillis}")
        private int minEvictableIdleTimeMillis;

        @Value("${spring.datasource.validationQuery}")
        private String validationQuery;

        @Value("${spring.datasource.testWhileIdle}")
        private boolean testWhileIdle;

        @Value("${spring.datasource.testOnBorrow}")
        private boolean testOnBorrow;

        @Value("${spring.datasource.testOnReturn}")
        private boolean testOnReturn;

        @Value("${spring.datasource.poolPreparedStatements}")
        private boolean poolPreparedStatements;

        @Value("${spring.datasource.maxPoolPreparedStatementPerConnectionSize}")
        private int maxPoolPreparedStatementPerConnectionSize;

        @Value("${spring.datasource.filters}")
        private String filters;

        @Value("{spring.datasource.connectionProperties}")
        private String connectionProperties;

        @Bean     //声明其为Bean实例
        @Primary  //在同样的DataSource中，首先使用被标注的DataSource
        public DataSource dataSource(){
            DruidDataSource datasource = new DruidDataSource();

            datasource.setUrl(this.dbUrl);
            datasource.setUsername(username);
            datasource.setPassword(password);
            datasource.setDriverClassName(driverClassName);

            //configuration
            datasource.setInitialSize(initialSize);
            datasource.setMinIdle(minIdle);
            datasource.setMaxActive(maxActive);
            datasource.setMaxWait(maxWait);
            datasource.setTimeBetweenEvictionRunsMillis(timeBetweenEvictionRunsMillis);
            datasource.setMinEvictableIdleTimeMillis(minEvictableIdleTimeMillis);
            datasource.setValidationQuery(validationQuery);
            datasource.setTestWhileIdle(testWhileIdle);
            datasource.setTestOnBorrow(testOnBorrow);
            datasource.setTestOnReturn(testOnReturn);
            datasource.setPoolPreparedStatements(poolPreparedStatements);
            datasource.setMaxPoolPreparedStatementPerConnectionSize(maxPoolPreparedStatementPerConnectionSize);
            try {
                datasource.setFilters(filters);
            } catch (SQLException e) {
                logger.error("druid configuration initialization filter", e);
            }
            datasource.setConnectionProperties(connectionProperties);

            return datasource;
        }
    }
   ```


4. appliction.properties 加入FreeMarker配置
   ``` properties
    #spring.freemarker.suffix=.ftl
    #spring.freemarker.templateEncoding=UTF-8
    #spring.freemarker.templateLoaderPath=classpath:/templates/
    #spring.freemarker.content-type=text/html

    #用于再freemarker页面取值 页面中使用${request.contextPath} 等同于jsp ${pageContext.request.contextPath}
    spring.freemarker.request-context-attribute=request
   ```

5. 创建 static 文件夹，放入页面图片、js等资源

6. 创建 templates 文件夹 使用.ftl 页面获取资源
   ``` ftl
    <!DOCTYPE html>
    <html lang="en">
    <head >
        <meta charset="UTF-8"/>
        <title></title>
        <script src="${request.contextPath}/js/jquery.js"></script>
        <script>
            $(function () {
                //添加图片点击事件
                $("#carImg").click(function() {
                    alert("js引用成功");
                });
            });
        </script>
    </head>
    <body>
    引入图片<br>
    <img id="carImg" src="${request.contextPath}/imgs/test.jpg" width="300px" height="200px"/>

    </body>
   ```

7. 创建用于测试的controller
   ``` java
    package com.chen.springboot.web.contorller;

    import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.RequestMapping;

    @Controller //声明Rest风格的控制器
    @EnableAutoConfiguration //自动配置,相当于写了Spring的配置文件
    @RequestMapping("user")
    public class UserController {

        @RequestMapping("test")
        public String test() {
            return "user/list";
        }
    }
   ```
8. 创建SpringBoot 启动类并启动测试
   ``` java
    package com.chen.springboot;

    import org.mybatis.spring.annotation.MapperScan;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
    import org.springframework.boot.context.properties.EnableConfigurationProperties;
    import org.springframework.context.annotation.ComponentScan;

    @EnableAutoConfiguration
    //@ComponentScan(basePackages = {"com.chen.springboot.dbconfig","com.chen.springboot.web", "com.chen.springboot.datasource", "com.chen.springboot.service", "com.chen.springboot.db2.service"})
    @ComponentScan(basePackages = "com.chen.springboot")
    //@MapperScan("com.chen.springboot.mapper")
    public class App {
        public static void main(String[] args) {
            //启动SpringBoot项目
            SpringApplication.run(App.class, args);
        }
    }
   ```


# 打包部署
## .jar
1. 在需要打包的项目pom.xml配置文件中加入下面代码
   ``` xml
    <build>
        <!--解决打包时候不打入xml的问题-->
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.*</include>
                </includes>
            </resource>
        </resources>
        <plugins>
            <!--<plugin>-->
                <!--&lt;!&ndash; 指定maven编译的jdk版本,如果不指定,maven3默认用jdk 1.5 maven2默认用jdk1.3 &ndash;&gt;-->
                <!--<groupId>org.apache.maven.plugins</groupId>-->
                <!--<artifactId>maven-compiler-plugin</artifactId>-->
                <!--<version>3.1</version>-->
                <!--<configuration>-->
                    <!--&lt;!&ndash; 一般而言，target与source是保持一致的，但是，有时候为了让程序能在其他版本的jdk中运行(对于低版本目标jdk，源代码中不能使用低版本jdk中不支持的语法)，会存在target不同于source的情况 &ndash;&gt;-->
                    <!--<source>1.8</source> &lt;!&ndash; 源代码使用的JDK版本 &ndash;&gt;-->
                    <!--<target>1.8</target> &lt;!&ndash; 需要生成的目标class文件的编译版本 &ndash;&gt;-->
                    <!--&lt;!&ndash;<encoding>UTF-8</encoding>&lt;!&ndash; 字符集编码 &ndash;&gt;&ndash;&gt;-->
                    <!--&lt;!&ndash;<skipTests>true</skipTests>&lt;!&ndash; 跳过测试 &ndash;&gt;&ndash;&gt;-->
                    <!--<verbose>true</verbose>-->
                    <!--<showWarnings>true</showWarnings>-->
                    <!--<fork>true</fork>&lt;!&ndash; 要使compilerVersion标签生效，还需要将fork设为true，用于明确表示编译版本配置的可用 &ndash;&gt;-->
                    <!--<executable>&lt;!&ndash; path-to-javac &ndash;&gt;</executable>&lt;!&ndash; 使用指定的javac命令，例如：<executable>${JAVA_1_4_HOME}/bin/javac</executable> &ndash;&gt;-->
                    <!--<compilerVersion>1.3</compilerVersion>&lt;!&ndash; 指定插件将使用的编译器的版本 &ndash;&gt;-->
                    <!--<meminitial>128m</meminitial>&lt;!&ndash; 编译器使用的初始内存 &ndash;&gt;-->
                    <!--<maxmem>512m</maxmem>&lt;!&ndash; 编译器使用的最大内存 &ndash;&gt;-->
                    <!--&lt;!&ndash;<compilerArgument>-verbose -bootclasspath ${java.home}\lib\rt.jar</compilerArgument>&lt;!&ndash; 这个选项用来传递编译器自身不包含但是却支持的参数选项 &ndash;&gt;&ndash;&gt;-->
                <!--</configuration>-->
            <!--</plugin>-->

            <!--打包插件-->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.chen.springboot.App</mainClass>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>

            </plugin>
        </plugins>
    </build>
    ```

2. 使用 `mvn clean install` 打包项目
3. 使用`java -jar target/springboottext3-1.0-SNAPSHOT.jar` 运行项目，并在浏览器中测试