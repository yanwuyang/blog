---
title: Netty-4.1.4源码分析一
date: 2016-12-23
comments: true
categories: Java
toc: true 
---

## Netty是什么
- Netty是一个异步的事件驱动的网络应用程序框架用于快速开发可维护的高性能协议服务器和客户端。基于NIO客户端服务器框架，可以快速，轻松地开发网络应用程序，如协议服务器和客户端。 它大大简化和简化了网络编程，如TCP和UDP套接字服务器。
<!--more-->
![](components.png)
### 特点
- 1、设计
 - 1、针对各种传输类型的统一API - 阻止和非阻塞套接字
 - 2、基于灵活和可扩展的事件模型，允许清楚地分离关注点
 - 3、高度可定制的线程模型 - 单线程，一个或多个线程池，如SEDA
 - 4、真正的无连接数据报插座支持（自3.1版本）
- 2、使用简单
 - 1、完善的文档
 - 2、独立不依赖其他环境包JDK1.5以上就足够了
- 3、性能
 - 1、更好的吞吐量，更低的延迟
 - 2、减少资源消耗
 - 3、最小化不必要的内存复制
- 4、安全
 - 1、支持SSL / TLS和StartTLS
- 5、社区
 - 1、社区活跃。
 
上面已经对Netty有了个大概的了解了下面我们就将具体分析Netty的一个内部运行机制、以及线程模型和内存零拷贝。今天主要是分析Netty的源码对于还不会使用Netty的同学请点击查看[Netty官方文档](http://netty.io/wiki/user-guide-for-4.x.html)。

### 线程模型
![](threadmodel.png)
## 实例分析
- 
```java
public class NettyServer{
    public static void main(String[] args) throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup(); // (1)
        EventLoopGroup workerGroup = new NioEventLoopGroup();// (2)
		
        try {
            ServerBootstrap b = new ServerBootstrap(); // (3)
            b.group(bossGroup, workerGroup)
             .channel(NioServerSocketChannel.class) // (4)
             .childHandler(new ChannelInitializer<SocketChannel>() { // (5)
                 @Override
                 public void initChannel(SocketChannel ch) throws Exception {
                     ch.pipeline().addLast(new DiscardServerHandler());
                 }
             })
             .option(ChannelOption.SO_BACKLOG, 128)          // (6)
             .childOption(ChannelOption.SO_KEEPALIVE, true); // (7)

            // 绑定并开始接受传入连接。
            ChannelFuture f = b.bind(port).sync(); // (8)

            // 等待服务器套接字关闭。
            // 关闭您的服务器。
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }
}

```
第一步：实例化bossGroup NioEventLoopGroup用于接受处理来自客户端的连接请求
第二步：实例化workerGroup NioEventLoopGroup用于接受来处理自客户端的数据读写请求
第三步：实例化一个设置服务器的助手类，对netty运行环境一个基础配置
第四步：使用NioServerSocketChannel类，用于实例化新的通道以接受传入连接。
第五步：指定的处理程序将始终由新接受的通道计算 。ChannelInitializer是一个特殊的处理程序，用于帮助用户配置新的通道。很可能要通过添加一些处理程序（例如DiscardServerHandler）来配置新频道的ChannelPipeline来实现您的网络应用程序
第六步：设置通道选项参数如tcpNoDelay和keepAlive
第七步：option（）用于接受传入连接的NioServerSocketChannel， childOption（）用于父NioServerSocketChannel接受的通道的设置
第八步：绑定端口启动服务器。这里，我们绑定到机器中所有NIC（网络接口卡）的端口8080。 您现在可以根据需要多次调用bind（）方法（使用不同的绑定地址）。

**注意：ServerBootstrap的配置中带有child的如（childHandler、childOption）是针对处理数据读取的NioSocketChanel的配置 ，不带的（如option）是针对NioServerSocketChannel的配置**

## 源码分析
NioEventLoopGroup是处理I/O操作的多线程事件循环。Netty为不同的类型传输提供了各种EventLoopGroup实现。new NioEventLoopGroup()实例化了一个多线程的事件执行器。线程池大小可以通过参数指定。不指定的情况下默认是cpu核数的2倍
```java
public abstract class MultithreadEventLoopGroup extends MultithreadEventExecutorGroup implements EventLoopGroup {
    private static final int DEFAULT_EVENT_LOOP_THREADS;
    static {
        //计算默认线程池大小
        DEFAULT_EVENT_LOOP_THREADS = Math.max(1, SystemPropertyUtil.getInt(
                "io.netty.eventLoopThreads", Runtime.getRuntime().availableProcessors() * 2));
    }
	
    protected MultithreadEventLoopGroup(int nThreads, Executor executor, EventExecutorChooserFactory chooserFactory,Object... args) {
        super(nThreads == 0 ? DEFAULT_EVENT_LOOP_THREADS : nThreads, executor, chooserFactory, args);
    }
}
public abstract class MultithreadEventExecutorGroup extends AbstractEventExecutorGroup {
    
    protected MultithreadEventExecutorGroup(int nThreads, Executor executor,EventExecutorChooserFactory chooserFactory, Object... args) {
        if (executor == null) {
            executor = new ThreadPerTaskExecutor(newDefaultThreadFactory());
        }
        children = new EventExecutor[nThreads];
        //创建EventLoop
        for (int i = 0; i < nThreads; i ++) {
            boolean success = false;
            try {
                children[i] = newChild(executor, args);
                success = true;
            } catch (Exception e) {
                // TODO: Think about if this is a good exception type
                throw new IllegalStateException("failed to create a child event loop", e);
            } finally {
			     ...
            }   
        }
        //将事件执行器数组放入到选择器中
        chooser = chooserFactory.newChooser(children);
        ...
    }

}

public class NioEventLoopGroup extends MultithreadEventLoopGroup {
    //创建EventLoop
    @Override
    protected EventLoop newChild(Executor executor, Object... args) throws Exception {
        return new NioEventLoop(this, executor, (SelectorProvider) args[0],
            ((SelectStrategyFactory) args[1]).newSelectStrategy(), (RejectedExecutionHandler) args[2]);
    }

}

```
NioEventLoop是具体的事件循环类。通过SelectorProvider.openSelector()获得一个Selector。在run函数中循环的等待事件的发生
```java
public final class NioEventLoop extends SingleThreadEventLoop {
    private Selector openSelector() {
        final Selector selector;
        try {
            selector = provider.openSelector();
        }
        ...
    }
	
	@Override
    protected void run() {
        for (;;) {
            try {
                switch (selectStrategy.calculateStrategy(selectNowSupplier, hasTasks())) {
				
            }
            ...
            processSelectedKeys();
        }
    }
    private void processSelectedKeys() {
        if (selectedKeys != null) {
            processSelectedKeysOptimized(selectedKeys.flip());
        } else {
            processSelectedKeysPlain(selector.selectedKeys());
        }
    }
    private void processSelectedKeysOptimized(SelectionKey[] selectedKeys) {
        for (int i = 0;; i ++) {
            final SelectionKey k = selectedKeys[i];
            final Object a = k.attachment();
			k.attachment 获取在Channel register的时候设置的att附件对象
            if (a instanceof AbstractNioChannel) {
                processSelectedKey(k, (AbstractNioChannel) a);
            }
        }
    }
    private void processSelectedKey(SelectionKey k, AbstractNioChannel ch) {
        final AbstractNioChannel.NioUnsafe unsafe = ch.unsafe();
        ...
        if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
            unsafe.read();
        }
        ...
    }
}

```

Server端bind（绑定端口）从bossGroup NioEventLoopGroup线程池中按顺序从0开始获取一个NioEventLoop，如果已经超过了线程大小将又从0开始。
```java
public class ServerBootstrap extends AbstractBootstrap{
    Channel的管道配置当有OP_ACCEPT事件发生将调用管道的initChannel方法对新的Channel进行初始化并分别I/O线程
    void init(Channel channel){
        ChannelPipeline p = channel.pipeline();
        p.addLast(new ChannelInitializer<Channel>() {
            @Override
            public void initChannel(Channel ch) throws Exception {
                final ChannelPipeline pipeline = ch.pipeline();
                ChannelHandler handler = config.handler();
                if (handler != null) {
                    pipeline.addLast(handler);
                }
                ch.eventLoop().execute(new Runnable() {
                    @Override
                    public void run() {
                        pipeline.addLast(new ServerBootstrapAcceptor(
                        currentChildGroup, currentChildHandler, currentChildOptions, currentChildAttrs));
                    }
                });
            }
        });
    }
}

public abstract class AbstractBootstrap{
    private ChannelFuture doBind(final SocketAddress localAddress) {
        //初始并注册
        final ChannelFuture regFuture = initAndRegister();
    }

    final ChannelFuture initAndRegister() {
        Channel channel = channelFactory.newChannel();
        //初始化通道NioServerSocketChannel
        init(channel);
        //在Select中注册channel并添加OP_ACCEPT事件
        ChannelFuture regFuture = config().group().register(channel);
    }
}

public abstract class MultithreadEventLoopGroup{
    @Override
    public ChannelFuture register(Channel channel) {
        return next().register(channel);
    }

    @Override
    public EventLoop next() {
        return (EventLoop) super.next();
    }
	//调用选择管理器的next方法返回一个EventExecutor
    @Override
    public EventExecutor next() {
        return chooser.next();
    }
}

private static final class GenericEventExecutorChooser{
    //选择管理器next方法中通过idx.getAndIncrement()原子函数记录已经获取的次数通过%（摩）计算下标并返回EventExecutor
    @Override
    public EventExecutor next() {
        return executors[Math.abs(idx.getAndIncrement() % executors.length)];
    }
}

public abstract class AbstractNioChannel extends AbstractChannel {
    //在selector注册了Channel并且添加了OP_ACCEPT事件将this（NioMessageUnsafe的实例）作为附件att设置到selector上可通过selector.attachment()获取
    protected void doRegister() throws Exception {
        selectionKey = javaChannel().register(eventLoop().selector, 0, this);
    }
}

```
NioEventLoop是一个单线程的事件循环器（Selector），用户接受客服端的所有请求，当有新的客户端来连接Netty服务端接受到连接请求调用
NioServerSocketChannel内部类NioMessageUnsafe的read方法读取消息，
```java
private final class NioMessageUnsafe extends AbstractNioUnsafe {
    @Override
    public void read() {
        assert eventLoop().inEventLoop();
        final ChannelConfig config = config();
        final ChannelPipeline pipeline = pipeline();
        final RecvByteBufAllocator.Handle allocHandle = unsafe().recvBufAllocHandle();
        allocHandle.reset(config);
        boolean closed = false;
        Throwable exception = null;
        try {
            try {
                do {
                    int localRead = doReadMessages(readBuf);
                    if (localRead == 0) {
                        break;
                    }
                    if (localRead < 0) {
                        closed = true;
                        break;
                    }
                    allocHandle.incMessagesRead(localRead);
                } while (allocHandle.continueReading());
            } catch (Throwable t) {
                exception = t;
            }
            int size = readBuf.size();
            for (int i = 0; i < size; i ++) {
                readPending = false;
                pipeline.fireChannelRead(readBuf.get(i));
            }
            readBuf.clear();
            allocHandle.readComplete();
            pipeline.fireChannelReadComplete();
            ...
        } finally {
            ...
        }
    }
}
```
在doReadMessages方法中通过SocketChannel ch = javaChannel().accept();接受到新的连接SocketChannel，并实例化NioSocketChannel(this, ch)。
```java
protected int doReadMessages(List<Object> buf) throws Exception {
    SocketChannel ch = javaChannel().accept();
    try {
        if (ch != null) {
            buf.add(new NioSocketChannel(this, ch));
            return 1;
        }
    } catch (Throwable t) {
        logger.warn("Failed to create a new channel from an accepted socket.", t);
        try {
            ch.close();
        } catch (Throwable t2) {
            logger.warn("Failed to close a socket.", t2);
        }
    }
    return 0;
}
```
准备好上下文数据。通过pipeline.fireChannelReadComplete();发起了对NioServerSocketChannel管道ChannelInitializer initChannel方法的调用。注意Netty不管是针对handler 还是childHandler都是采用了管道设计模式。创建ServerBootstrapAcceptor用于处理新的连接并分配workGroup线程。
childGroup就是NioEventLoopGroup的实例对象workGroup
```java
private static class ServerBootstrapAcceptor extends ChannelInboundHandlerAdapter {
    //读取数据为新的连接分配线程并且注册OP_READ事件
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        final Channel child = (Channel) msg;
        child.pipeline().addLast(childHandler);
        childGroup.register(child).addListener(new ChannelFutureListener() {
            ... 
        }
    }
}
```
childChannel对应的数据读取类为NioByteUnsafe，
```java
protected class NioByteUnsafe extends AbstractNioUnsafe {
    public final void read() {
        final ChannelConfig config = config();
        final ChannelPipeline pipeline = pipeline();
        final ByteBufAllocator allocator = config.getAllocator();
        final RecvByteBufAllocator.Handle allocHandle = recvBufAllocHandle();
        allocHandle.reset(config);
        ....
        do {
            byteBuf = allocHandle.allocate(allocator);
            allocHandle.lastBytesRead(doReadBytes(byteBuf));
            if (allocHandle.lastBytesRead() <= 0) {
                // nothing was read. release the buffer.
                byteBuf.release();
                byteBuf = null;
                close = allocHandle.lastBytesRead() < 0;
                break;
            }

            allocHandle.incMessagesRead(1);
            readPending = false;
            pipeline.fireChannelRead(byteBuf);
            byteBuf = null;
        } while (allocHandle.continueReading());

        allocHandle.readComplete();
        pipeline.fireChannelReadComplete();
        ...
    }		
}
```


