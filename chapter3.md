#3.1 简介

这份指导书将展示如何编写一个契约优先的web service，这将是从XML WSDL协议开始，然后轮到java代码。Spring-ws关注的是开发风格，这份指导书将帮助你上手。需要说明的是，第一部分的几乎不包含Spring-ws内容：主要是XML,XSD以及WSDL。第二部分将关注如何使用Spring-ws实现。

当使用契约优先模式开发web service时，最主要的是尝试用XML的方式思考实现。这意味着java的概念并不重要。通过网络传输的是XML，这也正是我们的关注点。事实上，通过java实现web service只是实现细节，虽然这个细节是举足轻重的，但毕竟只是细节而已。

#3.2 消息

在这个场景里，我们要来处理关于“假日”的请求。因此如何描述XML显得至关重要：

``` xml
<Holiday xmlns="http://mycompany.com/hr/schemas">
    <StartDate>2006-07-03</StartDate>
    <EndDate>2006-07-07</EndDate>
</Holiday>
```


