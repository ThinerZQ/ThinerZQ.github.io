title: 'java反射基础'
date: 2016-05-01 16:38:52
tags: [反射]
categories: [java]
---
<center><h1>java 反射</h1></center>

>使用Java反射机制可以在运行时期检查Java类的信息，
检查Java类的信息往往是你在使用Java反射机制的时候所做的第一件事情，
通过获取类的信息你可以获取以下相关的内容：

1. Class对象
2. 类名
3. 修饰符
4. 包信息
5. 父类
6. 实现的接口
7. 构造器
8. 方法
9. 变量
10. 注解

<!--more-->
# Class对象
在你想检查一个类的信息之前，你首先需要获取类的Class对象。Java中的所有类型包括基本类型(int, long, float等等)，即使是数组都有与之关联的Class类的对象。

## 编译期间已知类名的获取Class对象的方法
```java
    Class myObjectClass = MyObject.class;
```
## 编译期间不知道类名的获取Class对象的方法

```java
   String className = ... ;//在运行期获取的类名字符串===全限定名字：包括包名字
   Class class = Class.forName(className);
```

<font color=red>拿到这上述的Class对象之后，就能得到对象的所有**元信息**</font>

# 使用Class对象获取信息
## 类名
```java
Class aClass = ... //获取Class对象
String className = aClass.getName();//获取到的是全限定名
String simpleClassName = aClass.getSimpleName();//获取到的是对象的名字
```

## 类的修饰符
```java
Class  aClass = ... //获
int modifiers = aClass.getModifiers();
```
修饰符都被包装成一个int类型的数字，这样每个修饰符都是一个位标识(flag bit)，这个位标识可以设置和清除修饰符的类型。
可以使用java.lang.reflect.Modifier类中的方法来检查修饰符的类型：
### Modifier类的判定方法
```java
Modifier.isAbstract(int modifiers);
Modifier.isFinal(int modifiers);
Modifier.isInterface(int modifiers);
Modifier.isNative(int modifiers);
Modifier.isPrivate(int modifiers);
Modifier.isProtected(int modifiers);
Modifier.isPublic(int modifiers);
Modifier.isStatic(int modifiers);
Modifier.isStrict(int modifiers);
Modifier.isSynchronized(int modifiers);
Modifier.isTransient(int modifiers);
Modifier.isVolatile(int modifiers);
```
### 每一个类修饰符的具体定义
```java
    public static final int PUBLIC           = 0x00000001;
    public static final int PRIVATE          = 0x00000002;
    public static final int PROTECTED        = 0x00000004;
    public static final int STATIC           = 0x00000008;
    public static final int FINAL            = 0x00000010;
    public static final int SYNCHRONIZED     = 0x00000020;
    public static final int VOLATILE         = 0x00000040;
    public static final int TRANSIENT        = 0x00000080;
    public static final int NATIVE           = 0x00000100;
    public static final int INTERFACE        = 0x00000200;
    public static final int ABSTRACT         = 0x00000400;
    public static final int STRICT           = 0x00000800;
```
## 包信息
```java
Class  aClass = ... //
Package package = aClass.getPackage();
```
通过Package对象你可以获取包的相关信息比如”包名“等

## 父类
```java
Class superclass = aClass.getSuperclass();
```
可以看到superclass对象其实就是一个Class类的实例，所以你可以继续在这个对象上进行反射操作。
## 实现的接口
```java
Class[] interfaces = aClass.getInterfaces();
```
由于一个类可以实现多个接口，因此getInterfaces();方法返回一个Class数组，在Java中接口同样有对应的Class对象。注意：getInterfaces()方法仅仅只返回当前类所实现的接口。当前类的父类如果实现了接口，这些接口是不会在返回的Class集合中的，尽管实际上当前类其实已经实现了父类接口。
## 构造器
```java
Constructor[] constructors = aClass.getConstructors();
```
## 方法
```java
Method[] method = aClass.getMethods();
```
## 成员变量
```java
Field[] method = aClass.getFields();
```
## 注解
```java
Annotation[] annotations = aClass.getAnnotations();
```


# 案例
<pre>Info.java</pre>
```java
public class Info {
    private int id=10;
    private String name="zhengqiang";

    private String getNameAndId(){
        return name+" "+id;
    }
}
```
<pre>ReflectionBasic.java</pre> 解析类前面Info.java类的类
```java
public class ReflectionBasic {

    public static void main(String[] args) throws ClassNotFoundException {

        Class infoClass= Info.class;

        System.out.println("全限定名："+infoClass.getName());
        System.out.println("类名："+infoClass.getSimpleName());

        int modifier = infoClass.getModifiers();

        System.out.println("是否时public的： "+Modifier.isPublic(modifier));

        Package packagename = infoClass.getPackage();

        System.out.println("包名："+packagename.getName());

        Method[] methods = infoClass.getDeclaredMethods();
        for (Method method :
                methods) {
            System.out.println("方法签名： "+method.getModifiers()+" " +method.getReturnType()+" "+method.getName());
        }
        Field[] fields = infoClass.getDeclaredFields();
        for (Field field :
                fields) {
            System.out.println("属性签名： "+field.getModifiers()+ " "+ field.getType() +" "+ field.getName());
        }

    }
}
```
输出结果
> 全限定名：com.zq.reflection.Info
> 类名：Info
> 是否时public的： true
> 包名：com.zq.reflection
> 方法签名： 2 class java.lang.String getNameAndId
> 属性签名： 2 int id
> 属性签名： 2 class java.lang.String name

 Process finished with exit code 0
