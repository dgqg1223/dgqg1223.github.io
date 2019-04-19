---
title: Spring.3.事务与转账案例
ate: 2019-03-04 11:32:29
tags:
    - Spring
categories:
    - Java
    - 框架
---
1此处缺少事务回滚内容
2转账案例中事务没有起作用？
## 案例：转帐
### 环境搭建
1. 创建数据库表
``` sql
create table account(
  id int primary key auto_increment,
  username varchar(50),
  money int
);
insert into account(username,money) values('jack','10000');
insert into account(username,money) values('rose','10000');
```

2. 导入jar包  
核心：4+1  （core,context,beans,expression）+logging
aop ： 4 (aop联盟、spring aop、aspectj.weaving规范、spring aspect)  
数据库：2  （jdbc/tx）  
驱动：mysql  
连接池：c3p0  

3. 创建dao
``` java
public class AccountDaoImpl extends JdbcDaoSupport implements IAccountDao {
    public void out(String outer, Integer money) {
        //转出
        String sql = "update account set money = money - ? where username = ?";
        this.getJdbcTemplate().update(sql, money, outer);
    }
    public void in(String inner, Integer money) {
        //转入
        String sql = "update account set money = money + ? where username = ?";
        this.getJdbcTemplate().update(sql, money, inner);
    }
}
```

4. 创建service
``` java 
public class AccountService {
    private IAccountDao accountDao;
    public void setAccountDao(IAccountDao accountDao) {
        this.accountDao = accountDao;
    }
    public void transfer(String outer,String inner,Integer money){
        accountDao.out(outer,money);
        accountDao.in(inner,money);
    }
}
```
5. 创建Spring xml 控制反转
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p ="http://www.springframework.org/schema/p"
       xmlns:context ="http://www.springframework.org/schema/context"
       xmlns:aop ="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd">
    <context:property-placeholder location="classpath:db.properties"/>
    <bean class="com.mchange.v2.c3p0.ComboPooledDataSource" id="dataSource">
        <property name="driverClass" value="${driverClass}"/>
        <property name="jdbcUrl" value="${jdbcUrl}"/>
        <property name="user" value="${user}"/>
        <property name="password" value="${password}"/>
    </bean>
    <bean class="com.chen.dao.AccountDaoImpl" id="accountDao">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <bean class="com.chen.web.service.AccountService" id="accountService">
        <property name="accountDao" ref="accountDao"/>
    </bean>
</beans>
```

6. 测试
``` java
    @Test
    public void test1(){
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        AccountService accountService = (AccountService) context.getBean("accountService");
        accountService.transfer("jack","rose",100);
    }
```

### 手动管理事务【了解】
**spring底层使用 `TransactionTemplate` 事务模板进行操作。**
**了解底层即可，因为以后都是通过aop来配置事务**
1. service中 设置事务模板
``` java
public class AccountService {
    private IAccountDao accountDao;

    //配置spring事务模板 由Spring 依赖注入
    private TransactionTemplate transactionTemplate;

    public void setTransactionTemplate(TransactionTemplate transactionTemplate) {
        this.transactionTemplate = transactionTemplate;
    }

    public void setAccountDao(IAccountDao accountDao) {
        this.accountDao = accountDao;
    }

    public void transfer(String outer, String inner, Integer money) {
        this.transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) {
                //扣钱
                accountDao.out(outer, money);
                // 用于测试事务是否配置成功
                int i = 10 / 0;
                //进账
                accountDao.in(inner, money);
            }
        });
    }
}
```
2. Spring xml配置文件中配置依赖注入事务模板和配置事务模板所需的事务管理
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p ="http://www.springframework.org/schema/p"
       xmlns:context ="http://www.springframework.org/schema/context"
       xmlns:aop ="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:property-placeholder location="classpath:db.properties"/>
    <bean class="com.mchange.v2.c3p0.ComboPooledDataSource" id="dataSource">
        <property name="driverClass" value="${driverClass}"/>
        <property name="jdbcUrl" value="${jdbcUrl}"/>
        <property name="user" value="${user}"/>
        <property name="password" value="${password}"/>
    </bean>
    <bean class="com.chen.dao.AccountDaoImpl" id="accountDao">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--配置事务管理器-->
    <bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="txManager">
        <!--注入事务管理器需要的DateSource-->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <!--配置事务模板-->
    <bean class="org.springframework.transaction.support.TransactionTemplate" id="transactionTemplate">
        <!--注入事务模板所需要的事务管理器-->
        <property name="transactionManager" ref="txManager"></property>
    </bean>
    <bean class="com.chen.web.service.AccountService" id="accountService">
        <property name="accountDao" ref="accountDao"/>
        <!--注入事务模板-->
        <property name="transactionTemplate" ref="transactionTemplate"/>
    </bean>
</beans>
```
3. 测试代码省略 和搭建环境中的测试代码相同

### 工厂bean生成代理 半自动
**Spring提供 管理事务的代理工厂bean `TransactionProxyFactoryBean`底层是对`TransactionTemplate`事务模板的封装**
1. 创建dao接口与实现类 代码省略与环境搭建中的dao相同
2. 创建service接口
``` java
public interface IAccountService {
    public void transfer(String outer, String inner, Integer money);
}
```
3. 创建service实现类
``` java
public class AccountService implements IAccountService {
    private IAccountDao accountDao;
    public void setAccountDao(IAccountDao accountDao) {
        this.accountDao = accountDao;
    }
    public void transfer(String outer, String inner, Integer money) {
        //扣钱
        accountDao.out(outer, money);
        int i=10/0;
        //进账
        accountDao.in(inner, money);

    }
}
```
3. 创建Spring xml配置文件 使用`TransactionProxyFactoryBean` 工厂bean实现事务操作
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p ="http://www.springframework.org/schema/p"
       xmlns:context ="http://www.springframework.org/schema/context"
       xmlns:aop ="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:property-placeholder location="classpath:db.properties"/>
    <bean class="com.mchange.v2.c3p0.ComboPooledDataSource" id="dataSource">
        <property name="driverClass" value="${driverClass}"/>
        <property name="jdbcUrl" value="${jdbcUrl}"/>
        <property name="user" value="${user}"/>
        <property name="password" value="${password}"/>
    </bean>
    <!--配置事务管理器-->
    <bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="txManager">
        <!--注入事务管理器需要的DateSource-->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <!--配置事务模板-->
    <bean class="org.springframework.transaction.support.TransactionTemplate" id="transactionTemplate">
        <!--注入事务模板所需要的事务管理器-->
        <property name="transactionManager" ref="txManager"></property>
    </bean>
    <bean class="com.chen.dao.AccountDaoImpl" id="accountDao">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <bean class="com.chen.web.service.AccountService" id="accountService">
        <property name="accountDao" ref="accountDao"/>
    </bean>
    <!--配置工厂代理-->
    <bean id="proxyService" class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
        <!--接口-->
        <property name="proxyInterfaces" value="com.chen.web.service.IAccountService"></property>
        <!--目标对象-->
        <property name="target" ref="accountService"></property>
        <!--切面对象：Spring做了不用写 -->

        <!--事务管理器-->
        <property name="transactionManager" ref="txManager"/>

        <!--transactionAttributes 事务的属性（详情）配置 可以用来配置隔离级别
            key：写方法名
            value写 事务配置
            格式:PROPAGATION,ISOLATION,readOnly,-Exception,+Exception
	               传播行为	 隔离级别   是否只读   异常回滚  异常提交
        -->
        <property name="transactionAttributes">
            <props>
                <prop key="transfer">PROPAGATION_REQUIRED,ISOLATION_DEFAULT,+java.lang.ArithmeticException</prop>
            </props>
        </property>
    </bean>
</beans>
```
5. 测试 通过事务属性配置`+java.lang.ArithmeticException` 出现该异常后数据库仍然提交
``` java
   @Test
    public void test2(){
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        IAccountService accountService= (IAccountService) context.getBean("proxyService");
        accountService.transfer("jack","rose",100);
    }
```

### 基于AOP的事务配置（xml）【掌握】
1. 在Spring xml配置文件中配置事务管理器和事务管理器设置、把事务和切入点关联
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p ="http://www.springframework.org/schema/p"
       xmlns:context ="http://www.springframework.org/schema/context"
       xmlns:aop ="http://www.springframework.org/schema/aop"
       xmlns:tx ="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd
                           http://www.springframework.org/schema/tx
                           http://www.springframework.org/schema/tx/spring-tx.xsd">

    <context:property-placeholder location="classpath:db.properties"/>
    <bean class="com.mchange.v2.c3p0.ComboPooledDataSource" id="dataSource">
        <property name="driverClass" value="${driverClass}"/>
        <property name="jdbcUrl" value="${jdbcUrl}"/>
        <property name="user" value="${user}"/>
        <property name="password" value="${password}"/>
    </bean>
    <bean class="com.chen.dao.AccountDaoImpl" id="accountDao">
    <property name="dataSource" ref="dataSource"/>
    </bean>
    <bean class="com.chen.web.service.AccountService" id="accountService">
    <property name="accountDao" ref="accountDao"/>
    </bean>
    <!--使用AOP配置事务-->
    <!--1.配置事务管理器-->
    <bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="txManager">
        <!--注入事务管理器需要的DateSource-->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <!--2.配置通知事务管理器详情-->
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <!--事务详情：传播行为，隔离级别等-->
        <tx:attributes>
            <!--使用这种方式 传播行为和隔离级别使用默认的方式可以不用配置
                如：propagation="REQUIRED" isolation="DEFAULT-->
            <tx:method name="transfer" propagation="REQUIRED" isolation="DEFAULT"/>
        </tx:attributes>
    </tx:advice>
    <!--3.事务与切入点关联-->
    <aop:config>
        <!--配置切入点-->
        <!--<aop:pointcut id="myPointCut" expression="execution(* com.chen.web.service..*.*(..))"></aop:pointcut>-->
        <!--<aop:advisor advice-ref="" pointcut-ref="myPointCut"></aop:advisor>-->
        <!--下面这种写法等同于上面的写法，这种更简便-->
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.chen.web.service.AccountService.transfer(..))"></aop:advisor>
    </aop:config>
</beans>
```
2. dao层、service层、测试类不变 参考上面
### 基于AOP的事务配置（注解）【掌握】
1. 在需要使用事务得类或方法使用 `@Transactional`注解
``` java 
@Transactional(propagation = Propagation.REQUIRED,isolation = Isolation.DEFAULT)
public class AccountService implements IAccountService {
    private IAccountDao accountDao;
    public void setAccountDao(IAccountDao accountDao) {
        this.accountDao = accountDao;
    }
    public void transfer(String outer, String inner, Integer money) {
        //扣钱
        accountDao.out(outer, money);
        //int i=100/0;
        //进账
        accountDao.in(inner, money);
    }
}
```
2. 在Spring xml 中配置开启事务注解开发
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p ="http://www.springframework.org/schema/p"
       xmlns:context ="http://www.springframework.org/schema/context"
       xmlns:aop ="http://www.springframework.org/schema/aop"
       xmlns:tx ="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd
                           http://www.springframework.org/schema/tx
                           http://www.springframework.org/schema/tx/spring-tx.xsd">

    <bean class="com.chen.dao.AccountDaoImpl" id="accountDao">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <bean class="com.chen.web.service.AccountService" id="accountService">
        <property name="accountDao" ref="accountDao"/>
    </bean>

    <context:property-placeholder location="classpath:db.properties"/>
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${driverClass}"/>
        <property name="jdbcUrl" value="${jdbcUrl}"/>
        <property name="user" value="${user}"/>
        <property name="password" value="${password}"/>
    </bean>
    <!--1.配置事务管理器-->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <!--2.开启事务注解驱动-->
    <tx:annotation-driven transaction-manager="txManager"></tx:annotation-driven>
</beans>
```
3. 测试 代码省略 和基于AOPxml配置事务演示中的没有区别

### spring serlvet 整合
解决问题1：当我们在servlet中加载Spring配置文件时，每次访问servlet都会加载一次，所以Spring在`spring-web-3.2.0.RC2.jar`包中为我们提供了`ContextLoaderListener`用于在web.xml中配置监听器，在项目开始运行时候加载配置文件。  
解决问题2：`ContextLoaderListener`加载Spring配置文件加载默认路径查找文件`[/WEB-INF/applicationContext.xml]`,实际开发中Srping配置文件存放在src下所以需要在web.xml配置`ContextLoaderListener`中的属性`contextConfigLocation`定义Spring配置文件路径
1. 创建web项目导入jar包 新增 spring serlvet整合需要的jar包
``` java
spring-web-3.2.0.RC2.jar
//spring-framework-3.2.0.RC2\libs
```
2. 创建测试环境 service接口、service实现类 代码省略
3. src路径下创建Spring 配置文件 applicationContext.xml 配置文件中创建service
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/p
                           http://www.springframework.org/schema/p/spring-p.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/tx
                           http://www.springframework.org/schema/tx/spring-tx.xsd">
    <bean id="userService" class="com.chen.service.UserServiceImpl"></bean>
</beans>
```
4. 创建servlet 
``` java
@WebServlet("/register")
public class RegisterServlet extends javax.servlet.http.HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //获取参数
        String username=request.getParameter("username");
        //调用service
        //service 的创建，由Spring创建，从spring的容器获取

        IUserService userService = null;
        //传统方式创建方式
        //IUserService userService = new UserServiceImpl();
        /* 使用Spring为我们创建这种方式每次访问servlet都会加载一次配置文件
           通过spring-web.jar 包中定义的监听器 在web.xml 配置解决加载多次的问题*/
        //ApplicationContext context= new ClassPathXmlApplicationContext("applicationContext.xml");
        //userService = (IUserService) context.getBean("userService");

        //配置web.xml后获取ApplicationContext
        ApplicationContext context = WebApplicationContextUtils.getWebApplicationContext(this.getServletContext());
        userService = (IUserService) context.getBean("userService");
        userService.add(username);

        //响应
        response.getWriter().write("register success");
    }
}
```
5. web.xml中添加监听器 修改Spring 配置文件路径
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!--配置Spring配置文件加载路径 默认找[/WEB-INF/applicationContext.xml]-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>
    <!--监听器-->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
</web-app>
```
