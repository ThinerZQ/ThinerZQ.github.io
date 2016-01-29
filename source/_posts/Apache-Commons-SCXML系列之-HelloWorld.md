title: 'Apache Commons-SCXML系列之 HelloWorld '
date: 2016-01-28 16:38:52
tags: 
 Commons SCXML
 用法
 Hello World
categories: 
 Commons-SCXML
---

Commons-SCXML 是一个状态机框架，
<br>
阅读本文之前，最好有对UML状态机有一个基本认识

# 编程思路：
>1. <font color=blue>先画出状态图（uml状态图）</font>：（这一步只是为了能直观的表现状态的变化，可以随便在纸上画或者使用EA，Rose等工具，）
>2. <font color=blue> 编写状态图xml文件定义</font>：根据画的状态图，编写对应的xml文件。
>3. <font color=blue>编写程序加载xml文件</font>，编写界面，控制状态图的状态转移。

我们通过一系列的例子来讲学习SCXML标准和Commons-SCXML框架。

# HelloWorld例子
## 画出状态图
>本例比较简单就不画图了

## 编写状态图xml文件
```xml
<scxml xmlns="http://www.w3.org/2005/07/scxml"
       version="1.0" inital="end">
    <final id="end">
        <onentry>
            <log expr=" 'entry  final state end ' " />
            <log expr="'hello world'"/>
        </onentry>
        <onexit>
            <log expr=" 'exit  final state end ' " />
        </onexit>
    </final>
</scxml>
```
说明：如果你用的IDE在xmlns上面有报错，不用管它。
## 编程控制转移
```java
package helloworld;

 /**
 * Created with IntelliJ IDEA
 * Date: 2015/11/19
 * Time: 20:53
 * User: ThinerZQ
 * GitHub: <a>https://github.com/ThinerZQ</a>
 * Blog: <a>http://thinerzq.me</a>
 * Email: 601097836@qq.com
 */

import org.apache.commons.scxml2.SCXMLExecutor;
import org.apache.commons.scxml2.io.SCXMLReader;
import org.apache.commons.scxml2.model.SCXML;
import java.net.URL;

public class HelloWorld {
    
    //通过加载HelloWorld类的类加载器加载helloworld.xml资源文件，得到URL
    private static final URL HELLOWORLD = HelloWorld.class.getResource("helloworld.xml");

    public static void main(String[] args) throws Exception {
        
        //得到xml文件所对应的 SCXML对象
        SCXML scxml = SCXMLReader.read(HELLOWORLD);
        
        //实例化状态机引擎，
        SCXMLExecutor executor = new SCXMLExecutor();

        //将得到的SCXML对象，交给状态机引擎管理
        executor.setStateMachine(scxml);

        //然后引擎调用.go()方法启动状态机。
        executor.go();

    }
}   
```

##3.4 分析
![这里写图片描述](http://img.blog.csdn.net/20151127210902233)
>上面的输出内容，和xml文件<code>log</code>标签里面定义的一样。<code>log</code>是一个记录日志的可执行内容。


接下来准备再写一个 秒表和请假流程的例子，再把相关的标签和属性值约束给汉化了。