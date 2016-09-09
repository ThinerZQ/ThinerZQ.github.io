title: Java Io学习
date: 2016-07-09 09:08:06
tags: [io]
categories: [java]
---
# Io概述

流是一组有顺序的，有起点和终点的字节集合，是对数据传输的总称或抽象。即数据在两设备间的传输称为流，流的本质是数据传输，根据数据传输介质将流抽象为各种类，方便更直观的进行数据操作。说白了这些类就是用来读取和写入数据的，由于可以读取或写入的媒介非常多，针对每一种媒介jdk提供了单独的类来读和写数据。同时根据读写单位又分为读取按字节读写和按字符读取(根据不同的编码方案,一个字符可能按多个字节读取)。大致可以分为如下

从读写介质的角度来看


|读写介质|字节输入|字节输出|字符输入|字符输出|
|:--|:--|:--|:--|:--|
|Basic|InputStream|OutputStream|Reader, InputStreamReader|Writer, OutputStreamReader|
|Arrays|ByteArrayInputStream|ByteArrayOutputStream|CharArrayReader|CharArrayWriter|
|File|FileInputStream, RandomAccessFile|FileOutputStream, RandomAccessFile|FileReader|FileWriter|
|Pipes|PipedInputStream|PipedOutputStream|PipedReader|PipedWriter|
|Buffering|BufferedInputStream|BufferedOutputStream|BufferedReader|BufferedWriter|
|Filtering|FilterInputStream|FilterOuptStream|FilterReader|FilterWriter|
|Parsing|PushbackInputStream, StreamTokenizer||PushbackReader, LineNumberReader||
|String|||StringReader|StringWriter|
|Data|DataInputStream|DataOutputStream|||
|Data-Formatterd||PrintStream||PrintWriter|
|Objects|ObjectInputStream|ObjectOutputStream|||
|Utilities|SequenceInputStream|||||
<!-- more-->
从功能和继承关系的角度看
![Java IO](/images/java_io.png)

# 基本方法

InputStream类（不是接口）提供的基本方法
```java
public abstract int read()
public int read(byte b[])
public int read(byte b[], int off, int len)
public long skip(long n)
public void close()
public synchronized void mark(int readlimit)
public synchronized void reset()
public boolean markSupported()
```
OutputStream类（不是接口）提供的基本方法
```java
public abstract void write(int b)
public void write(byte b[])
public void write(byte b[], int off, int len)
public void flush()
public void close()
```

其他InputStream和OutputStream的子类都根据各自的功能重写了上述的方法或者提供了另外的函数

RandomAccessFile类提供的
```java
 public void seek(long pos)
```

PipedInputStream类和PipedOutputStream类提供的

```java
public void connect(PipedOutputStream src)
public synchronized void connect(PipedInputStream snk)
```

DataInputStream类提供的一系列方法，DataOutputStream提供的方法也类似。

```java
public final boolean readBoolean()
public final byte readByte()
public final short readShort()
public final char readChar()
public final int readInt(){
  //in 是通过DataInputStream的构造函数传递进去的InputStream具体类的对象
  int ch1 = in.read();
  int ch2 = in.read();
  int ch3 = in.read();
  int ch4 = in.read();
  if ((ch1 | ch2 | ch3 | ch4) < 0)
      throw new EOFException();
  return ((ch1 << 24) + (ch2 << 16) + (ch3 << 8) + (ch4 << 0));
}
public final long readLong()
public final float readFloat()
public final double readDouble()
public final String readLine()
```

PushbackInputStream提供的unread()方法其实很简单，使用一个buf数组，将unread的字节放入buf，下次read的时候从buf里面读。PushbackReader原理类似

```java
public int read() throws IOException {
    ensureOpen();
    if (pos < buf.length) {
        return buf[pos++] & 0xff;
    }
    return super.read();
}
public void unread(int b) throws IOException {
    ensureOpen();
    if (pos == 0) {
        throw new IOException("Push back buffer is full");
    }
    buf[--pos] = (byte)b;
}
```

SequenceInputStream把一个或者多个InputStream整合起来，形成一个逻辑连贯的输入流。当读取SequenceInputStream时，会先从第一个输入流中读取，完成之后再从第二个输入流读取... 以此类推。程序内部将多个流放入到一个Enumeration中，在read的时候判断需不需要切换到下一个流

```java
public
class SequenceInputStream extends InputStream {
    Enumeration<? extends InputStream> e;
    InputStream in;
public SequenceInputStream(Enumeration<? extends InputStream> e) {
        this.e = e;
        try {
            nextStream();
        } catch (IOException ex) {
            // This should never happen
            throw new Error("panic");
        }
    }
    final void nextStream() throws IOException {
   if (in != null) {
       in.close();
   }

   if (e.hasMoreElements()) {
       in = (InputStream) e.nextElement();
       if (in == null)
           throw new NullPointerException();
   }
   else in = null;

}
public int read() throws IOException {
    while (in != null) {
        int c = in.read();
        if (c != -1) {
            return c;
        }
        nextStream();
    }
    return -1;
}
```

PrintStream允许你把格式化数据写入到底层OutputStream中。比如，写入格式化成文本的int，long以及其他原始数据类型到输出流中，而非它们的字节数据。通过其内部的构造方法可以看出，这个流内部其实用了Writer来写int ,long 类型的数据。

```java
private PrintStream(boolean autoFlush, OutputStream out, Charset charset) {
        super(out);
        this.autoFlush = autoFlush;
        this.charOut = new OutputStreamWriter(this, charset);
        this.textOut = new BufferedWriter(charOut);
    }
public void print(int i) {
        write(String.valueOf(i));
    }
private void write(String s) {
     try {
         synchronized (this) {
             ensureOpen();
             textOut.write(s);
             textOut.flushBuffer();
             charOut.flushBuffer();
             if (autoFlush && (s.indexOf('\n') >= 0))
                 out.flush();
         }
     }
     catch (InterruptedIOException x) {
         Thread.currentThread().interrupt();
     }
     catch (IOException x) {
         trouble = true;
     }
 }
```

从PrintWriter的最终的构造方法可以看出，PrintWriter内部只使用Wrtier来作为最终的输出流。它提供的方法和PrintStream基本一样

```java
public PrintWriter(Writer out,boolean autoFlush) {
        super(out);
        this.out = out;
        this.autoFlush = autoFlush;
        lineSeparator = java.security.AccessController.doPrivileged(
            new sun.security.action.GetPropertyAction("line.separator"));
}
public PrintWriter(OutputStream out, boolean autoFlush) {
     this(new BufferedWriter(new OutputStreamWriter(out)), autoFlush);

     // save print stream for error propagation
     if (out instanceof java.io.PrintStream) {
         psOut = (PrintStream) out;
     }
 }
 public void print(long l) {
      write(String.valueOf(l));
  }
public void write(String s, int off, int len) {
    try {
        synchronized (lock) {
            ensureOpen();
            out.write(s, off, len);
        }
    }
    catch (InterruptedIOException x) {
        Thread.currentThread().interrupt();
    }
    catch (IOException x) {
        trouble = true;
    }
}
```

LineNumberReader类通过getLineNumber()方法获取当前行号，通过setLineNumber()方法设置当前行数。流的读取依然是顺序进行，不能通过setLineNumber()实现流的跳跃读取。使用这个流可以快速定位流读取过程中出现的错误。这个流的本质就是在读取的时候使用判断读到的字符是否是行结束符，如果是就lineNumber++（文件的行结束符视操作系统而定）。这个流没有LineNumberInputStream一说。

```java
public int read() throws IOException {
        synchronized (lock) {
            int c = super.read();
            if (skipLF) {
                if (c == '\n')
                    c = super.read();
                skipLF = false;
            }
            switch (c) {
            case '\r':
                skipLF = true;
            case '\n':          /* Fall through */
                lineNumber++;
                return '\n';
            }
            return c;
        }
    }
```

StreamTokenizer通过循环调用nextToken()可以遍历底层输入流的所有符号。在每次调用nextToken()之后，StreamTokenizer有一些变量可以获取读取到的符号的类型和值。这些变量是：
1. ttype 读取到的符号的类型(字符(TT_WORD)，数字(TT_NUMBER)，或者行结尾符(TT_EOL))
2. sval 如果读取到的符号是字符串类型，该变量的值就是读取到的字符串的值
3. nval 如果读取到的符号是数字类型，该变量的值就是读取到的数字的值

以此类推，只要在需要的什么样的流的时候，使用对应的io流对象就行了。说到io，大家都会想到装饰器模式，其实装饰器模式很好理解。下面三个类就是io中一个简单的装饰器模式体现。使用io流的时候，主要注意的问题是几个对象之间能不能互相装饰。
```java
public class FilterInputStream extends InputStream {

    protected volatile InputStream in;
    protected FilterInputStream(InputStream in) {
        this.in = in;
    }
    public int read() throws IOException {
        return in.read();
    }
    public synchronized void reset() throws IOException {
        in.reset();
    }
public class BufferedInputStream extends FilterInputStream {
    public BufferedInputStream(InputStream in, int size) {
         super(in);
         buf = new byte[size];
     }
     public synchronized int read() throws IOException {
       if (pos >= count) {
           fill();
           if (pos >= count)
               return -1;
       }
       return getBufIfOpen()[pos++] & 0xff;
     }
     public synchronized void reset() throws IOException {
        getBufIfOpen(); // Cause exception if closed
        if (markpos < 0)
            throw new IOException("Resetting to invalid mark");
        pos = markpos;
    }
}
public class DataInputStream extends FilterInputStream implements DataInput{

  public DataInputStream(InputStream in) {
         super(in);
  }
  public final boolean readBoolean() throws IOException {
      int ch = in.read();
      if (ch < 0)
          throw new EOFException();
      return (ch != 0);
  }
  //没有reset(),mark()方法
}
```
如果想使用DataInputStream 同时使用BufferedInputStream的mark()方法，可以如下使用
DataInputStream dis = new DataInputStream(new BufferedInputStream(new FileInputStream("filepath")));

# 需要注意的点

1. 任何关于文件本身或者目录的操作在File类里面

2. int read()和 int read(byte[] b)方法返回值的区别
read()放回读取到的字节，read(byte[] b)返回读取到的字节数量。

3. System.out System.in. System.err 可以通过System.setErr()等重定向到任意输出流，

3. 流关闭问题的两种解决方法，一统一写一个模板类，二使用try-with-resource风格编程

4. BufferedInputStream和BufferedOutputStream缓冲区大小设置为1024整数倍效率更高

5. ObjectInputStream和ObjectOutputStream所读写的对象对应的类必须实现了java.io.Serializable接口

6. InputStreamReader和OutputStreamWriter是一种字符流，主要目的是将字节流转化成字符流，使用字符流的形式来处理数据。

7. FilterInputStream和和FilterOuFilterReader和FilterReader是过滤流的基类， 都只是简单的覆盖了其父类的方法，主要使用它们的子类来处理读写。
