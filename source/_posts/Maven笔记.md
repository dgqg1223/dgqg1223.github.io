---
title: Maven笔记
date: 2019-02-17 15:51:32
tags: Maven
categories: 
	- Java
	- 构建工具
---

# Maven 
自己理解：统一管理jar包
Maven是基于POM（工程对象模型），通过一小段描述来对项目的代码、报告、文件进管理的工具。
Maven是一个跨平台的项目管理工具，它是使用java开发的，它要依赖于jdk1.6及以上
Maven主要有两大功能：管理依赖、项目构建。
（依赖指的就是jar包）

项目构建方式（清理、编译、单元测试、报告、打包、部署）
Eclipse	使用eclipse构建项目，相对来说，步骤零散，不好操作
Ant	专门项目构建工具。需要详细配置高诉Ant源码包、class文件、资源文件存放位置，需要了解相关Ant语法
Maven	项目管理工具，也是项目构建工具，可以对项目进行简单快速构建。需要安装maven，按照maven规范进行代码开发（有约束）

官方网站：http://maven.apache.org 本课程使用的maven的版本为3.0.5


## 开发中遇到的问题 
1. 都是同样的代码，为什么在我的机器上可以编译执行，而在他的机器上就不行？
2. 为什么在我的机器上可以正常打包，而配置管理员却打不出来?
3. 项目组加入了新的人员，我要给他说明编译环境如何设置，但是让我挠头的是，有些细节我也记不清楚了。
4. 我的项目依赖一些jar包，我应该把他们放哪里？放源码库里？
5. 这是我开发的第二个项目，还是需要上面的那些jar包，再把它们复制到我当前项目的svn库里吧
6. 现在是第三次，再复制一次吧    ----- 这样真的好吗？
7. 我写了一个数据库相关的通用类，并且推荐给了其他项目组，现在已经有五个项目组在使用它了，今天我发现了一个bug，并修正了它，我会把jar包通过邮件发给其他项目组
-----这不是一个好的分发机制，太多的环节可能导致出现bug
8. 项目进入测试阶段，每天都要向测试服务器部署一版。每次都手动部署，太麻烦了。


## 原版maven使用步骤
1.下载 官方网站：http://maven.apache.org  
2.解压添加环境变量：MAVEN_HOME,PATH(MAVEN_HOME/bin)  
3.测试安装是否成功：mvn –v  
4.Maven常用命令：
```
	compile		编译
	clean		清除target目录
	test		test目录中的源码进行编译
	package		打包命令
	install		安装命令，会将打好的包，安装到本地仓库
	clean compile	组合命令先清除再编译
	clean test		同上先清除再执行test，通常应用于测试环节
	clean package 	清理>编译>测试>打包，通常应用于发布前 
```
Maven 安装目录有两个配置文件,全局配置目录和用户配置目录 用户配置需要自己复制setting.xml 到~/.m2目录下 (“~”指用户目录) 
本地仓库目录地址: Maven目录下 `/conf/setting.xml
  | Default: ~/.m2/repository`
  <localRepository>/path/to/local/repo</localRepository>



## Maven目录结构
```
Project
	|--src（源码包）
		|--main（正常的源码包）
			|--java（.java文件的目录）
			|--resources（资源文件的目录）
		|--test（测试的源码包）
|--java
			|--resources
	|--target（class文件、报告等信息存储的地方）
	|--pom.xml（maven工程的描述文件）
```


## 安装M2Eclipse插件（高版本eclipse、mclipse自带）
1. 解压后拷贝到eclipse中的dropins目录
2. 查看eclipse中是否安装成功，eclipse>windows>Preferences>左边菜单中出现Maven
3. 设置Maven的安装路径，eclipse>windows>Preferences>Maven>Installations>add（添加目录后勾选）
4. 上步骤中Maven>User Setting可设置用户Maven配置文件和本地配置文件（使用eclipse可以指定maven配置文件地址）
5. 卸载Maven删除dropins目录之前拷贝的文件即可


## Maven的核心概念 - 坐标与依赖管理
### Maven坐标主要组成
```
<dependencies>
	<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
		<version>4.9</version>
		<scope>test</scope>
		<optional>false</scope>
		<exclusion>
			<groupId>com.itheima.maven</groupId>
			<artifactId>MavenFirst</artifactId>
		<exclusion>
	</dependency>
</dependencies>
```
groupld：定义当前Maven组织名称  
artifactld：定义项目实际名称  
version：定义目的当前项版本  

### Maven依赖
依赖范围
prm.xml中 <scope>标签用来定义依赖范围  
```
comiple 对于主代码、测试代码、打包后运行时有效 例如 log4j  
test 仅对于测试代码时有效 例如 junit  
provided 仅对于主代码、测试代码有效 例如 servlet-api  
runtime 仅对于打包后运行时有效 例如 JDBC Driver Implementation  
```

### 依赖传递？？？
工程A依赖工程B 工程B依赖工程C  工程A直接依赖工程B间接依赖工程C

### 依赖冲突
多个项目中（跨pom文件） 每个pom.xml 中相具有同包的坐标 但<version>版本不同 使用直接依赖 就近原则
单个项目（同一个pom文件） pom.xml 中出现两个以上相同包的坐标但<version>版本不同 使用pom.xml文档中最下面坐标的jar包（从上往下解析）
可选依赖
<Optional>标签标识该依赖是否可选，默认是false，false会传递下去 true不传递。

### 依赖排除
```
<exclusion>
	<groupId>com.itheima.maven</groupId>
	<artifactId>MavenFirst</artifactId>
<exclusion>
```
该方法是在依赖项目中排除 实际工作中使用该方法 <Optional>是被依赖的pom中定义

## Maven生命周期 
maven每个插件（命令）是有相对应的之前步骤，这个步骤我们叫生命周期 生命周期有很多大概记住下面的这些
pre-site 执行一些需要在生成站点文档之前完成的工作 
site 生成项目的站点文档 
post-site 执行一些需要在生成站点文档之后完成的工作，并且为部署做准备 
site-deploy 将生成的站点文档部署到特定的服务器上 

## 插件（plugin）
插件，每个插件都能实现一个阶段的功能。Maven的核心是生命周期，但是生命周期相当于主要指定了maven命令执行的流程顺序，而没有真正实现流程的功能，功能是有插件来实现的。
比如：compile就是一个插件实现的功能。(其实compile是个插件)

编译插件
```
<build>
	<plugins>
		<!-- 编译插件，指定编译用的jdk版本 -->
		<plugin>
			<groupId>org.apache.maven.plugins</groupld>
			<artifactId>maven-comp-pllugin</artifactId>
			<configuration>
				<source>1.7</source>
				<target>1.7</target>
				<encoding>utf-8</encoding>
			</configuration>
		</plugin>
	</plugins>
	<plugins>
		<!-- maven工程可以不安装tomcat，使用tomcat插件即可,tomcat:run 插件命令默认为tomcat6-->
		<!-- 若使用tomcat7来运行web工程，需要在pom中指定tomcat7 它的执行命令是：tomcat7:run -->
		<plugin>
			<groupId>org.apache.tomcat.maven</groupld>
			<artifactId>tomcat7-maven-plugin</artifactId>
			<configuration>
				<pornt>80</pornt>
				<pornt>/</pornt>
			</configuration>
		</plugin>
	</plugins>
<build>
```


## 7.第一个Maven工程
创建web工程时候在创建MavenProject时候packging要选择war方式  
工程创建好时需要自己创建创建WEB-INF及web.xml文件  
创建index.jsp文件使用tomcat:run命令测试即可  


## 8.Maven继承
maven中的继承指的是pom文件的继承   
创建父类工程在创建工程步骤中packging必须选择pom方式  
创建子类工程可以在创建图形界面中选择父类pom工程  
创建子类工程也可以在pom.xml文件中指定父类工程pom文件  
```
<parent>
	<artifactId>父类工程名</artifactId>
	<groupId>com.itheima.maven</groupId>
	<version>版本信息</version>
</parent>
```

### 9.统一依赖jar包 
是指父工程统一定义一些jar包子工程来选择使用 不这么做子工程会继承所有父工程指定的所有jar包（不需要的也会继承）防止冗余


## 10.统一管理版本号（父工程抽取版本号）
作用方便更改版本号，防止同一jar包定义多个版本号

## 11.聚合
在真实项目中  创建多个maven工程 发布时候合并为一个


## 12.maven 仓库类型
仓库的分类：
1.本地仓库默认在~/.2/repository中，以配置文件为准
2.中央仓库（开源jar包，不包含有版权的依赖包） http://repo1.maven.org/moven2
   私服（自建远程仓库） 访问私服 局域网通过私服下载 私服没有私服访问中央仓库下载，另外私服存储本地开发的jar包

### 12.1.私服安装
为所有来自中央仓库的构建安装提供本地缓存。  
下载网站：http://nexus.sonatype.org/  
安装版本：nexus-2.7.0-06.war  
第一步：安装tomcat  
第二步：将nexus的war包拷贝到tomcat的webapps下  
第三步：启动tomcat  

访问URL: http://localhost:8080/nexus-2.7.0-06/
默认账号:
用户名： admin
密码： admin123

具体可参考web私服搭建文章

