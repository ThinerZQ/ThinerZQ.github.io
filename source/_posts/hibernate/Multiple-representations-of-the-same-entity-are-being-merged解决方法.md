title: Multiple representations of the same entity are being merged解决方法
date: 2016-02-01 13:52:20
tags: [hibernate,Multiple representation]
categories: [hibernate]
---

最近在写网站过程中发现自己hibernate学的还不到家，好多错误，特地来记录一下这个错误:  
<code>java.lang.IllegalStateException: Multiple representations of the same entity are being merged.</code>

# 错误信息
```java
 java.lang.IllegalStateException: Multiple representations of the same entity [com.sysu.workflow.entity.FormEntity#5] are being merged. Detached: [com.sysu.workflow.entity.FormEntity@79bc3f5b]; Detached: [com.sysu.workflow.entity.FormEntity@7e5533c0]
    userworkit5_.itemId as itemId1_10_3_,
at org.hibernate.event.internal.EntityCopyNotAllowedObserver.entityCopyDetected(EntityCopyNotAllowedObserver.java:51)
    userworkit5_.itemAssignee as itemAssi7_10_3_,
```
<!--more-->

#  错误原因
>因为试图给 某一个new 的Transient对象 的某一个属性赋一个 已经Persistent 对象或者Detached 对象值。导致最后save 或者merge 这个Transient对象报这个错误。



# 解决方案
这应该算是Hibernate 自身的一个bug ,已经在4.2.15版本中[解决](https://hibernate.atlassian.net/browse/HHH-9261)了
> 1、更新hibernate版本到4.2.15 以上

> 2、在hibernate的配置文件中添加如下属性：
```xml
<property name="hibernate.event.merge.entity_copy_observer">allow</property>
```
> 如果使用的Spring 管理hibernate， 在你的spring的数据源中配置
```xml
<prop key="hibernate.event.merge.entity_copy_observer">allow</prop>
```

hibernate还是有点复杂的。路漫漫其修远兮....
