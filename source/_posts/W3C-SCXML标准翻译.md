title: State Chart XML (SCXML) 状态机规范
date: 2016-02-01 21:53:08
updated: 2016-02-05 22:10:23
tags: 
 SCXML标准翻译
 SCXML标准
categories: 
 w3c
 SCXML
---

# 摘要
这篇文档描述了SCXML，或者说是“状态图可扩展标记语言”。SCXML基于CCXML和Harel State Tables为状态机提供了一个一般性的可执行环境，


# 概述
这篇文档描述的SCXML，是一种基于事件的状态机语言。它是CCXML和Harel State Tables 结合的产物。CCXML是一种基于事件的状态机语言，被设计用来在语音应用中支持通话控制。CCXML1.0规范定义了一个状态机和事件处理语法以及一系列的通话控制元素。Harel State Table由David Harel于1987年提出的一种状态机记号，后来UML中的状态机沿用了这些记号。Harel State Tables 提供了一个简洁、语义良好和功能强大的控制结构。这篇文章就是使用XML语法和Harel State Table语义来描述CCXML中状态和事件转移的逻辑概念

<!--more-->

# 核心结构
[这部分是非正式的]
## 介绍
### 基础的状态机概念
### 组合状态
### 并行状态
### 初始化，终止和历史状态
### 转移类型
## scxml
[这部分是规范的]  
文档的顶层元素，携带了版本信息。一个状态机由&lt;scxml&gt;和他的孩子元素共同组成。
>提示：任何时候只有一个孩子是处于活跃状态。  

### 属性
|名字|必须|约束|类型|默认值|有效值|描述|
|:---|:---|:----|:---|:---|:---|:---|
| inital | False | None | Id引用 | None |  | 初始状态的id值，如果没有指定，默认的初始状态是文档中第一个孩子状态 |
| name | False | None |	名称记号 | None |  |任何有效的名称记号	状态机的名字 |
| xmlns |True | None | URI | None | 必须是：“http://w3.org/005/07/scxml” |  |
| version |True | None | 数字 | None | 必须是：“1.0” |  |
| datamodel |False | None | 名称记号 | 自定义 | “null”,“ecmascript”,”xpath”，或者实现者定义的值 | 表示状态机中的数据模型 |
| binding |False | None | Enum | early | "early","late" | 指明数据绑定的时间 |

### 孩子元素

|名字名字|说明|出现次数|
|:---|:---|:----|
| &lt;state&gt; | 一个组合状态或者原子状态 | 0…* |
| &lt;parallel&gt; | 一个并行状态 | 0…* |
| &lt;final&gt; | 一个顶层的终止状态，当状态机达到这个状态必须终止 | 0…* |
| &lt;datamodel&gt; | 定义了部分或者所有的数据模型 | 0…1 |
| &lt;script&gt; | 提供了全局脚本的能力 | 0…1 |

### 说明
一个结构良好的SCXML文档必须至少有一个&lt;state&gt;,或者&lt;parallel&gt;或者&lt;final&gt;孩子，在系统初始化的时候，如果执行呢‘initial’属性，SCXML Processer必须进入 ‘initial’属性指定的状态中。如果没有执行‘initial’属性，Processer必须进入文档中的第一个状态。平台应该顶一个默认的datamodel值。


## state
[这部分是规范的]  


### 属性
|名字|必须|约束|类型|默认值|有效值|描述|
|:---|:---|:----|:---|:---|:---|:---|
| id | False | None | Id引用 | None | 一个有效的id定义  | 状态的标识符 |
| initial | False | 不能和&lt;initial&gt;元素同时指定，不能在原子状态内部指定 |	名称记号 | None |  |另一个状态的ID标识符，但是要保证配置后是合法的状态机配置	复合状态或者并发状态默认初始化的状态 |

### 孩子元素

|名字名字|说明|出现次数|
|:---|:---|:----|
| &lt;onentry&gt; | 可选的元素，用来呈现进入这个状态时可执行内容 | 0…* |
| &lt;onexit&gt; | 可选的元素，用来呈现退出这个状态时可执行内容 | 0…* |
| &lt;transition&gt; | 定义了一个转移，源状态是当前状态 | 0…* |
| &lt;initial&gt; | 复合状态机的子状态，定义了默认的初始化状态 | 0…1 |
| &lt;parallel&gt; | 定义了并发子状态 | 0…* |
| &lt;final&gt; | 定义了一个final子状态 | 0…* |
| &lt;history&gt; | 一个伪状态机，记录了上转移出此状态机的子孙状态 | 0…* |
| &lt;datamodel&gt; | 定义了部分或者所有的数据模型 | 0…1 |
| &lt;invoke&gt; | 调用一个外部服务 | 0…* |

### 定义
定义1 ：一个原子（atomic）状态是一个没有&lt;state&gt;元素，&lt;parallel&gt;或者&lt;final&gt;孩子的&lt;state&gt;    

定义2 ：一个复合（compound）状态是一个有&lt;state&gt;元素，&lt;parallel&gt;或者&lt;final&gt;孩子（或者这几者的组合）的&lt;state&gt;  

定义3 ：一个复合状态的默认的初始化状态由‘initial’属性或者&lt;initial&gt;元素指定，如果前两者都没有出现，那么初始化状态就是当前复合状态的按文档顺序来说的第一个孩子状态

### 说明
在一个结构良好的SCXML文档，一个复合状态必须制定‘initial’属性或者一个&lt;initial&gt;元素，但是二者不能同时指定。

## parallel
[这部分是规范的]  
&lt;parallel&gt;元素封装了一系列的孩子状态，如果并行状态是处于活跃状态，当孩子元素的父元素是active的，这些孩子状态会同时处于active状态

### 属性
|名字|必须|约束|类型|默认值|有效值|描述|
|:---|:---|:----|:---|:---|:---|:---|
| id | False |  | Id | None | 一个有效的id定义  | 状态的标识符 |
| initial | False | 不能和&lg;initial&gt;元素同时指定，不能在原子状态内部指定 |	名称记号 | None |  |另一个状态的ID标识符，但是要保证配置后是合法的状态机配置	复合状态或者并发状态默认初始化的状态 |

### 孩子元素

|名字名字|说明|出现次数|
|:---|:---|:----|
| &lt;onentry&gt; | 可选的元素，用来呈现进入这个状态时可执行内容 | 0…* |
| &lt;onexit&gt; | 可选的元素，用来呈现退出这个状态时可执行内容 | 0…* |
| &lt;transition&gt; | 定义了一个转移，源状态是当前状态 | 0…* |
| &lt;initial&gt; | 复合状态机的子状态，定义了默认的初始化状态 | 0…1 |
| &lt;parallel&gt; | 定义了并发子状态 | 0…* |
| &lt;final&gt; | 定义了一个final子状态 | 0…* |
| &lt;history&gt; | 一个伪状态机，记录了上转移出此状态机的子孙状态 | 0…* |
| &lt;datamodel&gt; | 定义了部分或者所有的数据模型 | 0…1 |
| &lt;invoke&gt; | 调用一个外部服务 | 0…* |


## transition
[这部分是规范的]  
states 之间的Transition是有 事件（Events）和条件（Condition）触发的。在Transition中可以包含可执行内容（executable content），当转移发生的时候这些可执行内容将会执行。

### 属性
|名字|必须|约束|类型|默认值|有效值|描述|
|:---|:---|:----|:---|:---|:---|:---|
| event | False |  | EventTypes.datatype | None | 一个使用”.”分割符指定的字符串  | 触发这个转移的事件 |
| cond | False | |	Boolean表达式 | True | 任何boolean表达式 | 转移的守护条件 |
| target | False | | ID引用 | None |  | 转向的状态或者并行状态的标识符 |
| type | False | |	Enum | external | "internal","external" | 当目标状态是子孙状态的时候,决定是否源状态要不要退出再重新进入 |

### 孩子元素
可执行内容都可以作为孩子元素，

### 说明
一份结构良好的SCXML文档必须至少指定 ‘event’ ,'cond' 或者'target'中的一个。

## initial
[这部分是规范的]  

这个元素表示一个复合状态默认的初始化状态

### 属性
本元素没有属性

### 孩子元素
|名字名字|说明|出现次数|
|:---|:---|:----|
| &lt;transition&gt; | 定义了一个转移，这个转移不能指定cond和event属性，但是必须指定一个非空的有效的target属性，属性值必须是当前容器状态的子孙状态.可以包含可执行内容 | 1 |



## final
[这部分是规范的]  
这个元素表示&lt;scxml&gt;或者复合状态中的最终状态

### 属性
|名字|必须|约束|类型|默认值|有效值|描述|
|:---|:---|:----|:---|:---|:---|:---|
| id | False |  | Id | None | 一个有效的id值  | 状态的标识符 |

### 孩子元素

|名字名字|说明|出现次数|
|:---|:---|:----|
| &lt;onentry&gt; | 可选的元素，用来呈现进入这个状态时可执行内容 | 0…* |
| &lt;onexit&gt; | 可选的元素，用来呈现退出这个状态时可执行内容 | 0…*(个人觉得&lt;scxml&gt;节点下的&lt;final&gt;元素不应该有&lt;onexit&gt;)孩子节点 |
| &lt;donedata&gt; | 指定数据被包含在done.state.id 或者 done.invoke.id 事件里面 |  |

### 说明
当状态机进入一个&lt;state&gt;元素的&lt;final&gt;孩子节点的时候，SCXML Processor在完成了&lt;onentry&gt;元素中可执行内容之后 必须生成一个done.state.id事件，id是父亲状态的id值。然后，如果父状态是一个&lt;parallel&gt;的孩子元素，并且所有的&lt;parallel&gt;的孩子都进入了&lt;final&gt;状态，Processor必须生成一个done.state.id 事件，id是&lt;parallel&gt;元素的id值。

当状态机达到了&lt;scxml&gt;元素的&lt;final&gt;孩子时候，状态机必须终止。如果当前的SCXML状态机（以后简称SCXML session 会话）是被另外一个状态机（Session）的&lt;invoke&gt;元素触发的。当当前session终止的时候，SCXML Processor 必须生成一个done.invoke.id 事件，并且返回这个事件给调用的那个session, id的值是一个唯一的标识符。


## onentry
[这部分是规范的]  
一个包含可执行内容的元素，当进入某一个状态的时候，&lt;onentry&gt;内的内容被执行

### 属性
没有属性|

### 孩子元素
可执行内容都可以在这里定义
### 说明

SCXML Processor 必须按文档中定义的顺序，执行&lt;onentry&gt;元素和元素内的内容。


## onexit
[这部分是规范的]  
一个包含可执行内容的元素，当退出某一个状态的时候，&lt;onexit&gt;内的内容被执行

### 属性
没有属性|

### 孩子元素
可执行内容都可以在这里定义
### 说明

SCXML Processor 必须按文档中定义的顺序，执行&lt;onexit&gt;元素和元素内的内容。

## history
[这部分是规范的]  
&lt;history&gt;伪状态允许一个状态机记住它的状态配置。如果一个转移使用&lt;history&gt;状态作为它的目标状态，将会返回状态机记录到的状态配置。伪状态知识参考uml状态图

### 属性
|名字|必须|约束|类型|默认值|有效值|描述|
|:---|:---|:----|:---|:---|:---|:---|
| id | False |  | Id | None | 一个有效的id值  | 这个伪状态的标识符 |
| type | False |  | Enum | "shallow" | “deep”,”shallow”  | 决定了究竟是当前状态的活跃的原子状态还是，当前状态向内一层的活跃的子状态被记录 |

### 孩子元素

|名字名字|说明|出现次数|
|:---|:---|:----|
| &lt;transition&gt; | 转移的目标指定了默认的历史配置。 | 1 |


### 说明
在一份结构良好的SCXML文档中，这个转移不能包含cond和event属性，但是必须包含一个非空的target属性，这个值构成了一个有效的状态配置。

## 合法的状态配置
### 定义
定义4：一个&lt;state&gt;或者&lt;parallel&gt;元素是active：如果它已经通过一个转移进入了这个元素，并且没有退出。
定义5：状态机的状态配置是一个当前 active 状态的集合

### 说明
一个SCXML文档，初始化的时候初始化一个状态机配置（通过initial属性或者&lt;scxml&gt;里面的&lt;initial&gt;元素），此后状态机的每一个转移都将导致状态机进入另一个状态配置（不需要和前一个有区别）。一个结构良好的scxml文档必须将状态机置于一个合法的状态配置下，一个合法的状态配置满足下面的条件：
1 配置明确的只包含一个&lt;scxml&gt;的元素
2 配置包含0到多个原子状态
3 当当前配置包含一个原子状态，当前配置必须包含这个原子状态所有的&lt;state>&gt;&lt;parallel&gt;祖先
4 当配置包含一个非原子的&lt;state&gt; ，配置必须包含&lt;state&gt;的一个，仅且一个孩子。
如果配置包含&lt;parallel&gt;状态，配置必须包含所有的孩子。

如果一个尊从上面定义的状态机有超过一个原子状态，那么任何一个原子状态都能通过他们的父亲和祖先&lt;state&gt;或者&lt;parallel&gt; 回溯到同一个&lt;parallel&gt;祖先

一个转移的“target”属性或者initial属性要么是空，要么指定的状态是一个合法的状态配置。

## SCXML Events
[这部分是规范的]  
Events是SCXML中的一个基础概念，他们驱动了转移。事件的内部结构是由实现指定的，只要如下的外部接口是可见的
1、	SCXML Processor 必须使数据在一个事件中可以通过“_event”变量访问
2、	SCXML Processor 必须使事件名字可以通过“_event”变量访问。SCXML Processor必须使用相同的名字值匹配转移的event属性
在很多情况下，事件是在SCXML会话执行过程中产生的，我们可以通过&lt;raise&gt;和&lt;send&gt;元素来控制。有一些事件是必须要产生的，或者解释器自动生成的。实现者可以通过添加后缀的方法来扩展这些自动生成的事件名字。例如，一个平台可以扩展done.state.id 用一个时间戳作为后缀done.state.id.timestamp，任何匹配这个事件的转移都将转移。

### 事件描述符
像一个事件名字一样，一个事件描述符是一系列通过“.”分割的的字母数字符号。一个转移的事件属性，可以同包含一个或多个这样的事件描述符，通过空格隔开

#### 定义
定义6：一个转移匹配一个事件，至少一个事件描述符和事件的名字匹配
定义7：一个事件描述符匹配事件属性的名字，如果事件属性能够和描述符完全匹配或者，时间名能够匹配上事件描述符的每一个”.”之前的部分。

例如，一个转移的事件属性event=” error foo”，将会匹配事件名”error”,”error.send”,”error.send.failed”,或者“foo”,”foo.bar”等，但是不能匹配“errors.my.custom”,”errorhandler.mistake”,”error.send”,”foobar”等（仔细体会一下吧）。为了更好的阅读SCXML文档，一个evnet属性可以这样写event=”error”,或者event=”error.”,或者event=”error.*”来匹配任意满足这个前缀的事件。这种事件匹配机制和没有事件属性指定的转移有一点不同：没有事件属性指定的转移，不会匹配任何事件，只要cond属性为真就转移，每当第一次进入一个状态的时候，SCXML Processor应该先寻找这样没有任何事件属性的转移来进行判断。

### 错误（Errors）
一旦一个SCXML Processor开始执行一个形式良好的SCXML文档，它必须将过程中任何内部错误以事件“error.”作为前缀抛出。Processor必须将这个事件放入到内部事件队列像处理其他事件一样处理它。（如果队列里面还有其他事件，这个error事件不会立即被处理，并且如果在当前状态下没有转移匹配这个事件，这个错误事件将会被忽略）。这里定义了两个error事件：“error.communication” 和”error.execution”.前者覆盖了当试图与外部实体进行通信的时候发生的错误，例如&lt;send&gt;和&lt;invoke&gt;里面的错误；后者定义了当前Session内部执行中的错误，例如表达式解析错误等。

这一些列的error事件，将来可能会被扩展。然而，这些以“error.platform”开头的名字将被保留作为平台实现者定义平台和应用相关的错误。因此，平台或者应用可以通过两种方式来扩展这些错误：在事件名上添加一个后缀，或者使用“error.platform”添加一个后缀。另外，平台还可添加额外的错误信息在event的“data”变量里面。

Xml文件编写者可以通过创建一个带有event=“error”的属性的转移，并且目标是顶层的final 状态，这样可以使程序一旦发生错误就终止。如果这样的一个转移T被放置在状态S里面，如果S或者S的任何子状态有任何其他转移t（t是被放置在S的子状态中，或者S中T转移的文档顺序之前）未处理的错误，转移T将会导致状态机终止。

### 错误和事件列表
|名字|描述|定义在|参考|
|:---|:---|:----|:---|
| done.state.id | 表明状态机已经进入一个某一个复合状态（id）的final子状态中 | &lt;final&gt; | |
| done.invoke.id | 表明被&lt;invoke id=””&gt;元素调用的过程已经结束了 | &lt;invoke&gt; |&lt;final&gt;退出解释器程序，SCXML解释器的算法|
| error.communication | 表明当试图和一个外部实体通信的时候，有错误发生 | | &lt;send&gt; ,SCXML的Event I/O Processor|
| error.execute | 表明状态机已经进入一个某一个复合状态（id）的final子状态中 | | &lt;foreach&gt;&lt;assign&gt;&lt;param&gt;条件表达式，位置表达式，合法的数据值和数据表达式，表达式错误，系统变量|
| error.platform | 表明一个平台或者应用相关的错误发生了 |  | |

## 选择和执行转移
为了简化下面的定义，我们介绍一个事件：NULL.NULL没有名字，被使用在这些定义里面。它从不进入到事件队列里面，所有的有名字的时间和NULL是截然不同的。（事实上，NULL是一个伪事件，只是被用来在这些定义里面作为一个无事件转移的触发器）。

定义8：在一个原子状态S中的一个具有event属性(“event=E”)的转移T是enabled,  

-  A、	如果T的源状态是S或者S的一个祖先
-  B、	如果T 匹配E的名字
-  C、	T缺少一个cond属性，或者cond属性为真  

在一个原子状态S中一个转移（NULL事件）是enabled  

* A、	T缺少一个event 属性
* B、	如果T的源状态是S或者S的一个祖先
* C、	T缺少一个cond属性，或者cond属性为真

定义9：一个转移的源状态是&lt;state&gt;或者&lt;parallel&gt;元素，有效的目标状态是是一个或者一些通过target属性指定的状态，任何history状态将会被系统存储的状态配置或者默认配置替代。一个转移完整的目标集是由所有转移发生后的active状态构成的。这个目标集包含这个转移有效的目标状态，和这些状态的所有祖先。可以通过递归来扩展这些应用程序：

1.	如果任何&lt;parallel&gt;元素是这个结合中的一个元素，如果它的任何孩子没有包含在这个集合里面，那么必须添加进去。

2.	如果任何组合&lt;state&gt;是这个结合中的一个元素，如果它没有孩子在这个结合中，那么它的默认转移将被添加到集合
定义：在配置C中一个转移的退出的集合是一个状态的集合（当状态机在配置C当中的时候，当转移发生的时候，将发生退出动作的状态）如果转移没有指定target属性，退出集是空的。如果转移指定了target属性，并且type属性是“external”,它的退出集合就是配置C中的所有活跃状态（这些活跃状态是源状态和目标状态最近的公共的祖先（least common compound ancestor））。如果转移的类型是”internal”，但是源状态不是compound复合状态，或者目标状态不是合适的源状态的子孙状态，这个转移的退出集就和type=”external”一样。多个转移的退出集是每个转移退出集的并集（U）

定义10：在配置C中一个转移的进入集合是一个状态的集合（当转移发生的的时候，这些状态将进入）。如果一个转移没有包含一个target属性，他的进入集合就是空的。进入集合由转移的完成目标集合

定义11：原子状态S下面的一个转移T（event=”E”）是最优的enabled的转移满足下面3个条件  

1.	T是S中的转移，是通过E达到enabled
2.	文档顺序中没有转移优先于T能处理E事件
3.	在S中没有转移是通过E事件enabled，或者任何T的源状态所在的子孙状态能处理E事件

定义11：在配置C中两个转移T1和T2 冲突了，如果他们的退出集合的交集是非空的
例如，如果两个转移是冲突的，这两个转移都发生将导致非配的配置，因此，只有一个转移可以安全的执行。为了解决这个冲突，我们提出了优先级的概念。如果T1和T2冲突，T1是位于原子状态S1中的，T2是处于原子状态S2中的，S1和S2都是active的。满足以下条件之一我们说T1的优先级比T2高：

1.	T1的源状态是T2源状态的子孙
2.	S1 在文档中的位置是在T2前面

定义12：一个microstep 由位于optimal enabled 转移集合中的转移的执行组成

定义13：一个microstep是一系列一个或者多个mircrostep，达到的最终效果是：内部事件队列是空的，没有转移是通过NULL enabled。

执行一个microstep，SCXML Processor必须执行所对应的optimal enabled 转移集合里面的转移。为了执行这些转移，SCXML Processor必须首先以某一种退出顺序退出这些转移退出集合里面的所有状态。它必须以文档先后顺序执行包含在转移里面的可执行内容，接下来它必须以进入顺序进入这些转移的进入集合。

为了退出一个状态，SCXML Processor必须执行&lt;onexit&gt;里面的可执行内容。它必须取消掉在本状态中触发的的还处于进行中的invocation。最终Processor必须从active state list
移除这个状态.
为了进入一个状态，SCXML Processor必须将这个状态添加到active list。然后执行&lt;onentry&gt;里面的内容。如果状态是一个复合状态有一个&lt;initial&gt;孩子，SCXML Processor必须执行&lt;initial&gt;里面转移的可执行内容。

在开始的时候，SCXML Processor必须将状态机放入到一个通过initial属性指定的配置环境中。

在进入了初始化配置之后，在执行了每一个microstep之后，SCXML Processor必须检查在这个microstep中状态配置是否进入了&lt;final&gt;状态。如果进入了&lt;scxml&gt;元素的&lt;final&gt;状态，那么状态机停机。如果进入了某一个复合状态的&lt;final&gt;那么生成一个done.state.id时间，id是复合状态的id值。如果复合状态本身是一个&lt;parallel&gt;元素的孩子，并且&lt;parallel&gt;元素的其他孩子都进入了&lt;final&gt;状态，Processor必须根生一个done.state.id事件，id是&lt;parallel&gt;元素的id

在检查完配置之后，Processor必须选择当前配置下通过NULL enabled的optimal 转移集合。如果集合不是空的，执行下一个microstep.如果集合是空的，Processor必须从internal event queue移除所有事件，直到queue是空的，并且在当前配置下它找到了一个event，这个事件enabled一个非空的optimal转移集合，然后Processor必须执行这样一个mircrostep。

在完成一个microstep之后，SCXML Processor必须按照文档顺序执行各个状态下&lt;invoke&gt;里面的内容。然后Processor必须移除事件从external event queue，等待需要的生成生成，然后找到一个enables 非空的转移集合。然后Processor像microstep一样执行这个集合。

## IDS
在一个良好的SCXML 文档里面，每一个id属性的值在某一个session会话里面必须是唯一的。当这样的一个属性被定义成optional的时候，并且你忽略了这个id。SCXML processor会在文档加载的时候自动生成一个唯一的id值（除了&lt;send&gt;和&lt;invok&gt;元素）（注意：这样生成一个一个id不能够在文档其他地方被引用，因为你不知道它的值是多少，特别是如果一个状态的id是自动生成的，那么这个状态不能作为任何转移的目标）。&lt;send&gt;和&lt;invoke&gt;的id属性有一点不同，当作者没有指定这两个元素的id的时候，系统不会在加载文档的时候生成这两个元素的id，而是每次这两个元素执行的生成生成id.而且idlocation属性能够捕捉到这个自动生成的id。&lt;invoke&gt;标签自动生成的id有一个特殊的形式，参见后面的&lt;invoke&gt;标签。SCXML Processor可以生成所有其他的id以任何的形式，只要是唯一的。

# 可执行内容
可执行内容允许状态机to do things. 它提供了hooks ，允许一个SCXML session去修改它的数据模型和与外部实体交互。可执行内容由一系列actions组成。特别的，可执行内容，发生在&lt;onentry&gt;和&lt;onexit&gt;元素内，或者&lt;transition&gt;内。一个状态内部的这三个元素的执行顺序是如果退出了当前状态：&lt;onentry&gt;&lt;onexit&gt;&lt;transition&gt;，：如果没有退出当前状态&lt;onentry&gt;,&lt;transition1&gt;,&lt;transition2&gt;,&lt;transition3&gt;,&lt;onexit&gt;,&lt;transition4&gt;

可执行内容，有如下的元素：
&lt;raise&gt;,&lt;send&gt;,&lt;log&gt;,&lt;script&gt;,&lt;assign&gt;,&lt;if&gt;,&lt;foreach&gt;
另外SCXML的实现者可以自由扩展有哪些可执行内容。
