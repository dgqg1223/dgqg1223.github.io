---
title: WebService笔记
date: 2019-02-25 14:12:14
tags: WebService
categories: 
	- Java
	- 框架
---
## Webservice 
远程调用技术-专门解决异构系统调用
系统a调用系统b 业务层调用业务层

### Webservice的原理
Webservice是使用http发SOAP协议数据的一种远程调用技术

UDDI：提供服务端的注册和搜索功能。
WSDL：Webservice服务端的一个使用说明书。描述接口、方法、参数、返回值等信息。WSDL是随服务发布成功自动生成无需编写。 
SOAP：即简单对象访问协议。他是使用http发送的XML格式数据，它可以跨平台，跨防火墙，SOAP不是Webservice的专有协议，SOAP=http+xml。
	  自己理解：是一种网络协议，定义Webservice使用什么传输方式协议进行通讯（http）和如何解析（xml） http+xml。解决了跨平台应用之间的通信 

### Webservice优缺点
	 发送方式采用http的post，http默认端口是90，所以跨越防火墙
	 数据封装使用XML格式，所以webservice可以跨平台
	 Webservice支持面向对象开发（XML面向对象）

### 应用场景
	软件集成和复用
	适用场景：发布服务（对内对外），不考虑性能，不考虑客户端类型，此时建议使用webservice

### 需要记住的知识点
	服务端实现类注释： @WebService
	java使用SOAP1.2服务端实现类注释： @BindingType(SOAPBinding.SOAP12HTTP_BINDING)
	java使用SOAP1.2需要导入插件（相关jar包）（不常用）
	发布服务Endpoint.publish("访问地址",实现类)：Endpoint.publish("http://127.0.0.1:12345/weather", new WeatherInterfaceImpl());
	WSDL地址：服务地址+？wsdl
	WSDL阅读方式，从下往上，service->binding->portType->其中有接口、方法、参数和返回值
	生成客户端代码： wsimport -s . http://127.0.0.1:12345/weather
	根据使用wsdl调用服务端 创建服务视图service的name属性获取 获取服务实现类从portType的name属性获取 调用查询方法从portType下的operation标间的name属性获取
	SOAP协议格式：必须有envelope和body 非必有header和fault

### SOAP1.1和1.2区别
	相同点：
		都使用http的POST发送请求
		协议格式都相同：都有envelope标签和body标签
	不同点
		SOAP1.1：content-type：text/xml;charset=utf-8
		SOAP1.2：content-type：application/soap+xml;charset=utf-8
		命名空间不同
		
http://www.webxml.com.cn/zh_cn/index.aspx
		
## CXF Webservice开源框架

CXF Webservice开源框架 以后补充