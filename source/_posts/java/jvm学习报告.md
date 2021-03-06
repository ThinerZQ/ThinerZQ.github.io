title: Jvm学习
date: 2016-07-11 09:08:06
tags: [jvm]
categories: [java]
---
# 内存布局

1. 程序计数器（Program Counter Register）：程序计数器是一个比较小的内存区域，用于指示当前线程所执行的字节码执行到了第几行，可以理解为是当前线程的行号指示器。字节码解释器在工作时，会通过改变这个计数器的值来取下一条语句指令。每个程序计数器只用来记录一个线程的行号，所以它是线程私有（一个线程就有一个程序计数器）的。
2. 虚拟机栈（JVM Stack）：一个线程的每个方法在执行的同时，都会创建一个栈帧（Statck Frame），栈帧中存储的有局部变量表、操作站、动态链接、方法出口等，当方法被调用时，栈帧在JVM栈中入栈，当方法执行完成时，栈帧出栈。局部变量表中存储着方法的相关局部变量，包括各种基本数据类型，对象的引用，返回地址等。在局部变量表中，只有long和double类型会占用2个局部变量空间（Slot，对于32位机器，一个Slot就是32个bit），其它都是1个Slot。局部变量表是在编译时就已经确定好的，方法运行所需要分配的空间在栈帧中是完全确定的，在方法的生命周期内都不会改变。每个线程对应着一个虚拟机栈，因此虚拟机栈也是线程私有的。
3. 本地方法栈（Native Method Statck）：本地方法栈在作用，运行机制，异常类型等方面都与虚拟机栈相同，唯一的区别是：虚拟机栈是执行Java方法的，而本地方法栈是用来执行native方法的，HotSpot会将本地方法栈与虚拟机栈放在一起使用。本地方法栈也是线程私有的。
4. 堆区（Heap）：在JVM所管理的内存中，堆区是最大的一块，堆区也是Java GC机制所管理的主要内存区域，堆区由所有线程共享，在虚拟机启动时创建。堆区的存在是为了存储对象实例，原则上讲，所有的对象都在堆区上分配内存（基于逃逸分析，没有逃逸出方法体的对象，没有必要再堆上分配）。

5. 方法区（Method Area）：方法区是各个线程共享的区域，用于存储已经被虚拟机加载的类信息（即加载类时需要加载的信息，包括版本、field、方法、接口等信息）、final常量、静态变量、编译器即时编译的代码等。

<!-- more -->

# 内存回收

## 垃圾对象的判断

Java 堆中存放着几乎所有的对象实例，垃圾收集器对堆中的对象进行回收前，要先确定这些对象是否还有用，有用与否又和引用的强弱有关。java将引用分为强引用（Strong Reference）、软引用（Soft Reference）、弱引用（Weak Reference）、虚引用（Phantom Reference）四种，引用强度依次减弱。

- 强引用：如“Object obj = new Object（）”，这类引用是 Java 程序中最普遍的。只要强引用还存在，垃圾收集器就永远不会回收掉被引用的对象。
- 软引用：SoftReference，它用来描述一些可能还有用，但并非必须的对象。在系统内存不够用时，这类引用关联的对象将被垃圾收集器回收。
- 弱引用：WeakReference，它也是用来描述非需对象的，但它的强度比软引用更弱些，被弱引用关联的对象只能生存岛下一次垃圾收集发生之前。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。
- 虚引用：PhantomReference，最弱的一种引用关系，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的是希望能在这个对象被收集器回收时收到一个系统通知。

对象是否有用还和是否有对象还在继续引用它，将来还会不会被使用有关。判定对象是否为垃圾对象有如下算法：

**引用计数法**

给对象添加一个引用计数器，每当有一个地方引用它时，计数器值就加 1，当引用失效时，计数器值就减1，任何时刻计数器都为 0 的对象就是不可能再被使用的。

引用计数算法的实现简单，判定效率也很高，在大部分情况下它都是一个不错的选择，当 Java 语言并没有选择这种算法来进行垃圾回收，主要原因是它很难解决对象之间的相互循环引用问题。

**根搜索法**
Java 采用根搜索算法来判定对象是否存活的。这种算法的基本思路是通过一系列名为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链，当一个对象到 GC Roots 没有任何引用链相连时，就证明此对象是不可用的。在 Java 语言里，可作为 GC Roots 的对象包括下面几种：

- 虚拟机栈（栈帧中的本地变量表）中引用的对象。
- 方法区中的类静态属性引用的对象。
- 方法区中的常量引用的对象。
- 本地方法栈中 JNI（Native 方法）的引用对象。

Java 中的垃圾回收一般是在 Java 堆中进行，因为堆中几乎存放了 Java 中所有的对象实例。

## 收集算法
当前商业虚拟机的垃圾收集 都采用分代收集，它根据对象的存活周期的不同将内存划分为几块，一般是把 Java 堆分为新生代和老年代。在新生代中，每次垃圾收集时都会发现有大量对象死去，只有少量存活，因此可选用复制算法来完成收集，而老年代中因为对象存活率高、没有额外空间对它进行分配担保，就必须使用标记—清除算法或标记—整理算法来进行回收。

### 复制算法
复制算法比较适合于新生代，将存活下来的少部分对象对象复制到另外一块空间。浪费空间
### 标记—清除算法
标记—清除算法是最基础的收集算法，它分为“标记”和“清除”两个阶段：首先标记出所需回收的对象，在标记完成后统一回收掉所有被标记的对象，它的标记过程其实就是前面的根搜索算法中判定垃圾对象的标记过程。有内存碎片
### 标记—整理算法
该算法标记的过程与标记—清除算法中的标记过程一样，但对标记后出的垃圾对象的处理情况有所不同，它不是直接对可回收对象进行清理，而是让所有的对象都向一端移动，然后直接清理掉端边界以外的内存。成本较高，无内存碎片。


## 可用的GC

### New Generation 可用的GC
- Serial: 这个收集器是一个单线程的收集器，但它的“单线程”的意义并不仅仅说明它只会使用一个CPU或一条收集线程去完成垃圾收集工作，更重要的是在它进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集结束。Serial收集器是虚拟机运行在Client模式下的默认新生代收集器。

- Parallel Scavenge：它也是使用复制算法的收集器，又是并行的多线程收集器。Parallel Scavenge收集器的目标则是达到一个可控制的吞吐量（Throughput）。自适应（会动态调整SurvivorRatio的大小,也可以固定).

- ParNew：ParNew收集器其实就是Serial收集器的多线程版本，除了使用多条线程进行垃圾收集之外，其余行为包括Serial收集器可用的所有控制参数、收集算法、Stop The World、对象分配规则、回收策略等都与Serial收集器完全一样。ParNew收集器是许多运行在Server模式下的虚拟机中首选的新生代收集器。配合使用老年代的CMS

### Old Generation 可用的GC

- Serial Old : Serial Old是Serial收集器的老年代版本，它同样是一个单线程收集器，使用标记－整理算法。Serial Old收集器的主要意义也是在于给Client模式下的虚拟机使用。

- Parallel Old：Parallel Old是Parallel Scavenge收集器的老年代版本，使用多线程和“标记－整理”算法。

- CMS：CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器。CMS收集器是基于“标记—清除”算法实现的，它的运作过程相对于前面几种收集器来说更复杂一些，整个过程分为4个步骤：
  - 初始标记（CMS initial mark）：初始标记仅仅只是标记一下GC Roots能直接关联到的对象，速度很快，需要“Stop The World”。
  - 并发标记（CMS concurrent mark）：并发标记阶段就是进行GC Roots Tracing的过程。
  - 重新标记（CMS remark）：重新标记阶段是为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段稍长一些，但远比并发标记的时间短，仍然需要“Stop The World”。
  - 并发清除（CMS concurrent sweep）：并发清除阶段会清除对象。

  由于整个过程中耗时最长的并发标记和并发清除过程收集器线程都可以与用户线程一起工作，所以，从总体上来说，CMS收集器的内存回收过程是与用户线程一起并发执行的。另外CMS收集器无法处理浮动垃圾，可能出现“Concurrent Mode Failure”失败而导致另一次Full GC的产生，同时也会产生大量的空间碎片

- G1：G1（Garbage-First）是一款面向服务端应用的垃圾收集器。G1收集器的运作大致可划分为以下几个步骤：
  - 初始标记（Initial Marking）：初始标记阶段仅仅只是标记一下GC Roots能直接关联到的对象，并且修改TAMS（Next Top at Mark Start）的值，让下一阶段用户程序并发运行时，能在正确可用的Region中创建新对象，这阶段需要停顿线程，但耗时很短。
  - 并发标记（Concurrent Marking）：并发标记阶段是从GC Root开始对堆中对象进行可达性分析，找出存活的对象，这阶段耗时较长，但可与用户程序并发执行。
  - 最终标记（Final Marking）：最终标记阶段是为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程Remembered Set Logs里面。最终标记阶段需要把Remembered Set Logs的数据合并到Remembered Set中，这阶段需要停顿线程，但是可并行执行。
  - 筛选回收（Live Data Counting and Evacuation）：筛选回收阶段首先对各个Region的回收价值和成本进行排序，根据用户所期望的GC停顿时间来制定回收计划，这个阶段其实也可以做到与用户程序一起并发执行，但是因为只回收一部分Region，时间是用户可控制的，而且停顿用户线程将大幅提高收集效率。

# JVM内存状况查看方法和工具分析

1. 输出GC日志：指定jvm启动参数，输出到控制台:-XX:PrintGC，-XX:PrintGCDetail。输出到指定的文件：-Xloggc:gc.log
2. GC Protal：输入GC日志文件，产生图形化的报表，部署麻烦，需要老版本jdk
3. JConsole：以图形化的方式查看JVM中内存你的变化状况
4. JVisualVM：查看内存的消耗状况，线程的执行状况以及程序中消耗cpu,内存的动作
5. JMap：JMap -heap pid 查看进程堆情况，JMap-histo pid 查看堆中对象的纤细占用情况，JMap -dump:format=b,file=文件名 pid：导出整个jvm的内存信息。
6. JHat：分析jvm dump的工具，和JMap搭配使用，JHat -J-Xmx2014M filename
7. JStat ：JStat -gcutil pid interval_time , 以一定的频率查看各代的占用情况

# jvm 代码编译与执行
Java 虚拟机屏蔽了与具体操作系统平台相关的信息,使得 Java 语言编译程序只需生成在 Java 虚拟机上运行的目标代码(字节码),就可以在多种平台上不加修改地运行。Java 虚拟机在执行字节码时,实际上最终还是把字节码解释成具体平台上的机器指令执行。


Java 代码编译和执行的整个过程包含了以下三个重要的机制：
- Java源码编译机制
- 类加载机制
- 类执行机制

## 类加载机制
类从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期包括：加载、验证、准备、解析、初始化五个阶段。在这五个阶段中，加载、验证、准备和初始化这四个阶段发生的顺序是确定的，而解析阶段则不一定，它在某些情况下可以在初始化阶段之后开始，这是为了支持 Java 语言的运行时绑定（运行时根据具体对象的类型进行绑定，ava 当中的方法只有 final，static，private 和构造方法是前期绑定的）。

### 加载
主要是找到class文件,并加载到jvm。通过类的全限定吗+对应的classloader一起标识一个加载的类。数组元素对应类型由classloader加载，数组类由jvm直接创建。在 Java 堆中生成一个代表这个类的 java.lang.Class 对象，作为对方法区中class文件元数据的访问入口。

### 链接

#### 验证
验证的目的是为了确保 Class 文件中的字节流包含的信息符合当前虚拟机的要求，而且不会危害虚拟机自身的安全（掌握了class文件的格式可以篡改class文件危害虚拟机）。主要包括如下四方面：
- 文件格式的验证：验证字节流是否符合 Class 文件格式的规范，并且能被当前版本的虚拟机处理（低版本不能处理高版本的），该验证的主要目的是保证输入的字节流能正确地解析并存储于方法区之内。
- 元数据的验证：其实就是对类中的各数据类型进行语法校验，保证不存在不符合 Java 语法规范的元数据信息。
- 字节码验证：该阶段验证的主要工作是进行数据流和控制流分析，对类的方法体进行校验分析，以保证被校验的类的方法在运行时不会做出危害虚拟机安全的行为。
- 符号引用验证：它发生在虚拟机将符号引用转化为直接引用的时候，主要是对类自身以外的信息（常量池中的各种符号引用）进行匹配性的校验。

#### 准备
初始化类中的静态成员变量，赋默认值，对static final 的常量赋指定的值

#### 解析
解析阶段是虚拟机将常量池中的符号引用（就是各种class中所关联的其他类的名字，需要调用的方法，字段名字等）转化为直接引用（对应在内存中的的目标地址）的过程。虚拟机会根据需要（是否动态绑定，方法的修饰符等）来判断，到底是在类被加载器加载时就对常量池中的符号引用进行解析（初始化之前），还是等到一个符号引用将要被使用前才去解析它（初始化之后）。主要包括下面的几种解析


- 类或接口的解析：判断所要转化成的直接引用是对数组类型，还是普通的对象类型的引用，从而进行不同的解析。
- 字段解析：对字段进行解析时，会先在本类中查找是否包含有简单名称和字段描述符都与目标相匹配的字段，如果有，则查找结束；如果没有，则会按照继承关系从上往下递归搜索该类所实现的各个接口和它们的父接口，还没有，则按照继承关系从上往下递归搜索其父类，直至查找结束。
- 类方法解析：对类方法的解析与对字段解析的搜索步骤差不多，解析方法的时候先搜索父类，再搜索接口。
- 接口方法解析：递归向上搜索父接口就行了。

### 初始化

当一个Java类第一次被真正使用到的时候，JVM会进行该类的初始化操作。初始化过程的主要操作是执行静态代码块和初始化静态域。在一个类被初始化之前，它的直接父类也需要被初始化。但是，一个接口的初始化，不会引起其父接口的初始化。在初始化的时候，会按照源代码中从上到下的顺序依次执行静态代码块和初始化静态域。

虚拟机会保证一个类(不是对象)的<cinit>方法在多线程环境中被正确地加锁和同步，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的<cinit>方法，其他线程都需要阻塞等待，直到活动线程执行初始化方法完毕。如果在一个类的<cinit>方法中有耗时很长的操作，那就可能造成多个线程阻塞。有一种常见的单例模式就是这么实现的。

### ClassLoader

#### Bootstrap ClassLoader
C++实现，
它负责加载存放在JDK\jre\lib下，或被-Xbootclasspath参数指定的路径中的，并且能被虚拟机识别的类库（如 rt.jar，所有的java.\*开头的类均被 Bootstrap ClassLoader 加载）。启动类加载器是无法被 Java 程序直接引用的。可以通过sun.misc.Launcher.getBootstrapClassPath().getURLs() 或者System.getProperty("sun.boot.class.path");  查看加载了哪些jar文件

#### Extension ClassLoader
ExtClassLoader
载JDK\jre\lib\ext目录的所有jar，或者由 java.ext.dirs 系统变量指定的路径中的所有类库（如javax.\*开头的类），

#### System ClassLoader
AppClassLoader,
加载用户类路径（ClassPath）所指定的类，开发者可以直接使用该类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

#### 自定义类加载器
继承java.lang.ClassLoader
重写父类的findClass(String 类的全限定名)方法
JDK已经在loadClass方法中帮我们实现了ClassLoader搜索类的算法，当在loadClass方法中搜索不到类时，loadClass方法就会调用findClass方法来搜索类。

#### 双亲委派模型

如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类。这种方法确保了系统类的正确加载

## 类执行机制

### 解释执行
jvm有自己的指令系统并使用于操作数栈的结构来解释执行指令。主要使用如下四种指令来执行一个方法
- invokestatic: static()方法
- invokevirtual：实例方法
- invokeinterface：接口方法
- invokespecial：private和<init>方法
- invokedynamic：jdk7中加入的动态语言支持

#### 栈顶缓存
基于操作数栈的执行方式，很多操作都要讲值放入操作数栈，这导致了寄存器需要不断和类存交换数据，使用栈顶缓存将本来位于栈顶的值直接缓存在寄存器上。对于大部分只需要一个值的操作而言，无需将数据放入操作数栈，可直接在寄存器上计算，然后放回操作数栈。
#### 部分栈帧共享

每个方法对应一个栈帧，当方法调用的时候，通常会传入参数到到另一个方法，而这个方法还位于前一个方法的操作数栈上。因此将前一个方法的操作数栈和当前方法的局部变量共享，节约了数据拷贝带来的开销。

### 编译执行
解释执行的效率比较低，为了提升代码性能，虚拟机将直接将大量循环执行代码编译为机器码（JIT）执行。根据硬件环境的不同，虚拟机会使用不同的编译方法，主要分为Client,Server模式。

#### Client
只做了少量性能开销比较高的优化，主要针对方法块进行优化，适用于桌面交互式应用，虚拟机启动较快。
##### 方法内联
把多个方法之间的调用，全部糅合到一个方法里面，消除了参数传递，返回值传递以及栈帧的开销。
##### 去虚拟化
在装载class文件之后，进行类层次分析，如果发现类种的方法只提供了一个实现类，那么对于调用此方法的代码，也可以进行内联（有一个接口，只有一个实现类，使用了调用接口中方法的地方直接替换为接口实现类的方法）。
##### 冗余消除
在编译时候，根据运行时状况进行代码折叠或消除，比如static final 类型是静态常量值，编译的时候就能确定，所有在根据这个常量做一些判断的地方都直接削处了冗余的代码。
#### Server
采用了大量的编译优化技巧来进行优化，更多的在于全局的优化，适用于服务端的应用，启动较慢。主要基于逃逸分析实现。
##### 标量替换
用标量替换聚合量，如果创建的实体对象并未用到其中的全部变量，则将对象展开取出各个属性值单独定义，以后直接使用属性值。这样节省了一定的内存，而且无需去找对象的引用，也更快了。
##### 栈上分配
如果一个对象的新建是在一个方法里面，并且没有把这个方法的引用传出方法外面，那么这个对象就直接在栈上分配，好处是更加快速，而且随着方法的结束，对象也被回收了。
##### 同步消除
如果发现同步的对象未逃逸，那就没有同步的必要的，直接去掉同步。
