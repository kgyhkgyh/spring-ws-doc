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


目前为止，创建一个XSD的最简单办法是从一个样例中推导得出，任何的一个java IDE或者XML编辑器都支持此功能。基本上，这些工具都是使用样例xml文件，然后生成并校验它。
其结果当然还需呀打磨，不过这是一个很好的出发点。

使用上面描述的例子，我们最终得到如下schema：

``` xml
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
        elementFormDefault="qualified"
        targetNamespace="http://mycompany.com/hr/schemas"
        xmlns:hr="http://mycompany.com/hr/schemas">
    <xs:element name="HolidayRequest">
        <xs:complexType>
            <xs:sequence>
                <xs:element ref="hr:Holiday"/>
                <xs:element ref="hr:Employee"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
    <xs:element name="Holiday">
        <xs:complexType>
            <xs:sequence>
                <xs:element ref="hr:StartDate"/>
                <xs:element ref="hr:EndDate"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
    <xs:element name="StartDate" type="xs:NMTOKEN"/>
    <xs:element name="EndDate" type="xs:NMTOKEN"/>
    <xs:element name="Employee">
        <xs:complexType>
            <xs:sequence>
                <xs:element ref="hr:Number"/>
                <xs:element ref="hr:FirstName"/>
                <xs:element ref="hr:LastName"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
    <xs:element name="Number" type="xs:integer"/>
    <xs:element name="FirstName" type="xs:NCName"/>
    <xs:element name="LastName" type="xs:NCName"/>
</xs:schema>
```

这份生成的schema显然可以被改进的。第一点需要注意的是，每种类型的元素都是有根节点的开始的，这意味着web service将会接收这里面的所有数据。这并不是我们想要的，我们只是想要接收<HolidayRequest/>。通过移除打包元素的标签(从而保留了类型)，而且直列结果，我们可以将其完成如下：

``` xml
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
        xmlns:hr="http://mycompany.com/hr/schemas"
        elementFormDefault="qualified"
        targetNamespace="http://mycompany.com/hr/schemas">
    <xs:element name="HolidayRequest">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="Holiday" type="hr:HolidayType"/>
                <xs:element name="Employee" type="hr:EmployeeType"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
    <xs:complexType name="HolidayType">
        <xs:sequence>
            <xs:element name="StartDate" type="xs:NMTOKEN"/>
            <xs:element name="EndDate" type="xs:NMTOKEN"/>
        </xs:sequence>
    </xs:complexType>
    <xs:complexType name="EmployeeType">
        <xs:sequence>
            <xs:element name="Number" type="xs:integer"/>
            <xs:element name="FirstName" type="xs:NCName"/>
            <xs:element name="LastName" type="xs:NCName"/>
        </xs:sequence>
    </xs:complexType>
</xs:schema>
```
这份schema依旧还存在问题：形如这样的一份schema，你可以预期如下的消息校验：

``` xml 
<HolidayRequest xmlns="http://mycompany.com/hr/schemas">
    <Holiday>
        <StartDate>this is not a date</StartDate>
        <EndDate>neither is this</EndDate>
    </Holiday>
    <!-- ... -->
</HolidayRequest>
```
再明显不过，我们必须保证开始和结束日期是真正的日期格式，XML Schema有一个内置的日期类型供我们使用。我们也可以改变NCName为string。最终，改变<HolidayRequest/>里的所有sequence，这将告诉xml解释器，这里面的内容并不重要，最终的XSD将如下：

``` xml
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
        xmlns:hr="http://mycompany.com/hr/schemas"
        elementFormDefault="qualified"
        targetNamespace="http://mycompany.com/hr/schemas">
    <xs:element name="HolidayRequest">
        <xs:complexType>
            <xs:all>
                <xs:element name="Holiday" type="hr:HolidayType"/>                                                     (1)
                <xs:element name="Employee" type="hr:EmployeeType"/>
            </xs:all>
        </xs:complexType>
    </xs:element>
    <xs:complexType name="HolidayType">
        <xs:sequence>
            <xs:element name="StartDate" type="xs:date"/>
            <xs:element name="EndDate" type="xs:date"/>                                                                (2)
        </xs:sequence>                                                                                                 (2)
    </xs:complexType>
    <xs:complexType name="EmployeeType">
        <xs:sequence>
            <xs:element name="Number" type="xs:integer"/>
            <xs:element name="FirstName" type="xs:string"/>
            <xs:element name="LastName" type="xs:string"/>                                                             (3)
        </xs:sequence>                                                                                                 (3)
    </xs:complexType>
</xs:schema>
```


(1) *all* 意思是告诉xml转换器 <Holiday>和<Employee>的顺序并不重要
(2)我们使用 *xsd:data*数据类型，它是由年，月，日组成
(3)*xsd:string*被用作解释first和last name

我们保存文件 hr.xsd


#3.4 服务协议

一个服务协议一般表现为一个WSDL文件，这个在Spring-ws中是不需要自己手写的。基于XSD和一些转换，Spring-ws可以自动为你创建WSDL，这个在章节3.6 “实现一个端点”会做出中解释。你可以选择跳过下一章，那一章节里会讲到如何手写一个WSDL文件。

我们从标准以及通过导入XSD的前言开始WSDL，来区分schema和定义，我们将为WSDL使用分隔的命名空间：http://mycompany.com/hr/definitions.

```xml
<wsdl:definitions xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/"
                  xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
                  xmlns:schema="http://mycompany.com/hr/schemas"
                  xmlns:tns="http://mycompany.com/hr/definitions"
                  targetNamespace="http://mycompany.com/hr/definitions">
    <wsdl:types>
        <xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema">
            <xsd:import namespace="http://mycompany.com/hr/schemas" schemaLocation="hr.xsd"/>
        </xsd:schema>
    </wsdl:types>
```

下一步，我们加入自己的基于已写schema类型的消息。我们目前只有一个消息：<HolidayRequest/>

```xml
<wsdl:message name="HolidayRequest">
        <wsdl:part element="schema:HolidayRequest" name="HolidayRequest"/>
</wsdl:message>
```

我们为端口类型加入消息：

```xml
<wsdl:portType name="HumanResource">
    <wsdl:operation name="Holiday">
        <wsdl:input message="tns:HolidayRequest" name="HolidayRequest"/>
    </wsdl:operation>
</wsdl:portType>
```
如此便结束了WSDL抽象部分的编写(接口，如同之前那样)，余下具体的部分。具体的部分是由一个告诉客户端如何调用操作定义的binding，和一个告诉你到哪里去调用它的service组成。

添加一个具体部分十分程式化，只需要引用你之前定义的抽象部分，确认你为soap:binding元素使用的Document/Literal模式(rpc/encoded模式并不赞成使用)，选择一个soapAction的操作(在本例中是http://mycompany.com/RequestHoliday，但是任何uri都是可以的)，决定你的请求入口(在这里是http://mycompany.com/humanresources)：

```xml
<wsdl:definitions xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/"
                  xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
                  xmlns:schema="http://mycompany.com/hr/schemas"
                  xmlns:tns="http://mycompany.com/hr/definitions"
                  targetNamespace="http://mycompany.com/hr/definitions">
    <wsdl:types>
        <xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema">
            <xsd:import namespace="http://mycompany.com/hr/schemas"                      (1)
                schemaLocation="hr.xsd"/>
        </xsd:schema>
    </wsdl:types>
    <wsdl:message name="HolidayRequest">                                                 (2)
        <wsdl:part element="schema:HolidayRequest" name="HolidayRequest"/>               (3)
    </wsdl:message>
    <wsdl:portType name="HumanResource">                                                 (4)
        <wsdl:operation name="Holiday">
            <wsdl:input message="tns:HolidayRequest" name="HolidayRequest"/>             (2)
        </wsdl:operation>
    </wsdl:portType>
    <wsdl:binding name="HumanResourceBinding" type="tns:HumanResource">                  (4)(5)
        <soap:binding style="document"                                                   (6)
            transport="http://schemas.xmlsoap.org/soap/http"/>                           (7)
        <wsdl:operation name="Holiday">
            <soap:operation soapAction="http://mycompany.com/RequestHoliday"/>           (8)
            <wsdl:input name="HolidayRequest">
                <soap:body use="literal"/>                                               (6)
            </wsdl:input>
        </wsdl:operation>
    </wsdl:binding>
    <wsdl:service name="HumanResourceService">
        <wsdl:port binding="tns:HumanResourceBinding" name="HumanResourcePort">          (5)
            <soap:address location="http://localhost:8080/holidayService/"/>             (9)
        </wsdl:port>
    </wsdl:service>
</wsdl:definitions>
```































