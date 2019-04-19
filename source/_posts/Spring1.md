---
title: Spring.1.基础入门
date: 2019-03-03 11:32:29
tags:
    - spring
categories:
    - Java
    - 框架
---
## Spring 概述
Spring是一个轻量级的控制反转（IoC）和面向切面（AOP）的容器框架  

自己理解：Spring会创建我们所需要的类放入到Spring容器中，所以当我们多次使用一个类时候该类只创建一次，解决了原本开发中每次调用同一个类都会创建一个实例的问题。（解耦）
## Spring 优点
1. 方便解耦，简化开发：    
Spring就是一个大工厂，专门负责生成Bran,可以将所有对象创建和依赖关系维护由Spring管理  
2. AOP编程的支持：  
Spring提供面向切面编程，可以方便的实现对程序进行权限拦截、运行监控等功能  
3. 声明式事务的支持：  
值需要通过配置就可以完成对事务的管理，而无需手动编程  
4. 方便程序的测试：  
Spring 对 Junit4支持，可以通过直接方便的测试Spring程序  
5. 方便继承各种优秀框架：  
Spring不白痴各种优秀的开源框架，其内部提供了对各种优秀框架的支持(Strats、Hibernate、MyBatis、Quartz等)  
6. 降低JavaEE API 使用难度
对JavaEE 开发中一些难用的API（JDBC、javaMail、远程调用等），都提供了封装，使这些API应用难度大大降低

## Spring 体系结构
Spring 框架是一个分层架构,,它包含一系列的功能要素并被分为大约20个模块。这些模块分为Core Container、Data Access/Integration、Web、AOP（Aspect Oriented Programming)、Instrumentation和测试部分,如下图所示：  
![1.png](1.png)

## Spring 快速入门
### 编写流程

1. 下载Srping 开发包  https://spring.io/projects 选择 Spring Framework    

```
spring-framework-3.2.0.RC2-dist.zip 核心包 
spring-framework-3.0.2.RELEASE-dependencies.zip 依赖包
spring-framework-3.2.0.RC2-docs.zip 文档包
```
2. 导入相关jar包  
3. 配置Spring核心的xml  
4. 在程序中读取Spring的配置文件获取Bean（Bean其实就是一个new好的对象，放入到Spring容器中）  


### Spring的核心jar包
``` java
spring-core-3.2.2.RELEASE.jar
//包含Spring框架基本的核心工具类，Spring其它组件要都要使用到这个包里的类,是其它组件的基本核心。
spring-beans-3.2.2.RELEASE.jar
//所有应用都要用到的，它包含访问配置文件、创建和管理bean
//以及进行Inversion of Control(IoC) / Dependency Injection(DI)操作相关的所有类
spring-context-3.2.2.RELEASE.jar
Spring提供在基础IoC功能上的扩展服务，此外还提供许多企业级服务的支持,
//如邮件服务、任务调度、JNDI定位、EJB集成、远程访问、缓存以及各种视图层框架的封装等。
spring-expression-3.2.2.RELEASE.jar
//Spring表达式语言
com.springsource.org.apache.commons.logging-1.1.1.jar
//第三方的主要用于处理日志(在依赖包中)
//spring-framework-3.0.2.RELEASE-dependencies\org.apache.commons\com.springsource.org.apache.commons.logging\1.1.1
```

### Spring入门案例
1. 创建Spring项目  
核心5个jar包拷入文件夹
创建Spring项目时候选择 Use Library 使用本地jar包 （为了了解spring核心jar包，不让idea为我们下载最新的Spring包）
![3.png](3.png)

2. 创建一个简单service接口和实现

``` java
package com.chen.service;

public class UserServiceImpl implements IUserService{
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    private String name ;
    @Override
    public void add() {
        System.out.println("创建用户" + name);
    }
}
```

3. 创建beans.xml 将service配置到beans.xml 
xml约束在`spring-framework-3.2.0.RC2-docs\reference\html\xsd-config.html` 中可以找到  
![4.png](4.png)
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 配置一个bean【对象】 -->
    <bean id="userService" class="com.chen.service.UserServiceImpl">
        <!--依赖注入数据 依赖注入调用属性的set方法-->
        <property name="name" value="张三" />
    </bean>

</beans>
```


4. 创建测试类
``` java
package com.chen.test;

import com.chen.service.IUserService;
import com.chen.service.UserServiceImpl;
import org.junit.Test;
import org.springframework.beans.BeanUtils;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class TestSpring {
    @Test
    public void Test(){
        //使用原始UserService
        IUserService userService = new UserServiceImpl();
        userService.add();
    }

    @Test
    public void Test1(){
        //UserServiceImpl()从Spring容器获取，需要提前在beans.xml定义
        //1.加载Spring配置文件beans.xml，Spring内部会创建我们定义的对象 
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        //2.从Spring容器中获取 userService对象
        IUserService userService = (IUserService) context.getBean("userService");
        userService.add();

    }
```

## 控制反转【IoC】
IoC Inverse of Control 反转控制的概念，就是将原本在程序中手动创建UserService对象的控制权，交由Spring框架管理，简单说，就是创建UserService对象控制权被反转到了Spring框架

## 依赖注入【DI】
Dependency Injection 依赖注入，在Spring框架负责创建Bean对象时，动态的将依赖对象注入到Bean组件。

## Spring容器创建的三种方式（了解）
``` java
    @Test
    public void Test2() {
        //Spring 容器加载的三种方式
        //第一种方式:通过类路径加载（常用）
        // ClassPathXmlApplicationContext ClassPath指classes（src下）路径
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        IUserService userService = (IUserService) context.getBean("userService");
        //第二种方式：通过文件系统路径获得配置文件（通过绝对路径）
        ApplicationContext context1 = new FileSystemXmlApplicationContext("C:\\Users\\CHEN\\SpringDemo1\\src\\beans.xml");
        IUserService userService1 = (IUserService) context1.getBean("userService");
        //第三种方式：BeanFactory（了解）
        String path = "C:\\Users\\CHEN\\SpringDemo1\\src\\beans.xml";
        BeanFactory factory = new XmlBeanFactory(new FileSystemResource(path));
        IUserService userService2 = (IUserService) factory.getBean("userService");
    }
```

### Spring内部创建对象的原理
1. 解析xml文件获取类名，属性
2. 通过反射，用类型创建对象
3. 通过依赖注入给对象赋值

### BeanFactor与ApplicationContext对比
BeanFactor 采用延时加载，第一次getBean时候才会初始化Bean  
ApplicationContext是即时加载  
ApplicationContext对BeanFactory扩展，提供了更多的功能：国际化处理、事件传递、Bean自动装配、各应用层的Context实现  


## 装配Bean（xml定义bean标签的方式）
### 实例化Bean的三种方式
创建静态工厂
``` java
package com.chen.service;

public class UserServiceFactory1 {
    public static IUserService createUserService(){
        return new UserServiceImpl();
    }
}
```
创建实例工厂
``` java
package com.chen.service;

public class UserServiceFactory2 {
    public IUserService createUserService(){
        return new UserServiceImpl();
    }
}
```

beans.xml 配置文件
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 装配bean的三种方式，所谓的装配bean就是在xml写一个bean标签 -->
    <!-- 第一种方式：new 实现类 -->
    <bean id="userService" class="com.chen.service.UserServiceImpl"></bean>
    <!-- 第二种方式：通过静态工厂 这个方式需要使用jdk1.7 否则报错 -->
    <bean id="userServiceStaticFactory" class="com.chen.service.UserServiceFactory1"
          factory-method="createUserService"></bean>
    <!-- 第三种方式：通过实例工厂 -->
    <bean id="factory" class="com.chen.service.UserServiceFactory2"></bean>
    <bean id="userService3" factory-bean="factory" factory-method="createUserService"></bean>
</beans>
```

测试类
``` java
   @Test
    public void Test3() {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        //new 对象
        IUserService userService = (IUserService) context.getBean("userService");
        userService.add();
        //new 静态工厂
        IUserService userService1 = (IUserService) context.getBean("userServiceStaticFactory");
        userService.add();
        //new 实例工厂
        IUserService userService2 = (IUserService) context.getBean("userService3");
        userService.add();
    }

```

### bean 的作用域
类别|说明
---|---
**singleton**|在Spring IoC容器中仅存在一个Bean实例，Bean以单例方式存在，默认值
**prototype**|每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean()时 ，相当于执行new XxxBean()
request|每次HTTP请求都会创建一个新的Bean，该作用域仅适用于WebApplicationContext环境
session|同一个HTTP Session 共享一个Bean，不同Session使用不同Bean，仅适用于WebApplicationContext 环境
globalSession|一般用于Portlet应用环境，该作用域仅适用于WebApplicationContext 环境

```xml
    <!-- scope 作用域：
         prototype   单例 -默认
         singleton   多例
    -->
    <bean id="userService1" class="com.chen.service.UserServiceImpl" scope="singleton"></bean>
```

## bean 的生命周期【了解】
### 生命周期图   
![5.png](5.png)  
### 生命周期图解释   
1. instantiate bean对象实例化
2. populate properties 封装属性
3. 如果Bean实现`BeanNameAware` 执行 `setBeanName`
4. 如果Bean实现`BeanFactoryAware` 执行`setBeanFactory` ，**获取Spring容器**s
5. 如果存在类实现 `BeanPostProcessor`（后处理Bean） ，执行`postProcessBeforeInitialization`
6. 如果Bean实现`InitializingBean` 执行 `afterPropertiesSet` 
7. 调用`<bean init-method="init"> `指定初始化方法 init
8. 如果存在类实现 `BeanPostProcessor`（处理Bean） ，执行`postProcessAfterInitialization` **执行业务处理**
9. 如果Bean实现 `DisposableBean` 执行 `destroy`
10. 调用`<bean destroy-method="customerDestroy"> `指定销毁方法 `customerDestroy`

### 演示
1. User 模型
``` java
package com.chen.model;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.*;
import org.springframework.context.annotation.Bean;

import java.sql.SQLOutput;

public class User implements BeanNameAware, BeanFactoryAware, InitializingBean, DisposableBean {
    private String username;
    private String password;

    @Override
    public String toString() {
        return "User{" +
                "username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }

    public User() {
        System.out.println("1.实例化...");
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        System.out.println("2.赋值属性" + username);
        this.username = username;
    }

    public String getPassword() {

        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public void setBeanName(String s) {
        System.out.println("3.设置bean名字：" + s);
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        //工厂生成bean对象放进容器
        System.out.println("4.bean工厂" + beanFactory);
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("6.属性赋值完成...");
    }

    public void myInit(){
        System.out.println("7.自定义初始化方法...");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("9.bean被销毁");
    }

    public void myDestory(){
        System.out.println("10.自定义销毁方法");
    }
}

```
2. 定义BeanPostProcessor 处理类
``` java
package com.chen.model;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

public class MyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        //这里可以用于多个对象共同的事情的处理
        System.out.println("5.预处理" + bean + ":" + beanName);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("8.业务处理" + bean + ":" + beanName);
        return bean;
    }
}
```
3. xml配置bean 
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
  
  <bean id="user" class="com.chen.model.User" init-method="myInit" destroy-method="myDestory">
    <property name="username" value="123" />
  </bean>

  <!--配置bean处理器-->
  <bean id="beanProcess" class="com.chen.model.MyBeanPostProcessor" ></bean>
</beans>
```
4. 测试代码
``` java
    @Test
    public void Test4() throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        // bean的生命周期
        ApplicationContext context = new lassPathXmlApplicationContext("beans2.xml");
        User user = (User) context.getBean("user");
        System.out.println(user);

        //手动关闭容器
        context.getClass().getMethod("close").invoke(context);
    }
```

## 依赖注入 Bean属性（xml）
### 手动装配 使用xml配置
1. 通过构造方法注入 
``` xml
    <!--构造方法注入属性值-->
    <bean id="stu" class="com.chen.model.Student">
        <constructor-arg name="username" value="xiaoming"></constructor-arg>
        <constructor-arg name="password" value="123456"></constructor-arg>
    </bean>

    <!--通过索引加类型给构造方法赋值-->
    <bean id="stu1" class="com.chen.model.Student">
        <constructor-arg index="0" value="zhangsan" type="java.lang.String"></constructor-arg>
        <constructor-arg index="1" value="88888" type="java.lang.String"></constructor-arg>
    </bean>
```

2. 通过set方法注入属性值  
注入的实例必须有默认构造方法
``` xml
    <!--通过set方法注入-->
    <bean id="stu2" class="com.chen.model.Student">
        <property name="username" value="xiaozhang" />
        <property name="password" value="6666" />
        <property name="age" value="21" />
    </bean>
```
3. 通过P命名空间注入【了解】  
必须有set方法
需要声明`xmlns:p="http://www.springframework.org/schema/p"` 命名空间
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!--xmlns xml namespace:xml命名空间-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--通过p命名空间注入-->
    <bean id="stu3" class="com.chen.model.Student" p:username="chen" p:age="20" p:password="123" />
</beans>
```

### spring表达式SpEl 【了解】
对`<property>`进行统一编程，所有的内容都使用value  
`<property name="" value="#{表达式}">`  

表达式|描述
---|---
`#{123}` | 数字
`#{'jack'}`|字符串  
`#{beanId}`	|另一个bean引用  
`#{beanId.propName}`	|操作数据  
`#{beanId.toString()}`	|执行方法  
`#{T(类).字段|方法}`	|静态方法或字段  

使用表达式的好处  
字符串#{'abc'} 会当作对象 可以调用相关方法

#### SpEL表达式演示
``` xml
    <bean id="address" class="com.chen.model.Address">
        <property name="city" value="黑龙江"/>
    </bean>

    <!--SpEL表达式-->
    <bean id="customer" class="com.chen.model.Customer">
        <!--调用对象方法-->
        <property name="custmoerNmae" value="#{'chen'.toUpperCase()}"/>
        <!--调用静态方法-->
        <property name="testPI" value="#{T(java.lang.Math).PI}"/>
        <!--ref 引用两种方式-->
        <!--<property name="address" ref="address"/>-->
        <property name="address" value="#{address}"/>
        <!--使用SpEL表达式 可以调用对象属性赋值-->
        <property name="testAddressCity" value="#{address.city}"/>
    </bean>
```

### 集合注入
集合的注入都是给`<property>`添加子标签
      
描述|标签|备注
---|---|---
数组|`<array>`|
List|`<list>`|
Set|`<set>`|
Map|`<map>` |map存放k/v 键值对，使用`<entry>`描述
Properties|`<props>`|  `<prop key=""></prop>`

普通数据：`<value>`
引用数据：`<ref>`

``` xml
 <!--集合注入-->
    <bean id="programmer" class="com.chen.model.Programmer">
        <!--List数据注入-->
        <property name="cars">
            <list>
                <value>自行车</value>
                <value>宝马</value>
            </list>
        </property>
        <!--Set数据注入-->
        <property name="pets">
            <set>
                <value>熊</value>
                <value>猴子</value>
            </set>
        </property>
        <!--Map数据注入-->
        <property name="infos">
            <map>
                <entry key="name" value="chen"/>
                <entry key="age" value="22"/>
            </map>
        </property>
        <!--property数据注入-->
        <property name="mysqlInfos">
            <props>
                <prop key="username">root</prop>
                <prop key="password">123456</prop>
            </props>
        </property>
        <!--数组注入-->
        <property name="menbers">
            <array>
                <value>爸爸</value>
                <value>妈妈</value>
            </array>
        </property>
    </bean>
```


## @Component注解使用
在类中使用可以取代xml中的`<bean>`标签
注解|对应标签
---|---
@Component|`<bean class="">`  
@Component(id)|`<bean id ="" class="">`

### 使用@Component注解案例
在Spring中默认注解是不生效的 需要在xml中开启使用注解开发  
![6.png](6.png)
1. xml中设置使用注解开发
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!--xmlns xml namespace:xml命名空间-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd">
    <!--在类中使用@Component 相当于在xml文件中 配置了
    <bean class="com.chen.service.UserServiceImpl"></bean>-->
    <!--开启注解-->
    <context:annotation-config/>
    <!--告诉Spring注解使用的位置 该位置下所有的注解都会被检测并使用-->
    <context:component-scan base-package="com.chen"/>
</beans>
```
2. 在需要使用注解的类使用@Component注解  
若只使用@Component则要在测试中通过类型(类型可以是接口也可以是实现类)来获取，也可以使用@Commponent("id") 在使用时候通过id获取
3. 测试注解
``` java
    @Test
    public void Test11() {
        // 测试注解
        ApplicationContext context = new ClassPathXmlApplicationContext("beans3.xml");
        //如果@Component没有配置id，通过类型来获取
        UserServiceImpl userService = context.getBean(UserServiceImpl.class);
        userService.add();
    }
```

## web开发提供的衍生注解
在web开发中，提供了3个@Component 注解的衍生注解（功能与@Commpoent一样） 取代`<bean class="">`   
衍生3中注释的原因：解决了创建对象的顺序

注解|对应层
---|---
@Repository("指定名称")|dao层使用 @Component的衍生注解
@Service("指定名称")|service层使用 @Component的衍生注解
@Controller("指定名称")|web层使用 @Component的衍生注解

注解|作用
---|---
@Autowired|自动根据类型注入
@Qualifer("指定的名称")|指定自动注入的id名称
@Resource(name="指定的名称")|可以取代@Autowired和@Qualifer
@PostConstruct|自定义初始化
@PreDestory|自定义销毁
@Scipe("protorype")|通过注解配多例，不配置Spring默认单例

### 使用dao层、web层、service层的注解案例
模拟web操作、xml开启注解开发、定义好需要扫描注解的包
1. xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!--xmlns xml namespace:xml命名空间-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd">
    <!--在类中使用@Component 相当于在xml文件中 配置了
    <bean class="com.chen.service.UserServiceImpl"></bean>-->

    <!--开启注解-->
    <context:annotation-config/>

    <!--告诉Spring注解使用的位置 该位置下所有的注解都会被检测并使用-->
    <context:component-scan base-package="com.chen"/>

    <!--使用bean标签方式-->
    <!--<bean id="suerdao" class="com.chen.dao.UserDaoImpl"></bean>-->
    <!--<bean id="userservice" class="com.chen.service.UserServiceImpl">-->
        <!--<property name="userDao" ref="suerdao"/>-->
    <!--</bean>-->
    <!--<bean id="useraction" class="com.chen.web.action.UserAction">-->
        <!--<property name="userService" ref="userservice"/>-->
    <!--</bean>-->
</beans>
```
2. action
``` java
@Controller
public class UserAction {
    //使用@Autowired Spring自动注入，不需要get/set方法3
    @Autowired
    private IUserService userService;

    public void save(User user){
         userService.add(user);
    }
```
3. service
``` java
@Service 
public class UserServiceImpl implements IUserService {
    //使用@Autowried标签不用定义属性的set/get方法 Spring会自动注入
    @Autowired
    private IUserDao userDao;
    public void add(User user) {
        //调用Dao
        userDao.add(user);
    }
}
```

4. dao
``` java
@Repository
public class UserDaoImpl implements IUserDao {
    @Override
    public void add(User user) {
        System.out.println("dao添加用户:" + user);
    }
}
```
5. 测试
``` java
   @Test
    public void Test12() {
        // @Repository、@Service、@Controller 注解的使用
        // web开发流程 action -> service -> dao
        ApplicationContext context = new ClassPathXmlApplicationContext("beans3.xml");
        UserAction userAction = context.getBean(UserAction.class);
        User user =new User();
        user.setUsername("范冰冰");
        user.setPassword("132");
        userAction.save(user);
    }
```

### @PostConstruct与@PreDestory案例
``` java
@Repository
public class UserDaoImpl implements IUserDao {
    //取代<bean init-method="" destroy-method=""></bean>
    @PostConstruct
    public void myInit(){
        System.out.println("自定义的初始化方法...");
    }
    @PreDestroy
    public void myDestroy(){
        System.out.println("自定义的初始化方法...");
    }
}
```













