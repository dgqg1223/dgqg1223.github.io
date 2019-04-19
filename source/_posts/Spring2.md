---
title: Spring.2.AOP切面编程与JdbcTemplate
ate: 2019-03-04 11:32:29
tags:
    - spring
categories:
    - Java
    - 框架
---
## AOP 面向切面编程详解
### AOP 概述
1. 在软件业，**AOP为Aspect Oriented Programming**的缩写，意为：**面向切面编程**，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。
2. AOP是OOP（面向对象编程）的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。
3. 利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。
4. AOP采取横向抽取机制，取代了传统纵向继承体系重复性代码（耦合性高）
5. **经典应用：事务管理、性能监视、安全检查、缓存 、日志等【画图】**
6. Spring AOP使用纯Java实现，不需要专门的编译过程和类加载器，在运行期通过代理方式向目标类织入增强代码
7. **AspectJ是一个基于Java语言的AOP框架**，Spring2.0开始，Spring AOP引入对Aspect的支持，AspectJ扩展了Java语言，提供了一个专门的编译器，在编译时提供横向代码的织入（AspectJ理解为对Proxy的扩展）

### AOP 相关术语
1. target：目标类，需要被代理的类。例如：UserService
2. Joinpoint(连接点):所谓连接点是指那些可能被拦截到的方法。例如：所有的方法
3. PointCut 切入点：已经被增强的连接点。例如：addUser()
4. advice 通知/增强，增强代码。例如：after、before
5. Weaving(织入):是指把增强advice应用到目标对象target来创建新的代理对象proxy的过程.
6. proxy 代理类
7. Aspect(切面): 是切入点pointcut和通知advice的结合
	一个线是一个特殊的面。
	一个切入点和一个通知，组成成一个特殊的面。
![1.png](1.png)


### AOP 实现方式
aop底层采用代理机制进行实现
2. 使用jdk的动态代理Proxy 需要接口+实现类
3. 使用 cglib字节码增强框架 有接口的实现类和没有接口的实现类都可以实现


### 手动代理（使用Proxy实现）
1. 创建要被代理的对象接口
``` java
public interface IUserService {
    //接口
    public int deleteUser(int id);
}
```
2. 创建要被代理的对象实现类 
``` java
public class UserServiceImpl implements IUserService {
    //实现类
    public int deleteUser(int id) {
        System.out.println("UserServiceImpl的方法deleteUser");
        return 1;
    }
}
```
3. 创建用于增强被代理对象的切面类
``` java
/**
 * 切面类：增强代码 与 切入点结合
 */
public class MyAspect {
     public void before(){
         System.out.println("开启事务");
     }
     public void after(){
         System.out.println("提交事务");
     }
}

```
4. 创建用于实现面向切面的工厂对象(重点)
``` java
/**
 * 工厂类 用于实现切面
 */
public class MyBeanFactory {

    public static IUserService createUserService() {
        //1.创建目标对象
        final IUserService userService = new UserServiceImpl();
        //2.声明切面对象
        final MyAspect aspect = new MyAspect();
        //3.把切面类中定义的2个方法应用到目标类
        //3.1创建JDK代理 用于拦截方法
        /*参数 ClassLoader loader 类加载器，写当前类
              Class<?>[] interfaces 接口，接口的方法会被拦截
              InvocationHandler h 处理     */
        IUserService ServiceProxy = (IUserService) Proxy.newProxyInstance(MyBeanFactory.class.getClassLoader(),
                userService.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        //开启事务
                        aspect.before();
                        //方法的返回值是业务方法的返回值
                        Object retObj = method.invoke(userService,args);
                        System.out.println("拦截返回值："+retObj);
                        //提交事务
                        aspect.after();
                        return retObj;
                    }
                });


        return ServiceProxy;
    }
}
```
5. 测试
```java
  @Test
    public void test() {
        //自己实现AOP编程，使用JDK代理来实现
        IUserService userService = MyBeanFactory.createUserService();
        userService.deleteUser(10);
        //通过输出可以看到代理对象成功了
        /* MyAspect的before方法执行了
        UserServiceImpl的方法deleteUser
        拦截返回值：1
        MyAspect的after方法执行了 */
    }
```

### 手动代理（使用cglib增强字节码实现）
cglib 依赖包spring已经集成。
1. 创建被代理类
``` java
public class StudentService {
    public int delete(int i) {
        System.out.println("StudentService的delete执行了");
        return 1;
    }
}
```

2. 创建切面类
``` java
/**
 * 切面类：增强代码 与 切入点结合
 */
public class MyAspect {
     public void before(){
         System.out.println("开启事务...");
     }
     public void after(){
         System.out.println("提交事务...");
     }
}
```

3.  创建用于实现面向切面的工厂对象(重点)
``` java
public class MyBeanFactory {
   public static StudentService createStudentService() {
        //创建目标对象target
        final StudentService studentService = new StudentService();
        //声明切面类对象
        final MyAspect aspect = new MyAspect();
        //创建增强对象
        Enhancer enhancer = new Enhancer();
        //设置父类（被代理的类）
        enhancer.setSuperclass(studentService.getClass());
        //设置回调（拦截）
        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
                /* System.out.println(proxy.getClass());
                 * proxy :class com.chen.service.StudentService$$EnhancerByCGLIB$$9502d5ce
                 * proxy 是StudentService的子类
                 */
                aspect.before();
                System.out.println(methodProxy);
                // 放行方法有两种方式
                //Object object = method.invoke(studentService, args);
                //解耦方式 没有引用外部的类
                Object object = methodProxy.invokeSuper(proxy, args);
                System.out.println("拦截方法执行了");
                aspect.after();
                return object;
            }
        });
        //创建代理对象
        StudentService serviceProxy = (StudentService) enhancer.create();
        return serviceProxy;
    }
}
```

### AOP联盟通知类型
AOP联盟为通知Advice定义了`org.aopalliance.aop.Advice`
Spring按照通知Advice在目标类方法的连接点位置，可以分为5类

通知类型|名称|作用
---|---|---
前置通知|org.springframework.aop.MethodBeforeAdvice|在目标方法执行前实施增强
后置通知|org.springframework.aop.AfterReturningAdvice|在目标方法执行后实施增强
环绕通知|org.aopalliance.intercept.MethodInterceptor|在目标方法执行前后实施增强
异常抛出通知|org.springframework.aop.ThrowsAdvice|在方法抛出异常后实施增强
引介通知|org.springframework.aop.IntroductionInterceptor|在目标类中添加一些新的方法和属性

### Spring AOP半自动
目标：掌握让Spring 创建代理对象，从spring容器中手动的获取代理对象。  
Spring核心4+1jar包、AOP联盟（规范）、spring-aop （实现）
1. 导入jar包   
``` java
spring-aop-3.2.0.RC2.jar    //AOP实现
//spring-framework-3.2.0.RC2\libs      
com.springsource.org.aopalliance-1.0.0.jar  //AOP联盟
//spring-framework-3.0.2.RELEASE-dependencies.zip\org.aopalliance\com.springsource.org.aopalliance
```

2. 创建service接口和实现类 
3. 创建切面类 实现 `MethodInterceptor` 接口
``` java
public class MyAspect implements MethodInterceptor {
    //拦截方法
    @Override
    public Object invoke(MethodInvocation methodInvocation) throws Throwable {
        System.out.println("开启事务");
        //执行放行
        methodInvocation.proceed();
        System.out.println("提交事务");
        return null;
    }
}
```
4. 配置Spring xml 
``` xml
    <!-- 配置UserService-->
    <bean id="userService" class="com.chen.service.UserServiceImpl"/>
    <!--配置切面类对象-->
    <bean id="myAspect" class="com.chen.service.MyAspect"/>
    <!--配置代理对象-->
    <bean id="serviceProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
        <!--接口 一个接口使用value 多个接口使用list-->
        <property name="interfaces" value="com.chen.service.IUserService" />
        <!--目标对象-->
        <property name="target" ref="userService"/>
        <!--切面类 此处不能使用ref -->
        <property name="interceptorNames" value="myAspect"/>
        <!--默认情况下Spring 生成的代理是JDK的Proxy实现的-->
        <!--配置使用cglib生成代理-->
        <property name="optimize" value="true"/>
    </bean>
```
5. 测试
``` java
    @Test
    public void Test13() {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans4.xml");
        IUserService userService = (IUserService) context.getBean("serviceProxy");
        userService.add();
    }
```

### Spring AOP全自动
明白什么是全自动织入
1. 导入jar包
``` java
com.springsource.org.aspectj.weaver-1.6.8.RELEASE.jar
//spring-framework-3.0.2.RELEASE-dependencies.zip\org.aspectj\com.springsource.org.aspectj.weaver\1.6.8.RELEASE
```
2. 增加aop xml约束 
![2.png]{2.png}

3. 编写xml 使用这种方式通过expression表达式可以使指定包下的所有对象都的多有方法都被拦截 执行aop切面操作
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!--xmlns xml namespace:xml命名空间-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd">
    <!--测试用的bean-->
    <bean id="userService" class="com.chen.service.UserServiceImpl"></bean>
    <bean id="studentService" class="com.chen.service.StudentService"></bean>

    <!--切面类-->
    <bean id="myAspect" class="com.chen.service.MyAspect"></bean>

    <!--全自动aop设置-->
    <!--1.在bean配置aop约束-->
    <!--2.配置aop:config内容,把切入点和通知结合-->
    <!--proxy-target-calss属性可以定义为cglib实现代理,不写默认是jdk实现代理-->
    <aop:config>
        <!--设置切入点
            使用expression表达式
        -->
        <aop:pointcut id="myPonitcut" expression="execution(* com.chen.service.*.*(..))" />
        <!--通知关联切入点-->
        <aop:advisor advice-ref="myAspect" pointcut-ref="myPonitcut"/>

    </aop:config>

</beans>
```
4. 测试类 （代码省略）

### AOP使用场景
AOP 多用于事务配置和日志记录

## AspectJ
### AspectJ 简介
AspectJ是一个基于Java语言的AOP框架  
Spring2.0以后新增了对AspectJ切点表达式支持  
@AspectJ 是AspectJ1.5新增功能，通过JDK5注解技术，允许直接在Bean类中定义切面  
**新版本Spring框架，建议使用AspectJ方式来开发AOP**  
主要用途：自定义开发  

### execution切入点表达式【掌握】
用于描述方法
语法：`execution(修饰符  返回值  包.类.方法名(参数) throws异常)`
表示内容|形式|作用
---|---|---
修饰符，一般省略|public|公共方法
修饰符，一般省略|*|任意
返回值，不能省略|void|返回没有值
返回值，不能省略|String|返回值字符串
返回值，不能省略|*|任意
包，省略|com.gyf.crm|固定包
包，省略|com.gyf.crm.*.service|crm包下面子包任意 （例如：com.gyf.crm.staff.service）
包，省略|com.gyf.crm..|crm包下面的所有子包（含自己）
包，省略|com.gyf.crm.*.service..|crm包下面任意子包，固定目录service，service目录任意包
类，省略|UserServiceImpl|指定类
类，省略|*Impl|以Impl结尾
类，省略|User*|以User开头
类，省略|*|任意
方法名，不能省略|addUser|固定方法
方法名，不能省略|add*|以add开头
方法名，不能省略|*Do|以Do结尾
方法名，不能省略|*|任意
(参数),不能省略|()|无参
(参数),不能省略|(int)|一个整型
(参数),不能省略|(int ,int)|两个
(参数),不能省略|(..)|参数任意
异常,省略|throws a,b,c |

案例1：`execution(* com.gyf.crm.*.service..*.*(..))`   

案例2(两个表达式 '或')：`<aop:pointcut expression="execution(* com.gyf.crm.service.*.*(..)) || 
                          execution(* com.gyf.*Do.*(..))" id="myPointCut"/>`   

**除execution（） 还有其他方式：**  
within:
匹配包或子包中的方法(了解)
within(com.gyf.aop..*)

this:
匹配实现接口的代理对象中的方法(了解)
this(com.gyf.aop.user.UserDAO)

target:
匹配实现接口的目标对象中的方法(了解)
target(com.gyf.aop.user.UserDAO)
args:
匹配参数格式符合标准的方法(了解)
args(int,int)
bean(id)  
对指定的bean所有的方法(了解)
bean('userServiceId')


### AspectJ 通知类型
aop联盟定义通知类型，具有特性接口，必须实现，从而确定方法名称。  
aspectj 通知类型，只定义类型名称，以及方法格式。  
6种，知道5种，掌握1中。  

通知名称|通知类型|描述|
---|---|---
before|前置通知(应用：各种校验)|在方法执行前执行，如果通知抛出异常，阻止方法运行
afterReturning|后置通知(应用：常规数据处理)|方法正常返回后执行，如果方法中抛出异常，通知无法执行必须在方法执行后才执行，所以可以获得方法的返回值。
around|环绕通知(应用：十分强大，可以做任何事情)|方法执行前后分别执行，可以阻止方法的执行**必须手动执行目标方法**
afterThrowing|抛出异常通知(应用：包装异常信息)|方法抛出异常后执行，如果方法没有抛出异常，无法执行
after|最终通知(应用：清理现场)|方法执行完毕后执行，无论方法中是否出现异常

需要添加到项目要的jar包(其他jar包请查看上面导入的jar包)
``` java 
spring-aspects-3.2.0.RC2.jar
//spring-framework-3.2.0.RC2-dist.zip\spring-framework-3.2.0.RC2\libs
```

#### 案例（使用xml配置）

1. 导入jar包
2. 创建service
``` java
public class UserServiceImpl implements IUserService{
    @Override
    public void addUser() {
        System.out.println("添加用户");
        //为测试异常通知
        //int a=10/0;
    }
}
```
3. 创建切面类
``` java
public class MyAspect {
    public void myBefore(JoinPoint joinPoint) {
        System.out.println("前置通知");

    }
    //后置通知获取业务方法执行后的返回值 可填加Object retValue传参
    public void myAfterRetruning(JoinPoint joinPoint,Object retValue) {
        System.out.println("后置通知");
        System.out.println("业务方法返回值"+retValue);
    }
    //joinPoint与ProceedingJoinPoint区别 ProceedingJoinPoint可以放行
    public Object myAround(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("环绕通知开启事务");
        //切入点就是个方法名
        System.out.println(joinPoint.getSignature().getName());
        //环绕通知需要放行
        Object retOjb=joinPoint.proceed();
        System.out.println("环绕通知提交事务");
        return retOjb;
    }

    public void myAfterThrowing(JoinPoint joinPoint,Throwable e){
        System.out.println("异常通知 异常信息:"+e.getMessage());
    }

    public void myAfter(JoinPoint joinPoint) throws Throwable {
        //无论有没有异常都会执行
        System.out.println("最终通知");
    }
}
```

4. 创建Spring Bean xml配置文件
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!--xmlns xml namespace:xml命名空间-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--配置UserService-->
    <bean id="userService" class="com.chen.web.service.UserServiceImpl"/>
    <!--配置切面对象-->
    <bean id="myAspect" class="com.chen.aspect.MyAspect"/>

    <!--配置aop-->
    <aop:config>
        <!--指定切面-->
        <aop:aspect ref="myAspect">
            <!--定义切入点-->
            <aop:pointcut id="myPontcut" expression="execution(* com.chen.web.service.*.*(..))" />
            <!--前置通知-->
            <aop:before method="myBefore" pointcut-ref="myPontcut"/>
            <!--后置通知-->
            <aop:after-returning method="myAfterRetruning" pointcut-ref="myPontcut" returning="retValue"/>
            <!--环绕通知-->
            <aop:around method="myAround" pointcut-ref="myPontcut"/>
            <!--异常通知-->
            <aop:after-throwing method="myAfterThrowing" pointcut-ref="myPontcut" throwing="e"/>
            <!--最终通知-->
            <aop:after method="myAfter" pointcut-ref="myPontcut"/>
        </aop:aspect>
    </aop:config>
</beans>
```
5. 测试类
``` java
public class test {
    @Test
    public void test1(){
       ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
       IUserService userService= (IUserService) context.getBean("userService");
       userService.addUser();
    }
}
```

### 案例（使用注解）
1. 导入jar包
2. 创建service 使用注解
``` java
@Service("userService")
public class UserServiceImpl implements IUserService{
    @Override
    public void addUser() {
        System.out.println("添加用户");
        //为测试异常通知
        //int a=10/0;
    }
}
```
3. 创建切面类 使用注解
``` java
@Component
@Aspect //告诉扫描器这个类是切入点类
public class MyAspect {
    //声明一个公共的切入点
    @Pointcut("execution(* com.chen.web.service.*.*(..))")
    public void myPointcut(){}

    //@Before("execution(* com.chen.web.service.*.*(..))")
    //使用声明公共切入点
    @Before("myPointcut()")
    public void myBefore(JoinPoint joinPoint) {
        System.out.println("前置通知");
    }
    @AfterReturning(pointcut = "execution(* com.chen.web.service.*.*(..))",returning = "retValue")
    public void myAfterRetruning(JoinPoint joinPoint,Object retValue) {
        System.out.println("后置通知");
        System.out.println("业务方法返回值"+retValue);
    }
    //joinPoint与ProceedingJoinPoint区别 ProceedingJoinPoint可以放行
    @Around("execution(* com.chen.web.service.*.*(..))")
    public Object myAround(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("环绕通知开启事务");
        //切入点就是个方法名
        System.out.println(joinPoint.getSignature().getName());
        //环绕通知需要放行
        Object retOjb=joinPoint.proceed();
        System.out.println("环绕通知提交事务");
        return retOjb;
    }
    @AfterThrowing(pointcut = "execution(* com.chen.web.service.*.*(..))",throwing = "e")
    public void myAfterThrowing(JoinPoint joinPoint,Throwable e){
        System.out.println("异常通知 异常信息:"+e.getMessage());
    }
   @After("execution(* com.chen.web.service.*.*(..))")
    public void myAfter(JoinPoint joinPoint) throws Throwable {
        //无论有没有异常都会执行
        System.out.println("最终通知");
    }
}
```

4. 编写 Spring xml 配置文件 配置注解扫描和aop注解生效
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!--xmlns xml namespace:xml命名空间-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <!--配置扫描注解的位置-->
    <context:component-scan base-package="com.chen"/>
    <!--配置aop注解生效-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```
5. 测试
``` java 
    @Test
    public void test2(){
       ApplicationContext context = new ClassPathXmlApplicationContext("beans2.xml");
       IUserService userService= (IUserService) context.getBean("userService");
       userService.addUser();
    }
```

6. 注解总结
@Aspect  声明切面，修饰切面类，从而获得通知。  
通知  
	@Before 前置  
	@AfterReturning 后置  
	@Around 环绕  
	@AfterThrowing 抛出异常  
	@After 最终  
切入点  
	@PointCut ，修饰方法 private void xxx(){}  之后通过“方法名”获得切入点引用  


## JdbcTemplate【了解】
### 1 简介
jdbcTemplate类似人DBUtils,用于操作Jdbc的工具类，它需要依赖于连接池DataSource(数据源)  
JDBC（Java DataBase Connectivity,java数据库连接）是一种用于执行SQL语句的Java API  
ODBC（Open Database Connectivity，ODBC）开放数据库连接,是微软公司开提供了一组对数据库访问的标准API（应用程序编程接口）  
DBCP（DataBase Connection Pool）数据库连接池，是java数据库连接池的一种，由Apache开发  
C3P0是一个开源的JDBC连接池，它实现了数据源和JNDI绑定，支持JDBC3规范和JDBC2的标准扩展。目前使用它的开源项目有Hibernate，Spring等  
**c3p0与dbcp区别**
dbcp没有自动回收空闲连接的功能  
c3p0有自动回收空闲连接功能  
 两者主要是对数据连接的处理不同c3p0提供最大空闲时间，dbcp提供最大连接数。前者是如果连接时间超过最大连接时间，就会断开当前连接。dbcp如果超过最大连接数，就会断开所有连接。

### 环境搭建
1. 创建数据库表
``` sql
create database spring_day02;
use spring_day02;
create table t_user(
  id int primary key auto_increment,
  username varchar(50),
  password varchar(32)
);
insert into t_user(username,password) values('jack','520');
insert into t_user(username,password) values('rose','521');
```
2. 导入jar包
``` java
com.springsource.com.mchange.v2.c3p0-0.9.1.2.jar   //c3p0链接池
//spring-framework-3.0.2.RELEASE-dependencies\com.mchange.c3p0\com.springsource.com.mchange.v2.c3p0\0.9.1.2
com.springsource.org.apache.commons.dbcp-1.2.2.osgi.jar   //dbcp连接池
//spring-framework-3.0.2.RELEASE-dependencies\org.apache.commons\com.springsource.org.apache.commons.dbcp\1.2.2.osgi
com.springsource.org.apache.commons.pool-1.5.3.jar  //dbcp依赖
//spring-framework-3.0.2.RELEASE-dependencies\org.apache.comm ons\com.springsource.org.apache.commons.pool\1.5.3
mysql-connector-java-5.0.8-bin.jar   
spring-jdbc-3.2.0.RC2.jar  //Spring JDBC
//spring-framework-3.2.0.RC2-dist.zip\spring-framework-3.2.0.RC2\libs
spring-tx-3.2.0.RC2.jar  //事务
//spring-framework-3.2.0.RC2-dist.zip\spring-framework-3.2.0.RC2\libs
//其他包参考上面spring依赖包
```

3. 创建JavaBean数据模型 代码省略

### JdbcTemplate 简单使用
``` java 
public class test2 {
    @Test
    public void test1() throws Exception{
        //创建数据源
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql:///spring_day02");
        dataSource.setUsername("root");
        dataSource.setPassword("123456");

        //创建JDBCTamplate
        JdbcTemplate jdbcTemplate= new JdbcTemplate(dataSource);
        int update = jdbcTemplate.update("update t_user set username=? where id=?","zzz",1);
    }
}
```

### 使用Spring xml配置数据源
1. 配置xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!--xmlns xml namespace:xml命名空间-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <!--配置扫描注解的位置-->
    <context:component-scan base-package="com.chen"/>
    <!--配置aop注解生效-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
    <!--配置dbcp数据源-->
    <bean id="dataSourceDbcp" class="org.apache.commons.dbcp.BasicDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://123.56.25.134:3306/spring_day02" />
        <property name="username" value="root" />
        <property name="password" value="123456" />
    </bean>
    <!--配置c3p0数据源-->
    <bean id="dateSourceC3p0" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver" />
        <property name="jdbcUrl" value="jdbc:mysql://123.56.25.134:3306/spring_day02" />
        <property name="user" value="root" />
        <property name="password" value="123456" />
    </bean>
    <!--配置jdbcTemplate-->
    <!--dao继承JdbcDaoSupport 不需要配置jdbcTemplate-->
    <!--<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">-->
        <!--<property ref="dateSourceC3p0" name="dataSource"/>-->
    <!--</bean>-->
    <!--配置dao-->
    <bean id="userDao" class="com.chen.dao.UserDaoImpl" >
        <!--<property name="jdbcTemplate" ref="jdbcTemplate"/>-->
        <property name="dataSource" ref="dataSourceDbcp"/>
    </bean>
</beans>
```

2. 编写dao用于测试
``` java
public class UserDaoImpl extends JdbcDaoSupport implements IUserDao{
    //使用JdbcDaoSupport模板获得JdbcTemplate 不需要自己创建
    //private JdbcTemplate jdbcTemplate;
    //public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
    //this.jdbcTemplate = jdbcTemplate;
    //}
    @Override
    public void insertUser(User user) {
        //int update = jdbcTemplate.update("insert into t_user(username,password) value (?,?)", user.getUsername(), user.getPassword());
        //使用模板 获得JdbcTemplate
        getJdbcTemplate().update("insert into t_user(username,password) value (?,?)", user.getUsername(), user.getPassword());
    }
}
```
3. 测试代码省略

### 使用properties存储数据库配置
``` xml
    <!--读取db.properties-->
    <context:property-placeholder location="classpath:db.properties"/>
    <!--配置dbcp数据源-->
    <bean id="dataSourceDbcp" class="org.apache.commons.dbcp.BasicDataSource">
        <property name="driverClassName" value="${driverClassName}" />
        <property name="url" value="${url}" />
        <property name="username" value="${uname}" />
        <property name="password" value="${password}" />
    </bean>
```




