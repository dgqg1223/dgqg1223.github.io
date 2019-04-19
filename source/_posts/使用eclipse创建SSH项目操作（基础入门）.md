---
title: 使用eclipse创建SSH项目操作（基础入门）
date: 2016-02-17 15:51:32
tags: SSH框架
categories: 
	- Java
	- 框架
---
### 一、创建Dynamic Web Project项目
1. eclipse 新建Dynamic Web Project项目。  
![](1.png)

2. 创建项目过程中Dynamic web module version 版本选择2.5。 （3.0协议没有web.xml）
![](2.png)

3. 创建项目后修改jre版本。（默认1.5版本太低）
选择项目右键菜单中选择build path >> confgure build path... 
![](3.png) 
弹出窗口中找到Libraries选项卡删除jre1.5，点击 Add Library... >> JRE System Library >> Next >> 选择需要更改的jre版本
![](4.png) 

### 二、导入相关jar包（ssh）
jar包复制到项目WebContent\WEB-INF\lib目录下
1. struts2相关jar包
	struts-2.3.15.3\apps\struts2-blank\WEB-INF\lib 下的jar包共11个
	struts-2.3.15.3\libstruts2-spring-plugin-2.3.15.3.jar  struts2和spring整合包1个
	总共导入12个jar包 
![](5.png)   
![](6.png)   

2. spring核心包   
	spring-framework-3.2.0.RELEASE\libs 路径下
	```
	spring-aop-3.2.0.RELEASE.jar
	spring-aspects-3.2.0.RELEASE.jar
	spring-beans-3.2.0.RELEASE.jar
	spring-context-3.2.0.RELEASE.jar
	spring-core-3.2.0.RELEASE.jar
	spring-expression-3.2.0.RELEASE.jar
	spring-jdbc-3.2.0.RELEASE.jar
	spring-orm-3.2.0.RELEASE.jar
	spring-tx-3.2.0.RELEASE.jar
	spring-web-3.2.0.RELEASE.jar
	```
3. spting 依赖包  
	```
	spring-framework-3.0.2.RELEASE-dependencies 3.2版本没有所以使用3.02版本包 中的
	spring-framework-3.0.2.RELEASE-dependencies\com.mchange.c3p0\com.springsource.com.mchange.v2.c3p0\0.9.1.2\com.springsource.com.mchange.v2.c3p0-0.9.1.2.jar
	spring-framework-3.0.2.RELEASE-dependencies\org.aopalliance\com.springsource.org.aopalliance\1.0.0\com.springsource.org.aopalliance-1.0.0.jar
	spring-framework-3.0.2.RELEASE-dependencies\org.apache.log4j\com.springsource.org.apache.log4j\1.2.15\com.springsource.org.apache.log4j-1.2.15.jar
	spring-framework-3.0.2.RELEASE-dependencies\org.apache.commons\com.springsource.org.apache.commons.logging\1.1.1\com.springsource.org.apache.commons.logging-1.1.1.jar
	spring-framework-3.0.2.RELEASE-dependencies\org.aspectj\com.springsource.org.aspectj.weaver\1.6.8.RELEASE\com.springsource.org.aspectj.weaver-1.6.8.RELEASE.jar
	```

4. hibernate 相关jar包
	```
	#导入hibernate核心jar包（根目录下）
	hibernate3.jar

	#导入hibernate必须的jar包 
	hibernate-distribution-3.6.10.Final\lib\required\antlr-2.7.6.jar
	commons-collections-3.1.jar
	dom4j-1.6.1.jar
	javassist-3.12.0.GA.jar
	jta-1.1.jar
	slf4j-api-1.6.1.jar

	#导入hibernate可选的jar包 
	#struts已导入，无需导入
	hibernate-distribution-3.6.10.Final\lib\optionalc3p0-0.9.1.jar		

	#导入jpa包
	ssh包\hibernate-distribution-3.6.10.Final\lib\jpa\hibernate-jpa-2.0-api-1.0.1.Final.jar

	#导入cglib和javassist包 
	#该包含在spring-core-3.2.0.RELEASE.jar包中所以无需导入
	hibernate-distribution-3.6.10.Final\lib\bytecode\cglib-2.2.jar		
	#导入ss jar包已导入无需再导入
	hibernate-distribution-3.6.10.Final\lib\bytecode\javassist-3.12.0.GA.jar 
	```
5. 导入slf4j日志包（hibernate依赖的日志包）
	获取方法：https://www.slf4j.org/
	```
	slf4j-log4j12-1.7.25.jar
	```
6. 导入数据库驱动包
	获取方法Mysql网站获取：https://downloads.mysql.com/archives/c-j/
	```
	mysql-connector-java-5.1.5-bin.jar
	```

### 三、整理jar包
1. 删除自动生成的Web App Libraries库，方便操作删除没用的jar包
项目下右键选择build path >> confgure build path... >>libraries选项卡 选择Web App Libraries >> remove >> Apply and Close
2. 删除web-inf下lib中重复的jar包（一般保留高版本）
3. 构建jar包 
选中web-inf下lib中的所有jar包右键 Build Path >> Add to Build Path 
所有jar包会构建到库下Java Resources.Libraries.Referenced Libraries
4. 整理后的jar包
![](9.jpg)   


