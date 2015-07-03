#2 为何使用契约优先

## 2.1 简介

当创建web Service的时候，有两种不同的风格：契约优先和契约末后。当使用契约末后方案时，你是从java代码开始，然后用它生成Web service契约(WSDL)。当使用契约优先方案时，你会先从WSDL契约开始，然后使用java去实现刚刚声明的契约。

Spring-ws仅支持契约优先的开发风格，在这一章里会阐述为何如此。


## 2.2 对象-XML阻抗不匹配

和ORM类似，我们都具有对象-XML阻抗不匹配的问题，这里有一个相似的问题当java转换到xml。乍看之下，O/X映射问题很简单：为每一个java对象创建一个XML元素，将java属性和字段转换为子元素和属性(attribute)。然而，事情并不是这样简单：按照层级划分的XML语言和图表模式的java语言有一个本质的不同。


### 2.2.1 XSD扩展

在java中，唯一改变一个java类行为的方法就是去继承它，然后在子类中添加新的行为。在XSD中，你可以使用限制的办法扩展数据的类型：那便是使用正则来限制数据的输入类型。例如：

``` xml
<simpleType name="AirportCode">
  <restriction base="string">
      <pattern value="[A-Z][A-Z][A-Z]"/>
  </restriction>
</simpleType>
```

使用这种常规的表达方式XSD限制了输入的字符串必须是大写字母。如果是用java来表示，则只能是用String类型来声明，然后迷失在转换的过程当中了，因为java并不支持这种方式的扩展。


### 2.2.2 非便携类型

webservice最重要的目标之一便是相互协作：去支持多种平台，例如：java，.NET，Python等。因为这些语言有不同的类库，你必须使用一些相同的格式去彼此通信。这个便是XML，一种这些不同语言都支持的格式。因为需要这样的转换，你必须确认你使用的便携类型实现了你发布的服务，例如：你的服务返回java.util.TreeMap类型

``` java
public Map getFlights() {
  // use a tree map, to make sure it's sorted
  TreeMap map = new TreeMap();
  map.put("KL1117", "Stockholm");
  ...
  return map;
}
```

毋庸置疑，这种map是可以被转换为xml的，但是这种map在xml当中并没有一个标准的转换方式，这将会变得因人而异。不仅如此，即便你可以顺利将其转换为xml，很多平台并没有TreeMap这样的数据结构，所以当.NET客户端发布这样的服务时，很可能被解释为System.Collections.HashTable这种完全不同的语义。
同样的问题也会出现在客户端，思考下面的例子，一个如下的XSD片段：

``` xml
<element name="GetFlightsRequest">
  <complexType>
    <all>
      <element name="departureDate" type="date"/>
      <element name="from" type="string"/>
      <element name="to" type="string"/>
    </all>
  </complexType>
</element>
```

这个协议定义了一个请求获取日期的服务，在XSD中数据类型表示年，月，日。如果我们使用java调用这个服务，我们会使用java.util.Date或者java.util.Calendar来解析数据，然而这两种类型都是来描述时间的类，而不是日期。那么最终得到的结果将会是(2007-04-04T00:00:00), 而不是 2007-04-04.


### 2.2.3循环描述
想象一下下面这种数据结构：

``` java
public class Flight {
  private String number;
  private List<Passenger> passengers;
    
  // getters and setters omitted
}

public class Passenger {
  private String name;
  private Flight flight;
    
  // getters and setters omitted
}
```

这是一种循环描述：Flight引用了Passenger，而后者又引用了Flight。循环描述在java中非常常见，如果使用一种幼稚的方式来描述成xml，将会变成：

``` xml
<flight number="KL1117">
  <passengers>
    <passenger>
      <name>Arjen Poutsma</name>
      <flight number="KL1117">
        <passengers>
          <passenger>
            <name>Arjen Poutsma</name>
            <flight number="KL1117">
              <passengers>
                <passenger>
                   <name>Arjen Poutsma</name>
                   ...
```

这将会花费相当多的时间去结束，因为这里并没有循环的结束控制。
一个解决这个问题的办法是使用已经序列号完成的引用，例如：

``` xml
<flight number="KL1117">
  <passengers>
    <passenger>
      <name>Arjen Poutsma</name>
      <flight href="KL1117" />
    </passenger>
    ...
  </passengers>
</flight>
```

## 2.3 契约优先 vs 契约末后

除了我们之前谈论的对象-XML映射问题，还有其他原因使得我们使用契约优先的开发风格。

###2.3.1 脆弱性

正如前面提到的，契约末后的开发风格是使用java协议(通常是接口)来生成web Service协议(WSDL和你的XSD)。如果你使用这种方式，你将无法保证你的协议会一直保持不变。每次你更改你的java协议并重新部署，可能会给web service协议带来后续的变化。

不仅如此，并不是所有的SOAP堆栈从java协议里会生成相同的web service协议。这意味着，更改你当前的SOAP堆栈可能会更改你的web service协议。


当你的web service协议变更时，协议的使用者将会被指导去获取新的协议，并且可能还将修改其实现以适应新的变化。

所以为了提高协议的可用性，那么这个协议必须尽可能的不要变动。如果协议经常修改，那你将不得不联系所有的使用者，指导他们去适应新的协议。

###2.3.2 外在表现

当java被自动的转换为xml，将无法得知到底什么被发送了出去。一个对象引用了另一个对象或者其他，最终导致，虚拟机一半的堆内存都被用于转换xml，这将会导致响应速度降低。

当使用契约优先模式，你显式地指定了你想要发送出去的xml。

###2.3.3 重用性

在分散的文件中定义你的模式可以使你在不同的场景里复用。如果你在airline.xsd文件中做出如下定义：

``` xml
<simpleType name="AirportCode">
    <restriction base="string">
        <pattern value="[A-Z][A-Z][A-Z]"/>
    </restriction>
</simpleType>
```

你可以在其他场景甚至WSDL文件里复用你的定义，只要使用import声明。

###2.3.4 版本

尽管协议需要尽可能的保持稳定不变，但是有时还是需要进行修改。在java中，这将会导致接口的修改，或者定义新的接口。当然，旧的服务依旧需要保留，因为很多客户端并没有修改调用。

如果使用契约优先，我们将在实现和协议形成松耦合。这样的松耦合可以允许一个类实现多个不同版本的协议。比如我们可以将旧风格使用XSLT样式进行替换。

