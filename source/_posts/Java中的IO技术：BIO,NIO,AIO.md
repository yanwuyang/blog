---
title: Java中的IO技术：BIO,NIO,AIO
date: 2016-08-10
comments: true
categories: Java
toc: true
---

# 同步异步、阻塞非阻塞概念
同步和异步是针对应用程序和内核的交互而言的，阻塞和非阻塞是针对于进程在访问数据的时候，根据IO操作的就绪状态来采取的不同方式，说白了是一种读取或者写入操作函数的实现方式，阻塞方式下读取或者写入函数将一直等待，而非阻塞方式下，读取或者写入函数会立即返回一个状态值。
<!--more-->
## 同步与异步
同步是自己的事情自己做，用户进程触发IO操作，只有等待IO操作完成了以后才能干别的事情;
![同步](02.png)
``` java

InputStream in = new FileInputStream(licenseDir);
BufferedReader br = new BufferedReader(new InputStreamReader(in,"UTF-8"));
String data = null;
StringBuffer encodedData = new StringBuffer();
while ((data = br.readLine()) != null) {
	encodedData.append(data);
}
in.close();
br.close();

```
异步是自己的事情让别人做，别人做完以后通知你事情做完了，在程序中当用户进程触发了IO操作，程序将委托内核帮忙向IO中读或写数据，并传入回调函数，然后程序就可以干其他的操作，当内容操作完成调用回调函数通知用户进程事情我已经干完了。
![异步](01.png)
``` java
public class Demo1 {

    static Thread current;

    public static void main(String[] args) throws IOException {
        AsynchronousFileChannel afc = AsynchronousFileChannel.open(Paths
                .get("E:\\NettyServer\\conf\\ftpusers.properties"));
        ByteBuffer byteBuffer = ByteBuffer.allocate(16 * 1024);
        current = Thread.currentThread();
        afc.read(byteBuffer, 0, null, new CompletionHandler<Integer, Object>() {
            @Override
            public void completed(Integer result, Object attachment) {
                System.out.println("Bytes Read = " + result);
                //中断主线程挂起 
                current.interrupt();
            }
            @Override
            public void failed(Throwable exc, Object attachment) {
                System.out.println(exc.getCause());
                current.interrupt();
            }
        });
        System.out.println("Waiting for completion...");
        try {
            current.join();//挂起现成 等待 消息读取完成 
        } catch (InterruptedException e) {
        }

        afc.close();
        byteBuffer.flip();
        while(byteBuffer.hasRemaining()){ 
            System.out.print((char) byteBuffer.get()); // read 1 byte at a time 
        } 
    }

}
```
## 阻塞非阻塞
阻塞 所谓阻塞方式的意思是指, 当试图对该文件描述符进行读写时, 如果当时没有东西可读,或者暂时不可写, 程序就进入等待 状态, 直到有东西可读或者可写为止 ；
非阻塞 非阻塞状态下, 如果没有东西可读, 或者不可写, 读写函数马上返回, 而不会等待；


# BIO
在JDK1.4之前，用Java编写网络请求，都是建立一个ServerSocket，然后，客户端建立Socket时就会询问是否有线程可以处理，如果没有，要么等待，要么被拒绝。即：一个连接，要求Server对应一个处理线程。
```java
public class PlainEchoServer {
  public void serve(int port) throws IOException {
    final ServerSocket socket = new ServerSocket(port); //Bind server to port
    try {
      while (true) {
        //Block until new client connection is accepted
        final Socket clientSocket = socket.accept();
        System.out.println("Accepted connection from " + clientSocket);
        //Create new thread to handle client connection
        //KeepAlive 设置为true表示保持长连接Socket 底层会定期发送心跳包检查连接是否中断。
        clientSocket.setKeepAlive(true);
        //设置超过指定时间没有数据可读的时候read立即返回并抛出异常SocketTimeoutException
        socket.setSoTimeout(1000);
        new Thread(new Runnable() {
          @Override
          public void run() {
            try {
                while(true){
                    BufferedReader reader = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
                    PrintWriter writer = new PrintWriter(clientSocket.getOutputStream(), true);
                    //Read data from client and write it back
                    //如果此时没有数据可读将一直阻塞在read函数上除非设置socket.setSoTimeout()，
                    //超过过期时间抛出异常SocketTimeoutException
                    while (true) {
                      writer.println(reader.readLine());
                      writer.flush();
                    }
                }
            } catch (IOException e) {
              e.printStackTrace();
              try {
                clientSocket.close();
              } catch (IOException ex) {
                // ignore on close
              }
            }
          }
        }).start();
        //Start thread
      }
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
}

```
# NIO
在Java里的由来，在JDK1.4及以后版本中提供了一套API来专门操作非阻塞I/O，我们可以在java.nio包及其子包中找到相关的类和接口。由于这套API是JDK新提供的I/O API，因此，也叫New I/O，这就是包名nio的由来。这套API由三个主要的部分组成：缓冲区（Buffers）、通道（Channels）和非阻塞I/O的核心类组成。在理解NIO的时候，需要区分，说的是New I/O还是非阻塞IO，New I/O是Java的包，NIO是非阻塞IO概念。这里讲的是后面一种。

NIO本身是基于事件驱动思想来完成的，其主要想解决的是BIO的大并发问题：在使用同步I/O的网络应用中，如果要同时处理多个客户端请求，或是在客户端要同时和多个服务器进行通讯，就必须使用多线程来处理。也就是说，将每一个客户端请求分配给一个线程来单独处理。这样做虽然可以达到我们的要求，但同时又会带来另外一个问题。由于每创建一个线程，就要为这个线程分配一定的内存空间（也叫工作存储器），而且操作系统本身也对线程的总数有一定的限制。如果客户端的请求过多，服务端程序可能会因为不堪重负而拒绝客户端的请求，甚至服务器可能会因此而瘫痪。 NIO基于Selector，当socket有流可读或可写入socket时，操作系统会相应的通知引用程序进行处理，应用再将流读取到缓冲区或写入操作系统。也就是说，这个时候，已经不是一个连接就要对应一个处理线程了，而是有效的请求，对应一个线程，当连接没有数据时，是没有工作线程来处理的。
```java
public class PlainNioEchoServer {
  public void serve(int port) throws IOException {
    System.out.println("Listening for connections on port " + port);
    ServerSocketChannel serverChannel = ServerSocketChannel.open();
    ServerSocket ss = serverChannel.socket();
    InetSocketAddress address = new InetSocketAddress(port);
    //Bind server to port
    ss.bind(address);
    serverChannel.configureBlocking(false);
    Selector selector = Selector.open();
    //Register the channel with the selector to be interested in new Client connections that get accepted
    serverChannel.register(selector, SelectionKey.OP_ACCEPT);
    while (true) {
      try {
        //Block until something is selected
        selector.select();
      } catch (IOException ex) {
        ex.printStackTrace();
        //handle in a proper way
        break;
      }
      //Get all SelectedKey instances
      Set<SelectionKey> readyKeys = selector.selectedKeys();
      Iterator<SelectionKey> iterator = readyKeys.iterator();
      while (iterator.hasNext()) {
        SelectionKey key = (SelectionKey) iterator.next();
        //Remove the SelectedKey from the iterator
        iterator.remove();
        try {
          if (key.isAcceptable()) {
            ServerSocketChannel server = (ServerSocketChannel) key.channel();
            //Accept the client connection
            SocketChannel client = server.accept();
            System.out.println("Accepted connection from " + client);
            client.configureBlocking(false);
            //Register connection to selector and set ByteBuffer
            client.register(selector, SelectionKey.OP_WRITE | SelectionKey.OP_READ, ByteBuffer.allocate(100));
          }
          //Check for SelectedKey for read
          if (key.isReadable()) {
            SocketChannel client = (SocketChannel) key.channel();
            ByteBuffer output = (ByteBuffer) key.attachment();
            //Read data to ByteBuffer
            client.read(output);
          }
          //Check for SelectedKey for write
          if (key.isWritable()) {
            SocketChannel client = (SocketChannel) key.channel();
            ByteBuffer output = (ByteBuffer) key.attachment();
            output.flip();
            //Write data from ByteBuffer to channel
            client.write(output);
            output.compact();
          }
        } catch (IOException ex) {
          key.cancel();
          try {
            key.channel().close();
          } catch (IOException cex) {
          }
        }
      }
    }
  }
}

```
# AIO

当进行读写操作时，只须直接调用API的read或write方法即可。这两种方法均为异步的，对于读操作而言，当有流可读取时，操作系统会将可读的流传入read方法的缓冲区，并通知应用程序；对于写操作而言，当操作系统将write方法传递的流写入完毕时，操作系统主动通知应用程序。

即可以理解为，read/write方法都是异步的，完成后会主动调用回调函数。

在JDK1.7中，这部分内容被称作NIO.2，主要在java.nio.channels包下增加了下面四个异步通道：
•AsynchronousSocketChannel
•AsynchronousServerSocketChannel
•AsynchronousFileChannel
•AsynchronousDatagramChannel

其中的read/write方法，会返回一个带回调函数的对象，当执行完读取/写入操作后，直接调用回调函数。
```java
public class PlainNio2EchoServer {
  public void serve(int port) throws IOException {
    System.out.println("Listening for connections on port " + port);
    final AsynchronousServerSocketChannel serverChannel = AsynchronousServerSocketChannel.open();
    InetSocketAddress address = new InetSocketAddress(port);
    // Bind Server to port
    serverChannel.bind(address);
    final CountDownLatch latch = new CountDownLatch(1);
    // Start to accept new Client connections. Once one is accepted the CompletionHandler will get called.
    serverChannel.accept(null, new CompletionHandler<AsynchronousSocketChannel, Object>() {
      @Override
      public void completed(final AsynchronousSocketChannel channel, Object attachment) {
        // Again accept new Client connections
        serverChannel.accept(null, this);
        ByteBuffer buffer = ByteBuffer.allocate(100);
        // Trigger a read operation on the Channel, the given CompletionHandler will be notified once something was read
        channel.read(buffer, buffer, new EchoCompletionHandler(channel));
      }

      @Override
      public void failed(Throwable throwable, Object attachment) {
        try {
          // Close the socket on error
          serverChannel.close();
        } catch (IOException e) {
          // ingnore on close
        } finally {
          latch.countDown();
        }
      }
    });
    try {
      latch.await();
    } catch (InterruptedException e) {
      Thread.currentThread().interrupt();
    }
  }

  private final class EchoCompletionHandler implements CompletionHandler<Integer, ByteBuffer> {
    private final AsynchronousSocketChannel channel;

    EchoCompletionHandler(AsynchronousSocketChannel channel) {
      this.channel = channel;
    }

    @Override
    public void completed(Integer result, ByteBuffer buffer) {
      buffer.flip();
      // Trigger a write operation on the Channel, the given CompletionHandler will be notified once something was written
      channel.write(buffer, buffer, new CompletionHandler<Integer, ByteBuffer>() {
        @Override
        public void completed(Integer result, ByteBuffer buffer) {
          if (buffer.hasRemaining()) {
            // Trigger again a write operation if something is left in the ByteBuffer
            channel.write(buffer, buffer, this);
          } else {
            buffer.compact();
            // Trigger a read operation on the Channel, the given CompletionHandler will be notified once something was read
            channel.read(buffer, buffer, EchoCompletionHandler.this);
          }
        }

        @Override
        public void failed(Throwable exc, ByteBuffer attachment) {
          try {
            channel.close();
          } catch (IOException e) {
            // ingnore on close
          }
        }
      });
    }

    @Override
    public void failed(Throwable exc, ByteBuffer attachment) {
      try {
        channel.close();
      } catch (IOException e) {
        // ingnore on close
      }
    }
  }
}

```

# 实现原理
说到实现原理，还要从操作系统的IO模型上了解 按照《Unix网络编程》的划分，IO模型可以分为：阻塞IO、非阻塞IO、IO复用、信号驱动IO和异步IO，按照POSIX标准来划分只分为两类：同步IO和异步IO。 如何区分呢？首先一个IO操作其实分成了两个步骤：发起IO请求和实际的IO操作，同步IO和异步IO的区别就在于第二个步骤是否阻塞，如果实际的IO读写阻塞请求进程，那么就是同步IO，因此阻塞IO、非阻塞IO、IO复用、信号驱动IO都是同步IO，如果不阻塞，而是操作系统帮你做完IO操作再将结果返回给你，那么就是异步IO。阻塞IO和非阻塞IO的区别在于第一步，发起IO请求是否会被阻塞，如果阻塞直到完成那么就是传统的阻塞IO，如果不阻塞，那么就是非阻塞IO。

说到操作系统的IO模型，又不得不提select/poll/epoll/iocp。 可以理解的说明是：在Linux 2.6以后，java NIO的实现，是通过epoll来实现的，这点可以通过jdk的源代码发现。而AIO，在windows上是通过IOCP实现的，在linux上还是通过epoll来实现的。 这里强调一点：AIO，这是I/O处理模式，而epoll等都是实现AIO的一种编程模型；换句话说，AIO是一种接口标准，各家操作系统可以实现也可以不实现。在不同操作系统上在高并发情况下最好都采用操作系统推荐的方式。Linux上还没有真正实现网络方式的AIO。

# 场景分析
BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，但程序直观简单易理解。
NIO方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4开始支持。
AIO方式使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK7开始支持。
另外，I/O属于底层操作，需要操作系统支持，并发也需要操作系统的支持，所以性能方面不同操作系统差异会比较明显。
