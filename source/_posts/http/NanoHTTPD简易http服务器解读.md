title: NanoHTTPD简易http服务器解读
date: 2016-04-26 13:38:32
tags: [http,socket]
categories: [java]
---

前两天学RMI的时候，看java core上说有一个简易的HTTP服务器，只有一个类文件。就像去瞧瞧，结果发现这个类里面有20+个内部类和接口。看了源代码之后，对http服务器的理解也更加深入了。[源代码](https://github.com/NanoHttpd/nanohttpd)

# NanoHTTPD 服务器的请求流程图
![NanoHTTPD服务器请求流程的序列图](/images/NanoHTTPD.jpg)

<font color="red"> 图上3中不同颜色的箭头，分别代表三种线程。</font>>
<!--more -->

1. 首先通过继承NanoHTTPD 类，自定义一个服务器启动的main入口，接下来继承NanoHTTPD里面的serve(HttpSession)方法 实现自定义的请求处理。  

2. 服务器启动之后，会简历一个监听线程(ServerRunnable),在这个线程内部监听 ServerScoket的连接。如果有客户端连接，就为每一个连接分配一个线程去处理连接请求。在这个连接线程里面，取得inputstream，和outputstream。  

3. 然后新建HttpSession对象，调用HttpSession对象的execute()方法,  在ClientHandler里面就用循环不断的监听接受到的这个Socket关闭了没有。  

4. HttpSession对象然后根据inputstream里面的值，读取请求行，请求头，读完之后这两部分之后，就调用自定义的serve()方法调用解析请求body，

5. 解析完body之后就需要返回Response,最后利用response里面的输出流，将返回的数据输出到 当前的Socket连接里面。最后就由浏览器来解析数据了。  

6. 将输出流，socket流，clienthandler线程依次关闭。完成一个处理



# 源码分析
## NanoHTTPD.start(timeout,deamo)方法
```java
 public void start(final int timeout, boolean daemon) throws IOException {

        this.myServerSocket = this.getServerSocketFactory().create();
        //  该选项用来决定如果网络上仍然有数据向旧的ServerSocket传输数据，
        // 是否允许新的ServerSocket绑定到与旧的ServerSocket同样的端口上，
        // 该选项的默认值与操作系统有关，在某些操作系统中，允许重用端口，而在某些系统中不允许重用端口。

        //当ServerSocket关闭时，如果网络上还有发送到这个serversocket上的数据，
        // 这个ServerSocket不会立即释放本地端口，而是等待一段时间，确保接收到了网络上发送过来的延迟数据，然后再释放端口

        //值得注意的是，该方法必须在ServerSocket还没有绑定到一个本地端口之前使用，
        // 否则执行该方法无效。此外，两个公用同一个端口的进程必须都调用serverSocket.setReuseAddress(true)方法，
        // 才能使得一个进程关闭ServerSocket之后，另一个进程的ServerSocket还能够立刻重用相同的端口
        this.myServerSocket.setReuseAddress(true);//绑定端口是否可以重用


        //绑定ServerSocket,返回一个可执行的Server线程
        ServerRunnable serverRunnable = createServerRunnable(timeout);

        //将返回的ServerRunable 绑定到线程上。这个线程就是监听线程
        this.myThread = new Thread(serverRunnable);
        //是否设置设置为守护线程
        this.myThread.setDaemon(daemon);
        //设置线程名字
        this.myThread.setName("NanoHttpd Main Listener");
        //然后启动监听线程
        this.myThread.start();

        //TODO：main线程就隔一段时间判断监听线程是否出错了，出错了就跑出异常
        while (!serverRunnable.hasBinded && serverRunnable.bindException == null) {
            try {
                Thread.sleep(10L);
            } catch (Throwable e) {
                // on android this may not be allowed, that's why we
                // catch throwable the wait should be very short because we are
                // just waiting for the bind of the socket
            }
        }
        if (serverRunnable.bindException != null) {
            throw serverRunnable.bindException;
        }
    }
```

## 监听线程（ServerRunnable）类
```java
 public class ServerRunnable implements Runnable {

        private final int timeout;

        private IOException bindException;

        private boolean hasBinded = false;

        private ServerRunnable(int timeout) {
            this.timeout = timeout;
        }

        @Override
        public void run() {
            try {
                myServerSocket.bind(hostname != null ? new InetSocketAddress(hostname, myPort) : new InetSocketAddress(myPort));
                hasBinded = true;
            } catch (IOException e) {
                this.bindException = e;
                return;
            }
            do {
                try {
                    final Socket finalAccept = NanoHTTPD.this.myServerSocket.accept();
                    if (this.timeout > 0) {
                        finalAccept.setSoTimeout(this.timeout);//设置两次连接之间的，
                        // 如果ServerSocket等待超过这个时间，从accept处返回。
                    }
                    final InputStream inputStream = finalAccept.getInputStream();
                    //启动ClientHandler线程
                    NanoHTTPD.this.asyncRunner.exec(createClientHandler(finalAccept, inputStream));
                } catch (IOException e) {
                    NanoHTTPD.LOG.log(Level.FINE, "Communication with the client broken", e);
                }
            } while (!NanoHTTPD.this.myServerSocket.isClosed());
        }
    }
```

##  DefaultAsyncRunner类
做这个类就是为了提供一个可插拔的策略，可以统计当前的连接数，方便管理连接等
```java
    public static class DefaultAsyncRunner implements AsyncRunner {

        private long requestCount;

        private final List<ClientHandler> running = Collections.synchronizedList(new ArrayList<NanoHTTPD.ClientHandler>());

        /**
         * @return a list with currently running clients.
         */
        public List<ClientHandler> getRunning() {
            return running;
        }

        @Override
        public void closeAll() {
            // copy of the list for concurrency
            for (ClientHandler clientHandler : new ArrayList<ClientHandler>(this.running)) {
                clientHandler.close();
            }
        }

        //感觉源码里面，没有在这里减去requestCount
        @Override
        public void closed(ClientHandler clientHandler) {
            this.running.remove(clientHandler);
        }

        @Override
        public void exec(ClientHandler clientHandler) {
            ++this.requestCount;
            Thread t = new Thread(clientHandler);
            t.setDaemon(true);
            t.setName("NanoHttpd Request Processor (#" + this.requestCount + ")");
            this.running.add(clientHandler);
            //这里才是真正的启动客户连接处理线程
            t.start();
        }
    }
```

## ClientHandler 线程
```java
    /**
     * 对应着每一个新的客户端连接的线程,线程里面有对应的当前连接的输入流，和取得的Socket连接对象，同时还取得了输出流
     */
    public class ClientHandler implements Runnable {

        private final InputStream inputStream;

        private final Socket acceptSocket;

        private ClientHandler(InputStream inputStream, Socket acceptSocket) {
            this.inputStream = inputStream;
            this.acceptSocket = acceptSocket;
        }

        public void close() {
            safeClose(this.inputStream);
            safeClose(this.acceptSocket);
        }

        @Override
        public void run() {
            OutputStream outputStream = null;
            try {
                //得到当前连接的输出流
                outputStream = this.acceptSocket.getOutputStream();
                //创建临时文件管理器
                TempFileManager tempFileManager = NanoHTTPD.this.tempFileManagerFactory.create();
                //---------------创建了当前连接对应的  session对象，---------------线程私有的。
                HTTPSession session = new HTTPSession(tempFileManager, this.inputStream, outputStream, this.acceptSocket.getInetAddress());
                while (!this.acceptSocket.isClosed()) {
                    //调用session对象的execute方法，取得请求行，请求头等信息。
                    session.execute();
                }
            } catch (Exception e) {
                // When the socket is closed by the client,
                // we throw our own SocketException
                // to break the "keep alive" loop above. If
                // the exception was anything other
                // than the expected SocketException OR a
                // SocketTimeoutException, print the
                // stacktrace
                if (!(e instanceof SocketException && "NanoHttpd Shutdown".equals(e.getMessage())) && !(e instanceof SocketTimeoutException)) {
                    NanoHTTPD.LOG.log(Level.SEVERE, "Communication with the client broken, or an bug in the handler code", e);
                }
            } finally {
                safeClose(outputStream);
                safeClose(this.inputStream);
                safeClose(this.acceptSocket);
                NanoHTTPD.this.asyncRunner.closed(this);
            }
        }
    }
```

## httpsession.execute()方法
```java
@Override
        public void execute() throws IOException {
            Response r = null;
            try {
                // Read the first 8192 bytes. 读取第一个8192字节
                // The full header should fit in here.
                // Apache's default header limit is 8KB.
                // Do NOT assume that a single read will get the entire header
                // at once!
                byte[] buf = new byte[HTTPSession.BUFSIZE];
                this.splitbyte = 0;
                this.rlen = 0;

                int read = -1;
                this.inputStream.mark(HTTPSession.BUFSIZE);
                try {
                    //开始读取输入流里面的数据
                    read = this.inputStream.read(buf, 0, HTTPSession.BUFSIZE);
                } catch (SSLException e) {
                    throw e;
                } catch (IOException e) {
                    safeClose(this.inputStream);
                    safeClose(this.outputStream);
                    throw new SocketException("NanoHttpd Shutdown");
                }
                //上面的代码没读出来，
                if (read == -1) {
                    // socket was been closed
                    safeClose(this.inputStream);
                    safeClose(this.outputStream);
                    throw new SocketException("NanoHttpd Shutdown");
                }
                //
                while (read > 0) {
                    this.rlen += read;
                    this.splitbyte = findHeaderEnd(buf, this.rlen);//多少个字节开始，出现header和body的分隔
                    if (this.splitbyte > 0) {
                        break;
                    }
                    read = this.inputStream.read(buf, this.rlen, HTTPSession.BUFSIZE - this.rlen);
                }

                if (this.splitbyte < this.rlen) {
                    this.inputStream.reset();
                    this.inputStream.skip(this.splitbyte);
                }

                //设置parms 属性,params就是url中 ？后面跟的东西
                this.parms = new HashMap<String, String>();
                //确保header为空
                if (null == this.headers) {
                    this.headers = new HashMap<String, String>();
                } else {
                    this.headers.clear();
                }

                // C创建BufferReader解析 Header
                BufferedReader hin = new BufferedReader(new InputStreamReader(new ByteArrayInputStream(buf, 0, this.rlen)));

                // 解析头部
                Map<String, String> pre = new HashMap<String, String>();
                decodeHeader(hin, pre, this.parms, this.headers);

                //添加远程主机信息
                if (null != this.remoteIp) {
                    this.headers.put("remote-addr", this.remoteIp);
                    this.headers.put("http-client-ip", this.remoteIp);
                }

                //方法字符串转化为对象
                this.method = Method.lookup(pre.get("method"));
                //请求的方法不能为空
                if (this.method == null) {
                    throw new ResponseException(Response.Status.BAD_REQUEST, "BAD REQUEST: Syntax error. HTTP verb " + pre.get("method") + " unhandled.");
                }

                this.uri = pre.get("uri");
                //新疆一个Cookies处理器，持有头部信息
                this.cookies = new CookieHandler(this.headers);

                //判断连接类型，长连接还是短连接
                String connection = this.headers.get("connection");
                boolean keepAlive = "HTTP/1.1".equals(protocolVersion) && (connection == null || !connection.matches("(?i).*close.*"));

                // Ok, now do the serve()
                // (requires implementation for totalRead())
                // 请求行，和请求头都解析出来了，去处理业务逻辑，并且给出返回Response对象
                r = serve(this);
                // TODO: this.inputStream.skip(body_size -
                // (this.inputStream.totalRead() - pos_before_serve))

                //开始返回Response, 如果返回的响应为空
                if (r == null) {
                    throw new ResponseException(Response.Status.INTERNAL_ERROR, "SERVER INTERNAL ERROR: Serve() returned a null response.");
                } else {
                    //设置返回的响应内容的编码方式
                    String acceptEncoding = this.headers.get("accept-encoding");
                    this.cookies.unloadQueue(r);
                    r.setRequestMethod(this.method);
                    r.setGzipEncoding(useGzipWhenAccepted(r) && acceptEncoding != null && acceptEncoding.contains("gzip"));
                    r.setKeepAlive(keepAlive);
                    //之后调用response的发送方法，将数据发送出去了。
                    r.send(this.outputStream);
                }
                //如果不是长连接
                if (!keepAlive || r.isCloseConnection()) {
                    throw new SocketException("NanoHttpd Shutdown");
                }
            } catch (SocketException e) {
                // throw it out to close socket object (finalAccept)
                throw e;
            } catch (SocketTimeoutException ste) {
                // treat socket timeouts the same way we treat socket exceptions
                // i.e. close the stream & finalAccept object by throwing the
                // exception up the call stack.
                throw ste;
            } catch (SSLException ssle) {
                Response resp = newFixedLengthResponse(Response.Status.INTERNAL_ERROR, NanoHTTPD.MIME_PLAINTEXT, "SSL PROTOCOL FAILURE: " + ssle.getMessage());
                resp.send(this.outputStream);
                safeClose(this.outputStream);
            } catch (IOException ioe) {
                Response resp = newFixedLengthResponse(Response.Status.INTERNAL_ERROR, NanoHTTPD.MIME_PLAINTEXT, "SERVER INTERNAL ERROR: IOException: " + ioe.getMessage());
                resp.send(this.outputStream);
                safeClose(this.outputStream);
            } catch (ResponseException re) {
                Response resp = newFixedLengthResponse(re.getStatus(), NanoHTTPD.MIME_PLAINTEXT, re.getMessage());
                resp.send(this.outputStream);
                safeClose(this.outputStream);
            } finally {
                //最后关闭Response, 清除零时文件管理器
                safeClose(r);
                this.tempFileManager.clear();
            }
        }
```
## serve(httpsession )方法
参数里面的hettpsession其实已经经过了请求行，和请求头的解析，只需要覆盖server方法里面，在里面解析请求body，处理我们自己的业务逻辑，比如说是post形式表单，那么表单数据在请求body里面,根据表单数据内容，返回响应的response对象，

## HelloService重载的serve()方法
不过这个这个重载的方法没有调用解析请求body的方法，他只是简单的返回一个response对象
```java
 @Override
    public Response serve(IHTTPSession session) {
        Method method = session.getMethod();
        String uri = session.getUri();
        HelloServer.LOG.info(method + " '" + uri + "' ");

        String msg = "<html><body><h1>Hello server</h1>\n";
        Map<String, String> parms = session.getParms();
        if (parms.get("username") == null) {
            msg += "<form action='?' method='get'>\n" + "  <p>Your name: <input type='text' name='username'></p>\n" + "</form>\n";
        } else {
            msg += "<p>Hello, " + parms.get("username") + "!</p>";
        }

        msg += "</body></html>\n";

        return newFixedLengthResponse(msg);
    }
```

## session 的 parseBody()方法
把所有请求体里面的内容，解析好，存放在files里面
```java
@Override
        public void parseBody(Map<String, String> files) throws IOException, ResponseException {
            RandomAccessFile randomAccessFile = null;
            try {
                long size = getBodySize(); //返回请求body有多大
                ByteArrayOutputStream baos = null;
                DataOutput requestDataOutput = null;

                // Store the request in memory or a file, depending on size
                //根据文件大小，决定吧这个请求存在内存中还是文件中
                if (size < MEMORY_STORE_LIMIT) {
                    baos = new ByteArrayOutputStream();
                    requestDataOutput = new DataOutputStream(baos);
                } else {
                    //得到一个随机访问的文件
                    randomAccessFile = getTmpBucket();
                    requestDataOutput = randomAccessFile;
                }

                // Read all the body and write it to request_data_output
                byte[] buf = new byte[REQUEST_BUFFER_LEN];
                while (this.rlen >= 0 && size > 0) {
                    this.rlen = this.inputStream.read(buf, 0, (int) Math.min(size, REQUEST_BUFFER_LEN));
                    size -= this.rlen;
                    if (this.rlen > 0) {
                        requestDataOutput.write(buf, 0, this.rlen);
                    }
                }

                ByteBuffer fbuf = null;
                if (baos != null) {
                    fbuf = ByteBuffer.wrap(baos.toByteArray(), 0, baos.size());
                } else {
                    fbuf = randomAccessFile.getChannel().map(FileChannel.MapMode.READ_ONLY, 0, randomAccessFile.length());
                    randomAccessFile.seek(0);
                }

                // If the method is POST, there may be parameters
                // in data section, too, read it:
                if (Method.POST.equals(this.method)) {
                    ContentType contentType = new ContentType(this.headers.get("content-type"));
                    if (contentType.isMultipart()) {
                        String boundary = contentType.getBoundary();
                        if (boundary == null) {
                            throw new ResponseException(Response.Status.BAD_REQUEST,
                                    "BAD REQUEST: Content type is multipart/form-data but boundary missing. Usage: GET /example/file.html");
                        }
                        decodeMultipartFormData(contentType, fbuf, this.parms, files);
                    } else {
                        byte[] postBytes = new byte[fbuf.remaining()];
                        fbuf.get(postBytes);
                        String postLine = new String(postBytes, contentType.getEncoding()).trim();
                        // Handle application/x-www-form-urlencoded
                        if ("application/x-www-form-urlencoded".equalsIgnoreCase(contentType.getContentType())) {
                            decodeParms(postLine, this.parms);
                        } else if (postLine.length() != 0) {
                            // Special case for raw POST data => create a
                            // special files entry "postData" with raw content
                            // data
                            files.put("postData", postLine);
                        }
                    }
                } else if (Method.PUT.equals(this.method)) {
                    files.put("content", saveTmpFile(fbuf, 0, fbuf.limit(), null));
                }
            } finally {
                safeClose(randomAccessFile);
            }
        }
```

## 返回的response 对象的send()方法
```java
 /**
         * 发送给定的响应到Socket
         */
        protected void send(OutputStream outputStream) {
            SimpleDateFormat gmtFrmt = new SimpleDateFormat("E, d MMM yyyy HH:mm:ss 'GMT'", Locale.US);
            gmtFrmt.setTimeZone(TimeZone.getTimeZone("GMT"));

            try {
                if (this.status == null) {
                    throw new Error("sendResponse(): Status can't be null.");
                }
                PrintWriter pw = new PrintWriter(new BufferedWriter(new OutputStreamWriter(outputStream, new ContentType(this.mimeType).getEncoding())), false);
                pw.append("HTTP/1.1 ").append(this.status.getDescription()).append(" \r\n");
                if (this.mimeType != null) {
                    printHeader(pw, "Content-Type", this.mimeType);
                }
                if (getHeader("date") == null) {
                    printHeader(pw, "Date", gmtFrmt.format(new Date()));
                }
                for (Entry<String, String> entry : this.header.entrySet()) {
                    printHeader(pw, entry.getKey(), entry.getValue());
                }
                if (getHeader("connection") == null) {
                    printHeader(pw, "Connection", (this.keepAlive ? "keep-alive" : "close"));
                }
                if (getHeader("content-length") != null) {
                    encodeAsGzip = false;
                }
                if (encodeAsGzip) {
                    printHeader(pw, "Content-Encoding", "gzip");
                    setChunkedTransfer(true);
                }
                long pending = this.data != null ? this.contentLength : 0;
                if (this.requestMethod != Method.HEAD && this.chunkedTransfer) {
                    printHeader(pw, "Transfer-Encoding", "chunked");
                } else if (!encodeAsGzip) {
                    pending = sendContentLengthHeaderIfNotAlreadyPresent(pw, pending);
                }
                pw.append("\r\n");
                //像样子是想输出响应行和响应头，再输出响应body,//发送响应body
                pw.flush();
                sendBodyWithCorrectTransferAndEncoding(outputStream, pending);
                outputStream.flush();
                //客户端已经显示了传输的数据了
                safeClose(this.data);
            } catch (IOException ioe) {
                NanoHTTPD.LOG.log(Level.SEVERE, "Could not send response to the client", ioe);
            }
        }
```


接下来就是关闭流，关闭socket，关闭线程的工作了。

现在看的还比较浅显，至于里面在解析请求头，请求行，请求体的时候的注意事项，编码方式，chunkedTransfer传输都了解的不深，接下来好好看一下。
