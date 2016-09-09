title: Apache Commons SCXML系列之项目介绍
date: 2016-01-28 16:23:33
tags: [Commons SCXML,状态机实现,SCXML标准]
categories: [Commons-SCXML]
---


Apache Commons 工具包比较多，具体可以参看[Apache Commons官网](http://commons.apache.org/)。这里只讨论SCXML

# FAQ :
<br>
## SCXML 是什么？
>SCXML（<font color=red>State Chart XML</font>），简单地说就是状态图的xml描述文件。这里的状态图和UML里面的状态图是基本一致的，都是继承自[Harel](http://www.wisdom.weizmann.ac.il/~harel/%20%E4%B8%BB%E9%A1%B5) [Start Chart](http://www.wisdom.weizmann.ac.il/~harel/SCANNED.PAPERS/Statecharts.pdf)。我还没有看到不一致的地方，欢迎指正。

<!--more-->

## xml的描述文件？那么一个大家都遵从的标准是什么？
>标准就是W3C制定的的标准：[State Chart XML(SCXML):State Machine Notation for Control Abstraction](http://www.w3.org/TR/scxml/)下面我简要的介绍一下这个标准，如下图所示：红框里面的标签，每一个链接里面都有标签的具体介绍（子标签和属性值等）。就是这个标准。
![这里写图片描述](http://img.blog.csdn.net/20151126102750992)

</br>
## Apache 做什么了？
>W3C只是制定了这个标准，告诉大家怎么做。Apache组织成立了SCXML项目，对这个标准进行了java语言的实现，of course,其他组织个人有其他语言的实现等等。



## 都有哪些实现？看下表
| 名字 | 描述 |
| ------------- |:-------------:|
| [scxmlcc](https://github.com/jp-embedded/scxmlcc) | 用于将SCXML图生成c++代码 |
| [Apache Commons SCXML](http://commons.apache.org/proper/commons-scxml/) | 一个基于java语言的解析和执行SCXML图的标准类库 |
| [Legian](https://github.com/pelatimtt/Legian) | 一个java语言实现的SCXML引擎，使用Rhino作为javascript引擎，它还支持其他的特性，和W3C的标准并不完全一致|
|[Qt SCXML](http://qt.gitorious.org/qt-labs/scxml) | Qt里面的状态机，将信号和槽与状态机紧密的结合起来了 |
|[PySCXML](https://github.com/jroxendal/PySCXML) | 一个Python的实现，支持多种通信，包括的WebSockets和SOAP。兼容性好。还支持ECMAScript的数据模型。 （最后发布日期2013年） |
| The PySCXML Console | 一个基于web的交互式SCXML控制台，可以运行SCXML文档。支持ECMAScript datamodel。  |
| [SCXML4Flex](https://github.com/gdtiti/scxml4flex)  | ActionScript的实现 |
| [SCXMLgui](https://github.com/fmorbini/scxmlgui) |java实现的SCXML图可视化编辑器. |
| [SCION](https://github.com/jbeard4/SCION) | javascript实现 |
| [JSSCxml](https://github.com/Touffy/JSSCxml)| 这是一个基于Web浏览器的实现，目前还处于开发阶段。和标准高度一致,良好的支持DOM事件，目前只支持ECMAScript datamodel。 |
| [uSCXML](https://github.com/tklab-tud/uscxml) |Standard-compliant SCXML implementation in C/C++ with language bindings for Java and C#. Full ECMAScript support (all tests passed) via JavaScriptCore or Google's v8, additional LUA and Prolog datamodels, only rudimentary support for XPath datamodel.|

ECMAScript就是javascript语言的标准。


## 这个库 可以用来做什么？
>如果你<font color=red>画了一个UML状态图，想要实现它</font>？你怎么办。如果使用这个库的话，首先定义状态图的xml文件，然后编程加载xml文件执行，由你来写界面控制状态机的流转。我们都知道<font color=red>一切事物都可以看做对象，那么一切对象都有其生命周期，那么状态机就能力描述一切对象从生到死的过程</font>。你就能通过（程序）界面控制这个对象的生命周期。<font color=blue>如果你不知道什么是状态机的，快去百度百科吧。</font>

# 比较有用的链接

> 1. Apache Commons
    SCXML官网：http://commons.apache.org/proper/commons-scxml/

 >2. Apache Commons SCXML Github：https://github.com/apache/commons-scxml
 >
 >3. StackOverfole上面关于这方面的问题：http://stackoverflow.com/questions/tagged/scxml
 >
 >4. 可以在线编辑SCXML文件并运行和查看监控信息的网站：http://www.ling.gu.se/~lager/Labs/SCXML-Lab/

# 说明：
截止日期：2015-11-26，官网上面显示最新版本0.9（发布日期：07，08年左右），这个版本是用Ant打包编译的，整个项目包名是：com.apache.commons.scxml.*

现在官网RoadMap上面的计划显示推出scxml2.0，目前GitHub源代码使用Maven编译，整个项目的包名是：com.apache.commons.scxml2.*，变化很大，但是Apache 目前并没有正式发布这个库（Maven 中央仓库和官网上面都没有）。
我自己下载源码编译了一下，想直接使用的话可以点击这里下载：[SCXML2.0](http://pan.baidu.com/s/1nt90nLB)。我现在正在使用，可以说基本上都实现了标准里面的所有描述。
>提醒一下：由于这个库依赖于其他的Commons库，而这个库在Maven上面找不到，Maven就不能分析依赖关系，所有需要自己手动下载依赖的jar包。所依赖的jar包可以参考这里：[项目依赖](http://commons.apache.org/proper/commons-scxml/dependencies.html)，可以之间在Maven仓库或Gradle里面添加这些依赖的jar包就行了。或者你不会使用自动构建工具，可以点击这里下载：[项目需要的jar包](http://pan.baidu.com/s/1bnsjHWN)


后序会慢慢通过例子来介绍怎么编写xml文件，这个工具包怎么使用，如果有时间何以来分析一下源代码。
