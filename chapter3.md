#3.1 简介

这份指导书将展示如何编写一个契约优先的web service，这将是从XML WSDL协议开始，然后轮到java代码。Spring-ws关注的是开发风格，这份指导书将帮助你上手。需要说明的是，第一部分的几乎不包含Spring-ws内容：主要是XML,XSD以及WSDL。第二部分将关注如何使用Spring-ws实现。

当使用契约优先模式开发web service时，最主要的是尝试用XML的方式思考实现。这意味着java的概念并不重要。通过网络传输的是XML，这也正是我们的关注点。事实上，通过java实现web service只是实现细节，虽然这个细节是举足轻重的，但毕竟只是细节而已。

#3.2 消息

##3.2.1 假期

在这个场景里，我们要来处理关于“假日”的请求。因此如何描述XML显得至关重要：

``` xml
<Holiday xmlns="http://mycompany.com/hr/schemas">
    <StartDate>2006-07-03</StartDate>
    <EndDate>2006-07-07</EndDate>
</Holiday>
```
一个假期是由起始日期和结束日期组成，我们决定去使用ISO 8601标准去格式化我们的日期，因为可以省下很多的转换的麻烦。我们给这个元素增加一个命名空间，这样保证我们的元素可以被另一个XML文件里使用。

##3.2.2 雇员

在此场景里有一个雇员的概念，它在xml里面是这样的：、

``` xml 
<Employee xmlns="http://mycompany.com/hr/schemas">
    <Number>42</Number>
    <FirstName>Arjen</FirstName>
    <LastName>Poutsma</LastName>
</Employee>
```

我们之前使用过这个命名空间，如果这个<Element/>可以被使用在其他的场景里，可以考虑去使用另一个命名空间，例如：http://mycompany.com/employees/schemas.

##3.2.3 假期请求

假期和雇员这两个元素都可以被放置进<Holiday/>：

``` xml
<HolidayRequest xmlns="http://mycompany.com/hr/schemas">
    <Holiday>
        <StartDate>2006-07-03</StartDate>
        <EndDate>2006-07-07</EndDate>
    </Holiday>
    <Employee>
        <Number>42</Number>
        <FirstName>Arjen</FirstName>
        <LastName>Poutsma</LastName>
    </Employee>
</HolidayRequest>
```

两者的顺序并不重要：<Employee/>也可以被放在第一个元素。最重要的是所以的数据都在此一堂。事实上，只有数据才是唯一重要的：我们正在讨论的正是“数据驱动”的方法。

* DTDs
* XML Schema (XSD)
* RELAX NG
* Schematron

DTD有着命名空间的支持限制，所以其并不适合webservice，Relax NG和Schematron 显然比XML Schema更加容易。不幸的是，他们对平台支持的广泛程度并不是那么好，所以我们将继续使用XML Schema。




