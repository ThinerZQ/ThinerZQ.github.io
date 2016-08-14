title: Java NIO学习
date: 2016-07-09 09:08:06
tags:
 nio
categories:
 java
---
# NIO概述
Java NIO: Non-blocking IO（非阻塞IO）是jdk 1.4提出的，简单的理解就是使用nio api读写数据的时候不会等待把所有数据读写完再返回(所以读写的时候应该使用while循环不断尝试读写数据)。NIO中关键的三个概念是通道，缓冲区，选择器。

<!-- more-->
## Channel(通道)
所有的数据读写在NIO 中都从Channel 开始，可以把Channel想象成io 中的Stream。数据是从Channel读到Buffer中，或者从Buffer 写到Channel中。Channel主要有以下几种
1. FileChannel: 从FileInputStream和FileOutputStream打开的文件通道
2. DatagramChannel：通过UDP协议读写网络中的数据
3. SocketChannel：通过TCP协议读写网络中的数据
4. ServerSocketChannel：对每一个连接创建一个SocketChannel

## Buffer(缓冲区)
读取数据的第一步是Channel，第二步是Buffer。数据从通道读取到Buffer中，然后再使用，或者从Buffer中写入通道。Buffer本质上是一块可以写入数据，然后可以从中读取数据的内存,这块内存被包装成NIO Buffer对象。

Buffer主要由三个属性，它们决定了每次读写缓冲区的什么位置。
1. capacity：缓冲区的大小
2. position：读模式的时候，position为下一个可以读取的位置；写模式的时候，position为下一个可以写入数据的位置
3. limit：写模式的时候，可以写入的数据的大小；读模式下，limit表示可以读取的数据大小

当向buffer写入数据时，buffer会记录下写了多少数据。一旦要读取数据，需要通过flip()方法将Buffer从写模式切换到读模式（相应的position和limit值会改变）。在读模式下，可以读取之前写入到buffer的所有数据。

一旦读完Buffer中的数据，需要让Buffer准备好再次被写入。可以通过clear()或compact()方法来完成。clear()将Buffer中剩余的数据清除，compact()会将没有读取的数据移动到Buffer起始处，之后还可以读取。

Buffer.allocate() 直接在Head上分配内存，Buffer.directAllocate()在堆外分配内存（分配的内存不受GC影响，需要自己手动释放）。


## Selector(选择器)

Selector（选择器）是NIO中能够侦听多个NIO Channel，并能够知晓这些Channel是否为特定的事件做好准备的组件。
Selector的使用主要是通过如下这种形式来使用
```java
       ServerSocketChannel ssChannel = ServerSocketChannel.open();
       ServerSocket ss = ssChannel.socket();
       InetSocketAddress address = new InetSocketAddress(port);
       ss.bind(address);

       Selector selector = Selector.open();

       ssChannel.configureBlocking(false);
       ssChannel.register(selector, SelectionKey.OP_ACCEPT); //向selector注册监听连接请求

       while (true) {
           selector.select();//阻塞 直到某个channel注册的感兴趣的事件被触发
           Set<SelectionKey> keys = selector.selectedKeys();
           for (SelectionKey key : keys) {
               if (key.isAcceptable()) { //客户端连接请求
                   ServerSocketChannel ssc = (ServerSocketChannel) key.channel();
                   SocketChannel sc = ssc.accept();
                   sc.configureBlocking(false);
                   sc.register(selector, SelectionKey.OP_READ); //向selector注册监听客户端输入
               }
               if (key.isReadable()) { //客户端输入

                   ByteBuffer buffer = ByteBuffer.allocate(1024);

                   SocketChannel socketChannel = (SocketChannel) key.channel();

                   // 针对当前连接做一些事情.read or write

               }
           }
           keys.clear();
       }
```
Selector必须和Channel配合使用，而且Channel必须处于非阻塞模式下。通过resigter()方法绑定selector和channel的时候，第二个参数指定了当前channel对什么事件感兴趣。一个channel可以有如下4种感兴趣的事件
1. SelectionKey.OP_CONNECT
2. SelectionKey.OP_ACCEPT
3. SelectionKey.OP_READ
4. SelectionKey.OP_WRITE

当channel向selector注册了事件之后，selector.selector()方法会阻塞直到channel关心的哪些事件已经就绪。事件就绪之后判断事件的类别，根据不同类别分别做不同的读写处理。


# 序列化
一个对象可以被表示为一个字节序列，该字节序列包括该对象的数据、有关对象的类型的信息和存储在对象中数据的类型。

对应的流是ObjectOutputStream，ObjectInputStream。static变量, transient不会序列化，static final会序列化。实现这两个接口的类可以序列化：Serializible ,Externalizible。

如果想自己控制一部分序列化的过程，可以在需要序列化的类种加入private级别的readObject()和writeObject()方法，这两个方法再ObjectInputStream和ObjectOutputStream中会反射调用。write和read的顺序必须一致。使用实现Externalizible接口来序列化，序列化的过程完全可控。

如果需要序列化的对象使用了Signleton模式，（序列化前的对象 == 序列化后的对象）=false,如果要使这两者相等，可以在Singleton类种加入readResolve()方法，在这个方法中返回Signleton对象替换掉在流中创建的对象。——这些都是在同一个jvm中才有用
