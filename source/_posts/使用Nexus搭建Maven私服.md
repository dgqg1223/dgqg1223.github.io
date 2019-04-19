---
title: 使用Nexus搭建Maven私服
date: 2016-02-17 15:51:32
tags: Maven
categories: 
	- Java
	- 构建工具
---
## 安装Nexus
http://nexus.sonatype.rog/download
## 访问Nexus

访问URL: <http://localhost:8080/nexus-2.7.0-06/>

默认账号:

用户名： admin

密码： admin123


## Nexus的仓库和仓库组 
![](1.png)

**仓库有4种类型 :**

-   group(仓库组)：一组仓库的集合

-   hosted(宿主)：配置第三方仓库 （包括公司内部私服 ）

-   proxy(代理)：私服会对中央仓库进行代理，用户连接私服，私服自动去中央仓库下载jar包或者插件

-   virtual(虚拟)：兼容Maven1 版本的jar或者插件

**Nexus的仓库和仓库组介绍:**

-   3rd party:
    一个策略为Release的宿主类型仓库，用来部署无法从公共仓库获得的第三方发布版本构建

-   Apache Snapshots: 一个策略为Snapshot的代理仓库，用来代理Apache
    Maven仓库的快照版本构建

-   Central: 代理Maven中央仓库

-   Central M1 shadow: 代理Maven1 版本 中央仓库

-   Codehaus Snapshots: 一个策略为Snapshot的代理仓库，用来代理Codehaus
    Maven仓库的快照版本构件

-   Releases: 一个策略为Release的宿主类型仓库，用来部署组织内部的发布版本构件

-   Snapshots: 一个策略为Snapshot的宿主类型仓库，用来部署组织内部的快照版本构件

-   **Public
    Repositories:该仓库组将上述所有策略为Release的仓库聚合并通过一致的地址提供服务**

![](2.png)

## 配置所有构建均从私服下载

在本地仓库的setting.xml中配置如下：

``` xml
<mirrors>
	 <mirror>
		 <!--此处配置所有的构建均从私有仓库中下载 *代表所有，也可以写central -->
		 <id>nexus</id>
		 <mirrorOf>*</mirrorOf>
		 <url>http://localhost:8080/nexus-2.7.0-06/content/groups/public/</url>
	 </mirror>
 </mirrors>
```


![](3.png)

## 部署构建到Nexus

### 第一步：Nexus的访问权限控制

在本地仓库的setting.xml中配置如下：

``` xml
<server>
 		<id>releases</id>
		<username>admin</username>
		<password>admin123</password>
	</server>
	<server>
		<id>snapshots</id>
		<username>admin</username>
		<password>admin123</password>
	</server>
```


### 第二步：配置pom文件
在需要构建的项目中修改pom文件
``` xml
<distributionManagement>
		<repository>
			<id>releases</id>
			<name>Internal Releases</name>
			<url>http://localhost:8080/nexus-2.7.0-06/content/repositories/releases/</url>
		</repository>
		<snapshotRepository>
			<id>snapshots</id>
			<name>Internal Snapshots</name>
			<url>http://localhost:8080/nexus-2.7.0-06/content/repositories/snapshots/</url>
		</snapshotRepository>
	</distributionManagement>
```


### 第三步：执行maven的deploy命令

![](4.png)
