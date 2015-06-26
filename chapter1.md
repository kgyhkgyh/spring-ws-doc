# 1. 什么是Spring web service


## 1.1  简介


Spring Web Service (Spring-WS)是一款Spring社区的产品，其聚焦于创建文档驱动式(document driven)的Web Service。Spring-WS致力于快速容易的开发契约优先(contract first)的SOAP服务，并使用多种操作xml有效负荷的方式来创建灵活的web service。这款产品是基于Spring自身，这意味着我们可以使用依赖注入等Spring的概念来整合web service服务。

人们有很多理由去使用Spring-WS，但是最具有说服力的是它可代替堆栈缺乏的SOAP成为web service的最佳实践。Spring-WS 使得web service好用却十分容易上手。Spring-WS包含了WS-I的基本描述，契约优先的开发模式，在契约和实现之间的松耦合实现等。其他的一些特性包括：

**强大的映射能力**：你可以分派来源xml到任何对象上，XPath表达式

**XML API支持**：来源XML信息可以使用包括标准JAXPAPI来解析，包括：DOM,SAX,StAX,JDOM, dom4j, XOM,甚至可以使用其他的xml序列化技术

**灵活的XML序列化**:Spring Web Services的对象-XML映射是建立在Spring的基础框架之上，后者支持很多的序列化工具，例如:JAXB 1 和 2, Castor, XMLBeans, JiBX, and XStream.

**重用你的Spring经验**：Spring-WS完全复用了Spring框架的配置信息，这使得Spring开发者可以快速上手，并且Spring-WS与Spring-MVC的架构非常类似

**支持web service安全**:WS-Security允许你对SOAP信息进行签名，加密，解密，授权

**与Spring Security进行整合**：集成自Spring Web Services的web service安全提供了与Spring Security的深度整合，这意味着你可以使用现有的Spring Security配置去配置你的SOAP服务


##1.2 运行时环境

Spring Web Services需要Java 1.6以及更高的运行时环境，需要Spring 3.2以及更高
Spring-WS是由一系列的模块组成，在剩下的章节中介绍

* XML模块(spring-xml.jar)包扩种类繁多的xml支持，这个模块主要是供Spring-WS自身使用，并不是一个web service的开发者
* 核心模块(spring-ws-core.jar)是Springweb service功能的核心部分，其提供了WebServiceMessage和SoapMessage的中心接口，服务侧的框架，强力的消息分发，多种Web service发布端点(endPoint)的实现支持，以及客户侧的WebService模板
* 支持模块(spring-ws-security.jar)提供额外的传输服务(JMS,EMAIL等)
*安全模块 (spring-ws-security.jar) 提供WS-Security的实现并整合在Web service的核心包中。这使得你可以添加主要的令牌，标记，加解密SOAP消息。不仅如此，它还可以允许你平衡你在Spring Security里对授权以及验证的实现。

下面这幅图中描述了Spring-ws和它所依赖的模块之间的关系，箭头方向代表依赖，例如：Spring-ws核心包依赖Spring-XML和Spring3当中的OXM模块

![](http://docs.spring.io/spring-ws/site/reference/html/images/spring-deps.png)



