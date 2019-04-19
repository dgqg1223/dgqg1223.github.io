---
title: SpringBoot.2.多数据源以及多数据源事务管理
ate: 2019-03-04 11:32:29
tags:
    - SpringBoot
categories:
    - Java
    - 框架
---
# 多数据源
## 准备工作
1. 创建用于测试的数据库及表
    ``` sql
    create database db1;
    create database db2;
    use db1;
    CREATE table user(
        id  int PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50),
        password VARCHAR(50),
        email VARCHAR(50),
    birthday TIMESTAMP
    );
    use db2;
    CREATE table customer(
        id  int PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50),
        tel VARCHAR(50)
    );
    ```

2. 创建 maven jar项目 在application.properties中配置两个数据源`spring.datasource.test1` 为自定义命名规则，后面用注解解析
   ``` properties
    spring.datasource.db1.driverClassName=com.mysql.jdbc.Driver
    spring.datasource.db1.url=jdbc:mysql://localhost:3306/db1?useUnicode=true&characterEncoding=utf-8
    spring.datasource.db1.username=root
    spring.datasource.db1.password=123456

    spring.datasource.db2.driverClassName=com.mysql.jdbc.Driver
    spring.datasource.db2.url=jdbc:mysql://localhost:3306/db2?useUnicode=true&characterEncoding=utf-8
    spring.datasource.db2.username=root
    spring.datasource.db2.password=123456
   ```

## 配置多数据源
### 涉及注解

注解|作用
---|---
@Configuration|用于定义配置类，可替换xml配置文件，被注解的类内部包含有一个或多个被@Bean注解的方法，这些方法将会被AnnotationConfigApplicationContext或AnnotationConfigWebApplicationContext类进行扫描，并用于构建bean定义，初始化Spring容器。
@MapperScan(basePackages="", sqlSessionFactoryRef="")|使用该注释扫描需要的包中所有的mapper
@Bean(name="")|同@Configuration配合使用 初始化Spring容器时候构建这个Bean 
@Primary|主数据源,多数据源要声明一个主数据源，只能有一个主数据源
@ConfigurationProperties(prefix="")|读取配置文件中prefix属性自定义命令规则的配置信息 使用@bean标签 它可以把读取同类的配置信息自动封装成实体类
@Qualifier|用于引用@Bean中的name 从而引用name标记的那个Bean

### 创建数据源配置
1. 创建一个`com.chen.springboot.datesource.DataSource01` 类用于配置第一个数据源
   ``` java
    package com.chen.springboot.datasource;

    import org.apache.ibatis.session.SqlSessionFactory;
    import org.mybatis.spring.SqlSessionFactoryBean;
    import org.mybatis.spring.SqlSessionTemplate;
    import org.mybatis.spring.annotation.MapperScan;
    import org.springframework.beans.factory.annotation.Qualifier;
    import org.springframework.boot.autoconfigure.jdbc.DataSourceBuilder;
    import org.springframework.boot.context.properties.ConfigurationProperties;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.Primary;
    import org.springframework.jdbc.datasource.DataSourceTransactionManager;

    import javax.sql.DataSource;

    @Configuration//注解到springboot容器中
    @MapperScan(basePackages = "com.chen.springboot.db1.mapper", sqlSessionFactoryRef = "db1SqlSessionFactory")
    public class DataSource01 {

        /**
        * @return 返回db1数据库的数据源
        */
        @Bean(name = "db1DataSource")
        @Primary//主数据源,多数据源要声明一个主数据源，只能有一个主数据源
        @ConfigurationProperties(prefix = "spring.datasource.db1")
        public DataSource dateSource() {
            return DataSourceBuilder.create().build();
        }

        /**
        * @return 返回db1数据库的会话工厂
        */
        @Bean(name = "db1SqlSessionFactory")
        public SqlSessionFactory sqlSessionFactory(@Qualifier("db1DataSource") DataSource ds) throws Exception {
            SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
            bean.setDataSource(ds);

            return bean.getObject();
        }

        /**
        * @return 返回db1数据库的事务
        */
        @Bean(name = "db1TransactionManager")
        @Primary
        public DataSourceTransactionManager transactionManager(@Qualifier("db1DataSource") DataSource dataSource) {
            return new DataSourceTransactionManager(dataSource);
        }

        /**
        * @return 返回db1数据库的会话模版
        */
        @Bean(name = "db1SqlSessionTemplate")
        public SqlSessionTemplate sqlSessionTemplate(
                @Qualifier("db1SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
            return new SqlSessionTemplate(sqlSessionFactory);
        }
    }
   ```
   参考：Spring中applicationContext.xml 配置数据步骤
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
2. 创建第二个数据源
   ``` java
    package com.chen.springboot.datasource;

    import org.apache.ibatis.session.SqlSessionFactory;
    import org.mybatis.spring.SqlSessionFactoryBean;
    import org.mybatis.spring.SqlSessionTemplate;
    import org.mybatis.spring.annotation.MapperScan;
    import org.springframework.beans.factory.annotation.Qualifier;
    import org.springframework.boot.autoconfigure.jdbc.DataSourceBuilder;
    import org.springframework.boot.context.properties.ConfigurationProperties;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.jdbc.datasource.DataSourceTransactionManager;

    import javax.sql.DataSource;

    @Configuration//注解到springboot容器中
    @MapperScan(basePackages = "com.chen.springboot.db2.mapper", sqlSessionFactoryRef = "db2SqlSessionFactory")
    public class DataSource02 {

        /**
        * @return 返回db2数据库的数据源
        */
        @Bean(name = "db2DataSource")
        @ConfigurationProperties(prefix = "spring.datasource.db2")
        public DataSource dateSource() {
            return DataSourceBuilder.create().build();
        }

        /**
        * @return 返回db2数据库的会话工厂
        */
        @Bean(name = "db2SqlSessionFactory")
        public SqlSessionFactory sqlSessionFactory(@Qualifier("db2DataSource") DataSource ds) throws Exception {
            SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
            bean.setDataSource(ds);

            return bean.getObject();
        }

        /**
        * @return 返回db2数据库的事务
        */
        @Bean(name = "db2TransactionManager")
        public DataSourceTransactionManager transactionManager(@Qualifier("db2DataSource") DataSource dataSource) {
            return new DataSourceTransactionManager(dataSource);
        }

        /**
        * @return 返回db2数据库的会话模版
        */
        @Bean(name = "db2SqlSessionTemplate")
        public SqlSessionTemplate sqlSessionTemplate(
                @Qualifier("db2SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
            return new SqlSessionTemplate(sqlSessionFactory);
        }
    }
   ```

## 搭建测试环境
最终目录结构   
![1.png](1.png)

1. 创建数据源1(主数据源)的mapper层和service层代码用于测试
   ``` java
    package com.chen.springboot.db1.mapper;

    import org.apache.ibatis.annotations.Insert;
    import org.apache.ibatis.annotations.Param;

    public interface UserMapper {

        @Insert("insert user (username,password) values (#{username},#{password})")
        public int save(@Param("username") String username, @Param("password") String password);
    }
   ```
   ``` java
    package com.chen.springboot.db1.service;

    import com.chen.springboot.db1.mapper.UserMapper;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;

    @Service
    @Transactional
    public class UserService{

        @Autowired
        private UserMapper userMapper;
        public void register(String username, String password) {
            userMapper.save(username,password);
        }
    }
   ```
2. 创建数据源2的mapper层和service层代码用于测试
   ``` java
    package com.chen.springboot.db2.mapper;

    import org.apache.ibatis.annotations.Insert;
    import org.apache.ibatis.annotations.Param;

    public interface CustomerMapper {

        @Insert("insert customer (name,tel) values (#{name},#{tel})")
        public int save(@Param("name") String name, @Param("tel") String tel);
    }
   ```
   ``` java
    package com.chen.springboot.db2.service;

    import com.chen.springboot.db2.mapper.CustomerMapper;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;

    @Service
    @Transactional
    public class CustomerService {

        @Autowired
        private CustomerMapper customerMapper;
        public void save(String name, String tel) {
            customerMapper.save(name,tel);
        }
    }
   ```
3. 创建Controller 用于测试
    ``` java
    package com.chen.springboot.web.controller;

    import com.chen.springboot.db1.service.UserService;
    import com.chen.springboot.db2.service.CustomerService ;
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
        @Autowired
        private CustomerService customerService;

        @RequestMapping("register")
        @ResponseBody
        public String register(String username, String password, String tel) {
            //数据保存到db1数据库
            System.out.println(username+password+tel);
            userService.register(username, password);
            //数据保存到db2数据库
            customerService.save(username, tel);
            return "success";
        }
    }
    ```
4. 创建SpringBoot启动类
   ``` java
    package com.chen.springboot;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
    import org.springframework.context.annotation.ComponentScan;

    @ComponentScan(basePackages = {"com.chen.springboot.web", "com.chen.springboot.datasource", "com.chen.springboot.db1.service", "com.chen.springboot.db2.service"})
    @EnableAutoConfiguration
    public class App {
        public static void main(String[] args) {
            //启动SpringBoot项目
            SpringApplication.run(App.class, args);
        }
    }

   ```
5. 浏览器测试
   > http://localhost:8080/user/register?username=zs&password=116332&tel=177777777


# 多事务管理jta-atmiok
在多数据源（多数据库）演示中，我们配置了2个事务管理器，一个事务管理器只能管理他所配置的数据源。所以我们在需要使用事务情况下操作多个事务源时事务无法起到效果。使用atmiok可以


## 整合jta-atomikos 
1. 添加依赖
   ``` xml
        <!--多事物统一管理-->
        <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jta-atomikos</artifactId>
        </dependency>
   ```

2. appliction.properties配置文件
   ``` properties
    # Mysql 1
    mysql.datasource.db1.url = jdbc:mysql://123.56.25.134:3306/db1?useUnicode=true&characterEncoding=utf-8
    mysql.datasource.db1.username = root
    mysql.datasource.db1.password = 123456

    mysql.datasource.db1.minPoolSize = 3
    mysql.datasource.db1.maxPoolSize = 25
    mysql.datasource.db1.maxLifetime = 20000
    mysql.datasource.db1.borrowConnectionTimeout = 30
    mysql.datasource.db1.loginTimeout = 30
    mysql.datasource.db1.maintenanceInterval = 60
    mysql.datasource.db1.maxIdleTime = 60

    mysql.datasource.db1.dbQuery = select 1
    # Mysql 2
    mysql.datasource.db2.url =jdbc:mysql://123.56.25.134:3306/db2?useUnicode=true&characterEncoding=utf-8
    mysql.datasource.db2.username =root
    mysql.datasource.db2.password =123456
    mysql.datasource.db2.minPoolSize = 3
    mysql.datasource.db2.maxPoolSize = 25
    mysql.datasource.db2.maxLifetime = 20000
    mysql.datasource.db2.borrowConnectionTimeout = 30
    mysql.datasource.db2.loginTimeout = 30
    mysql.datasource.db2.maintenanceInterval = 60
    mysql.datasource.db2.maxIdleTime = 60
    mysql.datasource.db2.dbQuery = select 1
   ```

3. 编写配置模型   
   - 数据源配置模型1
      ``` java
        package com.chen.springboot.dbconfig;
        
        import org.springframework.boot.context.properties.ConfigurationProperties;

        @ConfigurationProperties("mysql.datasource.db1")
        public class DBConfig1 {
            private String url;
            private String username;
            private String password;
            private int minPoolSize;
            private int maxPoolSize;
            private int maxLifetime;
            private int borrowConnectionTimeout;
            private int loginTimeout;
            private int maintenanceInterval;
            private int maxIdleTime;
            private String dbQuery;

            public String getUrl() {
                return url;
            }

            public void setUrl(String url) {
                this.url = url;
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

            public int getMinPoolSize() {
                return minPoolSize;
            }

            public void setMinPoolSize(int minPoolSize) {
                this.minPoolSize = minPoolSize;
            }

            public int getMaxPoolSize() {
                return maxPoolSize;
            }

            public void setMaxPoolSize(int maxPoolSize) {
                this.maxPoolSize = maxPoolSize;
            }

            public int getMaxLifetime() {
                return maxLifetime;
            }

            public void setMaxLifetime(int maxLifetime) {
                this.maxLifetime = maxLifetime;
            }

            public int getBorrowConnectionTimeout() {
                return borrowConnectionTimeout;
            }

            public void setBorrowConnectionTimeout(int borrowConnectionTimeout) {
                this.borrowConnectionTimeout = borrowConnectionTimeout;
            }

            public int getLoginTimeout() {
                return loginTimeout;
            }

            public void setLoginTimeout(int loginTimeout) {
                this.loginTimeout = loginTimeout;
            }

            public int getMaintenanceInterval() {
                return maintenanceInterval;
            }

            public void setMaintenanceInterval(int maintenanceInterval) {
                this.maintenanceInterval = maintenanceInterval;
            }

            public int getMaxIdleTime() {
                return maxIdleTime;
            }

            public void setMaxIdleTime(int maxIdleTime) {
                this.maxIdleTime = maxIdleTime;
            }

            public String getdbQuery() {
                return dbQuery;
            }

            public void setdbQuery(String dbQuery) {
                this.dbQuery = dbQuery;
            }
        }
      ```
   - 数据源配置模型2
      ``` java 
        package com.chen.springboot.dbconfig;


        import org.springframework.boot.context.properties.ConfigurationProperties;

        @ConfigurationProperties("mysql.datasource.db2")
        public class DBConfig2 {
            private String url;
            private String username;
            private String password;
            private int minPoolSize;
            private int maxPoolSize;
            private int maxLifetime;
            private int borrowConnectionTimeout;
            private int loginTimeout;
            private int maintenanceInterval;
            private int maxIdleTime;
            private String dbQuery;

            public String getUrl() {
                return url;
            }

            public void setUrl(String url) {
                this.url = url;
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

            public int getMinPoolSize() {
                return minPoolSize;
            }

            public void setMinPoolSize(int minPoolSize) {
                this.minPoolSize = minPoolSize;
            }

            public int getMaxPoolSize() {
                return maxPoolSize;
            }

            public void setMaxPoolSize(int maxPoolSize) {
                this.maxPoolSize = maxPoolSize;
            }

            public int getMaxLifetime() {
                return maxLifetime;
            }

            public void setMaxLifetime(int maxLifetime) {
                this.maxLifetime = maxLifetime;
            }

            public int getBorrowConnectionTimeout() {
                return borrowConnectionTimeout;
            }

            public void setBorrowConnectionTimeout(int borrowConnectionTimeout) {
                this.borrowConnectionTimeout = borrowConnectionTimeout;
            }

            public int getLoginTimeout() {
                return loginTimeout;
            }

            public void setLoginTimeout(int loginTimeout) {
                this.loginTimeout = loginTimeout;
            }

            public int getMaintenanceInterval() {
                return maintenanceInterval;
            }

            public void setMaintenanceInterval(int maintenanceInterval) {
                this.maintenanceInterval = maintenanceInterval;
            }

            public int getMaxIdleTime() {
                return maxIdleTime;
            }

            public void setMaxIdleTime(int maxIdleTime) {
                this.maxIdleTime = maxIdleTime;
            }

            public String getdbQuery() {
                return dbQuery;
            }

            public void setdbQuery(String dbQuery) {
                this.dbQuery = dbQuery;
            }
        }
      ```


4. 创建数据源配置
   - 数据源配置1
      ``` java
        package com.chen.springboot.datasource;

        import com.atomikos.jdbc.AtomikosDataSourceBean;
        import com.chen.springboot.dbconfig.DBConfig1;
        import com.mysql.jdbc.jdbc2.optional.MysqlXADataSource;
        import org.apache.ibatis.session.SqlSessionFactory;
        import org.mybatis.spring.SqlSessionFactoryBean;
        import org.mybatis.spring.SqlSessionTemplate;
        import org.mybatis.spring.annotation.MapperScan;
        import org.springframework.beans.factory.annotation.Qualifier;
        import org.springframework.context.annotation.Bean;
        import org.springframework.context.annotation.Configuration;
        import org.springframework.context.annotation.Primary;

        import javax.sql.DataSource;
        import java.sql.SQLException;

        @Configuration//注解到springboot容器中
        @MapperScan(basePackages="com.chen.springboot.db1.mapper",sqlSessionFactoryRef="db1SqlSessionFactory")
        public class DataSource01 {

            // 配置数据源
            @Primary
            @Bean(name = "db1DataSource")
            public DataSource dbDataSource(DBConfig1 dbConfig) throws SQLException {
                MysqlXADataSource mysqlXaDataSource = new MysqlXADataSource();
                mysqlXaDataSource.setUrl(dbConfig.getUrl());
                mysqlXaDataSource.setPinGlobalTxToPhysicalConnection(true);
                mysqlXaDataSource.setPassword(dbConfig.getPassword());
                mysqlXaDataSource.setUser(dbConfig.getUsername());
                mysqlXaDataSource.setPinGlobalTxToPhysicalConnection(true);

                AtomikosDataSourceBean xaDataSource = new AtomikosDataSourceBean();
                xaDataSource.setXaDataSource(mysqlXaDataSource);
                xaDataSource.setUniqueResourceName("db1DataSource");

                xaDataSource.setMinPoolSize(dbConfig.getMinPoolSize());
                xaDataSource.setMaxPoolSize(dbConfig.getMaxPoolSize());
                xaDataSource.setMaxLifetime(dbConfig.getMaxLifetime());
                xaDataSource.setBorrowConnectionTimeout(dbConfig.getBorrowConnectionTimeout());
                xaDataSource.setLoginTimeout(dbConfig.getLoginTimeout());
                xaDataSource.setMaintenanceInterval(dbConfig.getMaintenanceInterval());
                xaDataSource.setMaxIdleTime(dbConfig.getMaxIdleTime());
                xaDataSource.setTestQuery(dbConfig.getdbQuery());
                return xaDataSource;
            }

            @Bean(name = "db1SqlSessionFactory")
            public SqlSessionFactory dbSqlSessionFactory(@Qualifier("db1DataSource") DataSource dataSource)
                    throws Exception {
                SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
                bean.setDataSource(dataSource);
                return bean.getObject();
            }

            @Bean(name = "db1SqlSessionTemplate")
            public SqlSessionTemplate dbSqlSessionTemplate(
                    @Qualifier("db1SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
                return new SqlSessionTemplate(sqlSessionFactory);
            }
        }
      ```

   - 数据源配置2
      ``` java
        package com.chen.springboot.datasource;

        import com.atomikos.jdbc.AtomikosDataSourceBean;
        import com.chen.springboot.dbconfig.DBConfig2;
        import com.mysql.jdbc.jdbc2.optional.MysqlXADataSource;
        import org.apache.ibatis.session.SqlSessionFactory;
        import org.mybatis.spring.SqlSessionFactoryBean;
        import org.mybatis.spring.SqlSessionTemplate;
        import org.mybatis.spring.annotation.MapperScan;
        import org.springframework.beans.factory.annotation.Qualifier;
        import org.springframework.context.annotation.Bean;
        import org.springframework.context.annotation.Configuration;

        import javax.sql.DataSource;
        import java.sql.SQLException;

        @Configuration//注解到springboot容器中
        @MapperScan(basePackages="com.chen.springboot.db2.mapper",sqlSessionFactoryRef="db2SqlSessionFactory")
        public class DataSource02 {

            // 配置数据源
            @Bean(name = "db2DataSource")
            public DataSource dbDataSource(DBConfig2 dbConfig) throws SQLException {
                MysqlXADataSource mysqlXaDataSource = new MysqlXADataSource();
                mysqlXaDataSource.setUrl(dbConfig.getUrl());
                mysqlXaDataSource.setPinGlobalTxToPhysicalConnection(true);
                mysqlXaDataSource.setPassword(dbConfig.getPassword());
                mysqlXaDataSource.setUser(dbConfig.getUsername());
                mysqlXaDataSource.setPinGlobalTxToPhysicalConnection(true);

                AtomikosDataSourceBean xaDataSource = new AtomikosDataSourceBean();
                xaDataSource.setXaDataSource(mysqlXaDataSource);
                xaDataSource.setUniqueResourceName("db2DataSource");

                xaDataSource.setMinPoolSize(dbConfig.getMinPoolSize());
                xaDataSource.setMaxPoolSize(dbConfig.getMaxPoolSize());
                xaDataSource.setMaxLifetime(dbConfig.getMaxLifetime());
                xaDataSource.setBorrowConnectionTimeout(dbConfig.getBorrowConnectionTimeout());
                xaDataSource.setLoginTimeout(dbConfig.getLoginTimeout());
                xaDataSource.setMaintenanceInterval(dbConfig.getMaintenanceInterval());
                xaDataSource.setMaxIdleTime(dbConfig.getMaxIdleTime());
                xaDataSource.setTestQuery(dbConfig.getdbQuery());
                return xaDataSource;
            }

            @Bean(name = "db2SqlSessionFactory")
            public SqlSessionFactory dbSqlSessionFactory(@Qualifier("db2DataSource") DataSource dataSource)
                    throws Exception {
                SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
                bean.setDataSource(dataSource);
                return bean.getObject();
            }

            @Bean(name = "db2SqlSessionTemplate")
            public SqlSessionTemplate dbSqlSessionTemplate(
                    @Qualifier("db2SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
                return new SqlSessionTemplate(sqlSessionFactory);
            }
        }
      ```
5. SpringBoot启动类   
   需要加入`@EnableConfigurationProperties`
   ``` java
   package com.chen.springboot;

        import com.chen.springboot.dbconfig.DBConfig1;
        import com.chen.springboot.dbconfig.DBConfig2;
        import org.springframework.boot.SpringApplication;
        import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
        import org.springframework.boot.context.properties.EnableConfigurationProperties;
        import org.springframework.context.annotation.ComponentScan;

        @EnableAutoConfiguration
        //@ComponentScan(basePackages = {"com.chen.springboot.dbconfig","com.chen.springboot.web", "com.chen.springboot.datasource", "com.chen.springboot.db1.service", "com.chen.springboot.db2.service"})
        @ComponentScan(basePackages = "com.chen.springboot")
        @EnableConfigurationProperties(value = {DBConfig1.class, DBConfig2.class})
        public class App {
            public static void main(String[] args) {
                //启动SpringBoot项目
                SpringApplication.run(App.class, args);
            }
        }
   ```
6. 修改controller、service 用于测试
   - controller
      ``` java
        package com.chen.springboot.web.contorller;

        import com.chen.springboot.db1.service.UserService;
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


            @RequestMapping("register")
            @ResponseBody
            public String register(String username, String password, String tel) {
                //数据保存到数据库
                userService.register(username, password,tel);
                return "success";
            }
        }
      ```
   - service
      ``` java
        package com.chen.springboot.db1.service;

        import com.chen.springboot.db1.mapper.UserMapper;
        import com.chen.springboot.db2.mapper.CustomerMapper;
        import org.springframework.beans.factory.annotation.Autowired;
        import org.springframework.stereotype.Service;
        import org.springframework.transaction.annotation.Transactional;

        @Service
        @Transactional
        public class UserService{

            @Autowired
            private UserMapper userMapper;
            @Autowired
            private CustomerMapper CustomerMapper;

            public void register(String username, String password,String tel) {
                CustomerMapper.save(username,tel);
                int a = 1/0;
                userMapper.save(username,password);
            }

        }
      ```

7. 数据源2mapper参考上面中的CustomerService
8. 运行测试事务是否起作用
9. 最终项目结构   
   ![2.png](2.png)