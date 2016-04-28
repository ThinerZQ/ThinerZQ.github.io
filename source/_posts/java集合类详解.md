title: java集合类详解
date: 2016-04-20 12:21:48
tags: 
 java
 collection
categories:
 java
---

学了挺久java的了，但是发现对java里面的集合类没有达到融汇贯通的地步，之前都只是看别人的Blog，用一些常用的集合类，慢慢发现如果想再上一个台阶必须要自己去看源代码，自己去分析理解。

# 集合框架图
网上流传的集合框架图基本上都来自Core java这本书里面的集合类介绍，结合自己的理解，自己再画了一次。
![Java Collection](/images/java_collection.jpg)

<!--more -->
# 常用集合类的对比

|接口|实现类|元素有无顺序|元素是否重复|功能，特点|
|:---|:---|:----|:---|:---|
| List | ArrayList | 插入顺序 | 可重复 | 数组实现，查询快，增删慢，线程不安全，轻量级| 
| List | Vector | 插入顺序 | 可重复 | 数组实现，查询快，增删慢，线程安全，重量级| 
| List | LinkedList | 插入顺序 | 可重复 | 链表实现，增删快，查询慢，线程不安全|
| Set | HashSet | 无序 | 不可重复 | HashMap实现，允许存在一个null元素|
| Set | LinkedHashSet | 插入顺序 | 不可重复 | HashMap实现，允许存在一个null元素，维护一个双重链接列表|
| Set | TreeSet | 自定义排序 | 不可重复 | 默认使用元素自然顺序排序，或者自定义，通过红黑树实现排序|
| Map | HashMap | 无序 | key不能重复 | 允许一个null的key，线程不安全|
| Map | LinkedHashMap | 插入顺序 | key不能重复 | 许一个null的key，线程不安全,维护一个双重链接列表|
| Map | Hashtable | 无序 | key不能重复 | 不允许null的key或者value，线程安全|
| Map | TreeMap | 自定义key顺序 | 可重复 | 通过红黑树实现排序|

# ArrayList详解

[ArrayList详解](../java8-ArrayList详解)
# HashMap详解
[HashMap详解](../java8-HashMap详解)

