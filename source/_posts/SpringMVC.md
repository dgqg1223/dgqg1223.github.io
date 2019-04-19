---
title: SpringMVC
date: 2019-03-14 10:01:14
tags: SpringMVC
categories: 
    - Java
    - 框架
---
## SpringMVC简介
MVC  
M:Model  
V:View  
C:Controller - servlet/action/controller（SpringMvc）

Spring MVC是Spring提供的一个强大而灵活的web框架。借助于注解，Spring MVC提供了几乎是POJO的开发模式，使得控制器的开发和测试更加简单。这些控制器一般不直接处理请求，而是将其委托给Spring上下文中的其他bean，**通过Spring的依赖注入功能，这些bean被注入到控制器中**。  
Spring MVC主要由DispatcherServlet、处理器映射【找控制器】、适配器【调用控制器的方法】、控制器【业务】、视图解析器、视图组成

## SpringMVC原理

## SpringMVC入门案例
环境：Spring3.2(支持及jdk1.7，否则编译会报错) tincat8 jdk1.7   

1. 创建web项目
2. 在WEB-INF下创建lib文件夹 导入jar包 新增`spring-webmvc-3.2.0.RELEASE.jar`包
3. 修改web.xml配置DispatcherServlet
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--配置所有请求由Springmvc管理-->
    <servlet>
        <servlet-name>DispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>DispatcherServlet</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>

</web-app>
```
4. 在WEB-INF目录下创建DispatcherServlet-servlet.xml 配置springmvc 所需要的组件
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
		http://www.springframework.org/schema/mvc
		http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context-3.2.xsd
		http://www.springframework.org/schema/aop
		http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
		http://www.springframework.org/schema/tx
		http://www.springframework.org/schema/tx/spring-tx-3.2.xsd">

    <!--1.配置url处理映射-->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

    <!--2.配置控制器处理适配器-->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>

    <!--3.配置控制器-相当于配置了访问路径-->
    <bean name="/user.do" class="com.chen.backoffice.web.controller.UserController"/>

    <!--4.配置资源视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--前缀-->
        <property name="prefix" value="/WEB-INF/views/"></property>
        <!--后缀-->
        <property name="suffix" value=".jsp"></property>
    </bean>
</beans>
```
5. 创建一个Controller控制器(DispatcherServlet-servlet.xml控制器配置中与之对应) 
控制器相当于需要实现`Controller`接口
``` java
public class UserController implements Controller {

    @Override
    public ModelAndView handleRequest(javax.servlet.http.HttpServletRequest httpServletRequest, javax.servlet.http.HttpServletResponse httpServletResponse) throws Exception {
        ModelAndView mav = new ModelAndView("user/userlist");

        mav.addObject("name","chen");

        return mav;
    }
}
```
6. /WEB-INF/views/user下创建userList.jsp页面（DispatcherServlet-servlet.xml路径配置中与之对应）
``` html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>用户列表</title>
</head>
<body>
用户列表（使用EL表达式取出的值）：${name}
</body>
</html>
```
7. 运行web项目 进入userList.jsp 测试

## Springmvc执行流程
![1.png](1.png)

## URL处理映射HandlerMapping 【了解】
URL处理映射的四种方式：
1. `BeanNameUrlHandlerMapping` 通过URL信息（名字）找到对应bean的name值，从而找到的控制器
2. `SimpleUrlHandlerMapping` 通过key找到对应bean的id值，从而找到的控制器
3. `ControllerClassNameHandlerMapping`不配置访问路径，默认的访问路径就是类名，首字母变为小写 .do结尾 例如：UserController.java >> userController.do

``` xml
    <!--方式一 BeanNameUrlHandlerMapping-->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
    <!--方式二 SimpleUrlHandlerMapping-->
    <bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
        <property name="mappings">
            <props>
                <prop key="user1.do">userController</prop>
                <prop key="user2.do">userController</prop>
                <prop key="user3.do">userController</prop>
            </props>
        </property>
    </bean>
    <!--方式三 ControllerClassNameHandlerMapping-->
    <bean class="org.springframework.web.servlet.mvc.support.ControllerClassNameHandlerMapping"/>
```

## 处理适配器HandlerAdapter【了解】
HandlerAdapter有两种方式：
1. `SimpleControllerHandlerAdapter` 使用该处理适配器的控制器（Controller）需要实现`Controller`接口，该处理适配器可以使用控制器`ModelAndView`对象  
使用 `SimpleControllerHandlerAdapter`的控制器写法
``` java
public class UserController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse Response) throws Exception {
        ModelAndView mav = new ModelAndView("user/userlist");
        mav.addObject("message","使用ModelAndView传入数据");
        return mav;
    }
}
```
2. `HttpRequestHandlerAdapter` 使用该处理适配器的控制器（Controller）需要实现`HttpRequestHandler`接口，该处理适配器不能使用控制器`ModelAndView`对象  
使用 `HttpRequestHandlerAdapter`的控制器写法
``` java 
public class UserController2 implements HttpRequestHandler {
    @Override
    public void handleRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        request.setAttribute("message","使用原始方法传入数据");
        request.getRequestDispatcher("/WEB-INF/views/user/userlist.jsp").forward(request,response);
    }
}
```


## 命令控制器【了解】
创建控制器（Controller），继承`AbstractCommandController`抽象类。创建空参构造方法，构造方法中使用`setCommandClass(model.class)`告诉Spring接收表单存放数据的模型类型,在重写的`handle` 方法参数中可获得`Object o`实体对象
``` java 
public class CommandController extends AbstractCommandController {
    public CommandController(){
        //告诉Springmvc把表单数据存在User模型中
        this.setCommandClass(User.class);
    }
    @Override
    protected ModelAndView handle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, BindException e) throws Exception {
        User user = (User)o;
        System.out.println(o);
        ModelAndView mav = new ModelAndView();
        mav.setViewName("user/info");
        mav.addObject("user",o);
        return mav;
    }
}
```
## 解决中文乱码
1. pos请求中文乱码 在web.xml中配置过滤器 
``` xml
<!-- 配置编码过滤器  -->
  	<filter>
  		<filter-name>EncodingFilter</filter-name>
  		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
  		<init-param>
  			<param-name>encoding</param-name>
  			<param-value>UTF-8</param-value>
  		</init-param>
  	</filter>
	
	<filter-mapping>
		<filter-name>EncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
```
2. get请求乱码
Tomcat8 默认进行了url编码，解决了get请求乱码问题
Tomcat7:   
修改apache-tomcat-7.0.52\conf\server.xml     
``` xml
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"  URIEncoding="UTF-8"/>`
```
修改apache-tomcat-7.0.52\bin\catalina.bat在最后一行rem注释 后面加入下面代码
``` bat
rem ---------------------------------------------------------------------------
set JAVA_OPTS=-Xms512m -Xmx1024m -XX:MaxPermSize=1024m -Dfile.encoding=UTF-8
```
>>>>>>> a714ff44305259af199cbd02bdd1333ca5e7819b

## SpringMVC 注解开发【掌握】
1. WEB-INF下创建DispatcherServlet-servlet.xml SpringMVC配置文件
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
		http://www.springframework.org/schema/mvc
		http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context-3.2.xsd
		http://www.springframework.org/schema/aop
		http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
		http://www.springframework.org/schema/tx
		http://www.springframework.org/schema/tx/spring-tx-3.2.xsd">

    <!--1.配置注解扫描器位置-->
    <context:component-scan base-package="com.chen.backoffice.web.controller"/>
    <!--2.配置处理器映射，通过注解来查找-->
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>
    <!--3.配置注解处理适配器来执行控制器的方法-->
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>

    <!--配置Springmvc 视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```
2. 创建Controller 使用注解开发
``` java
@Controller
@RequestMapping("/user")  //该注解写在类上面，表示自定义根路径，定义后需要访问/user/list.do
public class UserController{

    @RequestMapping("/list")
    public String list(){

        return "user/userlist";
    }
}
```
### RequestMapping讲解
``` java
@RequestMapping(“list”) 
@RequestMapping(“/list.do”) //与(“list”)效果相同
@RequestMapping(value=”/list.do”)  //与(“list”)效果相同
@RequestMapping(value = "/list3",method=RequestMethod.POST) //只能使用POST方法
@RequestMapping(value = "/list3",method=RequestMethod.Get)  //只能使用GET方法 

```


### 接收请求参数-三种方式
封装参数：基本类型，和Pojo类（model）  

Struts2参数与SpringMVC参数对比  
Struts2参数:基于类属性封装，在action中会添加属性，提供set方法。
SpringMVC参数：基于方法进行封装。
1. Controller
``` java
@Controller
@RequestMapping("/user")
public class UserController{
    //表单接收方式1：接收基本类型
    @RequestMapping("/register")
    public String register(String username, String password, int age, String gender,
                           Date birthday,String[] hobbyIds){
        System.out.println(username);
        System.out.println(password);
        System.out.println(age);
        System.out.println(gender);
        System.out.println(birthday);
        System.out.println(Arrays.toString(hobbyIds));

        return "user/info";
    }
    //表单接收方式2：接收POJO类型(模型)
    @RequestMapping("/register2")
    public String register2(User user){
        System.out.println(user);
        return "user/info";
    }
    //表单接收方式3：接收包装类型参数 UserExt里面有一个User对象
    @RequestMapping("/register3")
    public String register3(UserExt user){
        System.out.println(user);
        return "user/info";
    }
}
```
2.jsp 页面
``` jsp
<form action="${pageContext.request.contextPath}/user/register2.do" method="post">
    用户名:<input type="text" name="username" /><br>
    密码：<input type="text" name="passowrd" /><br>
    性别：<input type="text" name="gender" /><br>
    年龄：<input type="text" name="age" /><br>
    生日：<input type="text" name="birthday" /><br>
    爱好：<input type="checkbox" name="hobbyIds" value="1">唱歌
    <input type="checkbox" name="hobbyIds" value="2">画画
    <input type="checkbox" name="hobbyIds" value="3">打游戏<br>
    <input type="submit"/>
</form>
<form action="${pageContext.request.contextPath}/user/register3.do" method="post">
    用户名:<input type="text" name="user.username" /><br>
    密码：<input type="text" name="user.passowrd" /><br>
    性别：<input type="text" name="user.gender" /><br>
    年龄：<input type="text" name="user.age" /><br>
    生日：<input type="text" name="user.birthday" /><br>
    爱好：<input type="checkbox" name="user.hobbyIds" value="1">唱歌
    <input type="checkbox" name="user.hobbyIds" value="2">画画
    <input type="checkbox" name="user.hobbyIds" value="3">打游戏<br>
    <input type="submit"/>
</form>
```
### 接收请求参数-集合、Map
jsp 
``` jsp
<br>表单使用List接收<br>
<form action="${pageContext.request.contextPath}/user/register4.do" method="post">
    用户名:<input type="text" name="userList[0].username" /><br>
    密码：<input type="text" name="userList[0].passowrd" /><br>
    用户名:<input type="text" name="userList[1].username" /><br>
    密码：<input type="text" name="userList[1].passowrd" /><br>
    <input type="submit"/>
</form>
<br>表单使用Map接收<br>
<form action="${pageContext.request.contextPath}/user/register5.do" method="post">
    用户名:<input type="text" name="infos['username']" /><br>
    密码：<input type="text" name="infos['passowrd']" /><br>
    性别：<input type="text" name="infos['gender']" /><br>
    年龄：<input type="text" name="infos['age']" /><br>
    <input type="submit"/>
</form>
```


### SpringMVC与Struts区别【面试题】
1. 实现机制：Struts2 基于过滤器实现，SpringMVC基于Servlet实现
2. 运行速度：   
**Struts2是多例**   
每一次请求，都会创建一个Action对象   
当请求发生时，struts2创建对象：ActionContext，valuestack，UAction，ActionSuport，ModelDriven   
**Springmvc是单例**   
同一个Controller请求，只会创建一个Controller
3. 参数封装来   
Struts基于属性进行封装，Action事有参数属性
SpringMVC基于方法封装，参数接收的定义在Controller的方法中。


### JSP页面回显
Controller
``` java
    //jsp回显数据
    @RequestMapping("/list1")
    public String list(Model model) {
        //假设从数据库取出的数据
        List<User> userList = new ArrayList<User>();
        User user1 = new User("彭摆鱼", 18, "男", new Date());
        User user2 = new User("刘访华", 20, "男", new Date());
        User user3 = new User("范冰冰", 18, "女", new Date());
        user1.setId(1);
        user2.setId(2);
        user3.setId(3);
        userList.add(user1);
        userList.add(user2);
        userList.add(user3);

        model.addAttribute("userlist", userList);
        return "user/userlist";
    }

    //修改数据
    @RequestMapping("/edit")
    public String edit(int id,Model model) {
        System.out.println("获取要修改的"+id);
        return "user/userlist";
    }
```
userlist.jsp 使用jstl需要导入入jstl jar包
``` HTML
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%--引入jstl包--%>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>用户列表</title>
</head>
<body>
用户列表：
<table border="1">
    <tr>
        <td>姓名</td>
        <td>年龄</td>
        <td>性别</td>
        <td>生日</td>
    </tr>

    <c:forEach items="${userlist}" var="user">
        <tr>
            <td>${user.username}</td>
            <td>${user.age}</td>
            <td>${user.gender}</td>
            <td>${user.birthday}</td>
            <td><a href="${pageContext.request.contextPath}/user/edit.do?id=${user.id}">修改</a></td>
        </tr>
    </c:forEach>
</table>

</body>
</html>
```

### Url映射模板
**url模板可以restfull软件架构**
1. jsp
``` html
    <!-- 正常传参 -->
     <td><a href="${pageContext.request.contextPath}/user/edit.do?id=${user.id}">修改</a></td>
    <!-- 使用url映射模板 -->
     <td><a href="${pageContext.request.contextPath}/user/edit/${user.id}.do">修改</a></td>
```
2. controller
``` java
    // 使用url映射模板
    @RequestMapping("/edit/{id}")
    //{}的值会注入 标记@PathVariable 的参数中
    public String edit(@PathVariable int id, Model model) {
        System.out.println("获取要修改的"+id);
        return "user/userlist";
    }
```
使用rest风格提交可以在web.xml中设置`/rest/*` 的拦截方式
``` xml
    <servlet-mapping>
        <servlet-name>DispatcherServlet</servlet-name>
        <url-pattern>/rest/*</url-pattern>
    </servlet-mapping>
    <!-- 配置后可以使用${pageContext.request.contextPath}/rest/user/edit/${user.id} -->
```

### 转发和重定向
``` java
    @RequestMapping("/toRegister")
    public String toRegister() {
        // 同控制器转发
        return "forword:register.do";
        // 不同控制器转发
        return "forword:/user/register.do";
        // 同控制器转发
        return "redirect:register.do";
        // 同控制器重定向
        return "redirect:/user/register.do"
    }
```


### RequestParam
在Controller定义的方法中，参数可以使用RequestParam，来定义参数默认值，参数名称，和是否为空等信息。
``` java
    @RequestMapping("/Register")
    public String Register1(@RequestParam(value = "uid",required = true ,defaultValue = "19") int id) {
        return "/user/info";
    }
```

## ResponseBody和RequestBody
### 简介
@ResponseBody**把后台pojo转换json对象**，返回到页面      
@RequestBody接受前台json数据，**把json数据自动封装javaBean**

### 案例
请求和响应都是json数据
1. 导入json jar包




## 配置多视图




---
contriller
adapter