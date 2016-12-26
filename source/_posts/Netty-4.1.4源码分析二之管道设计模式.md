---
title: Netty-4.1.4源码分析二之管道设计模式
date: 2016-12-26
comments: true
categories: Java
toc: true 
---

### 管道模式的好处
- 在Netty中对数据的处理编码解码读取都是通过管道模式串连起来，这些handler都是管道中的一个阀门。对于管道模式的好处在这里简单的说下。在一个比较复杂的大型系统中，假如存在某个对象或数据流需要被进行繁杂的逻辑处理的话，我们可以选择在一个大的组件中进行这些繁杂的逻辑处理，这种方式确实达到了目的，但却是简单粗暴的。或许在某些情况这种简单粗暴的方式将带来一些麻烦，例如我要改动其中某部分处理逻辑、我要添加一些处理逻辑到流程、我要在流程中减少一些处理逻辑时，这里有些看似简单的改动都让我们无从下手，除了对整个组件进行改动。整个系统看起来没有任何可扩展性和可重用性。
是否有一种模式可以将整个处理流程进行详细划分，划分出的每个小模块互相独立且各自负责一段逻辑处理，这些逻辑处理小模块根据顺序连起来，前以模块的输出作为后一模块的输入，最后一个模块的输出为最终的处理结果。如此一来修改逻辑时只针对某个模块修改，添加或减少处理逻辑也可细化到某个模块颗粒度，并且每个模块可重复利用，可重用性大大增强。
<!--more-->
![](20150502184145168.png)

### 管道模式在Netty中的应用
- 在Netty中的管道对象就是DefaultChannelPipeline，他是Channel中的一个属性。下面我们具体来看看他的应用与实现

```java
//初始化管道对管道对管道中的通道添加处理器
@Override
protected void initChannel(SocketChannel ch) throws Exception {
    ChannelPipeline p = ch.pipeline();
    p.addLast("frameDecoder",new ProtobufVarint32FrameDecoder());//用于半包处理
    p.addLast("protobufDecoder",  new ProtobufDecoder(MessageProto.Message.getDefaultInstance()));
    p.addLast("frameEncoder", new ProtobufVarint32LengthFieldPrepender());
    p.addLast("protobufEncoder",  new ProtobufEncoder());
    p.addLast("idleState", new IdleStateHandler(connector.getIdleTime(), 0, 0));//此两项为添加心跳机制 10秒查看一次在线的客户端channel是否空闲
    p.addLast("handler", serverHandler);
}
```
DefaultChannelPipeline源码分析
```java
public class DefaultChannelPipeline implements ChannelPipeline {
    //定义了一个设置头部处理器属性
    final AbstractChannelHandlerContext head;
	//定义了一个设置尾部处理器属性
    final AbstractChannelHandlerContext tail;
	
    //向管道中添加处理器
    @Override
    public final ChannelPipeline addFirst(String name, ChannelHandler handler) {
        return addFirst(null, name, handler);
    }
	
	@Override
    public final ChannelPipeline addFirst(EventExecutorGroup group, String name, ChannelHandler handler) {
        final AbstractChannelHandlerContext newCtx;
        synchronized (this) {
            checkMultiplicity(handler);
            name = filterName(name, handler);
            newCtx = newContext(group, name, handler);
            addFirst0(newCtx);
            ...
        }
    }
    //交换处理器位置将整个处理器串连在了Pipeline上
    private void addFirst0(AbstractChannelHandlerContext newCtx) {
        AbstractChannelHandlerContext nextCtx = head.next;
        newCtx.prev = head;
        newCtx.next = nextCtx;
        head.next = newCtx;
        nextCtx.prev = newCtx;
    }
	
    @Override
    public final ChannelPipeline addLast(String name, ChannelHandler handler) {
        return addLast(null, name, handler);
    }
    @Override
    public final ChannelPipeline addLast(EventExecutorGroup group, String name, ChannelHandler handler) {
        final AbstractChannelHandlerContext newCtx;
        synchronized (this) {
            checkMultiplicity(handler);
            newCtx = newContext(group, filterName(name, handler), handler);
            addLast0(newCtx);
        }
    }
    private void addLast0(AbstractChannelHandlerContext newCtx) {
        AbstractChannelHandlerContext prev = tail.prev;
        newCtx.prev = prev;
        newCtx.next = tail;
        prev.next = newCtx;
        tail.prev = newCtx;
    }
	
    @Override
    public final ChannelPipeline fireChannelActive() {
        AbstractChannelHandlerContext.invokeChannelActive(head);
        return this;
    }

    @Override
    public final ChannelPipeline fireChannelInactive() {
        AbstractChannelHandlerContext.invokeChannelInactive(head);
        return this;
    }

    @Override
    public final ChannelPipeline fireExceptionCaught(Throwable cause) {
        AbstractChannelHandlerContext.invokeExceptionCaught(head, cause);
        return this;
    }

    @Override
    public final ChannelPipeline fireUserEventTriggered(Object event) {
        AbstractChannelHandlerContext.invokeUserEventTriggered(head, event);
        return this;
    }

    @Override
    public final ChannelPipeline fireChannelRead(Object msg) {
        AbstractChannelHandlerContext.invokeChannelRead(head, msg);
        return this;
    }

    @Override
    public final ChannelPipeline fireChannelReadComplete() {
        AbstractChannelHandlerContext.invokeChannelReadComplete(head);
        return this;
    }

    @Override
    public final ChannelPipeline fireChannelWritabilityChanged() {
        AbstractChannelHandlerContext.invokeChannelWritabilityChanged(head);
        return this;
    }
}
```
当有新的数据可读的时候channel将调用管道中的fireChannelRead() ,fireChannelReadComplete()发起了对管道中处理器的调用
```java
abstract class AbstractChannelHandlerContext extends DefaultAttributeMap implements ChannelHandlerContext, ResourceLeakHint {
    static void invokeChannelRead(final AbstractChannelHandlerContext next, Object msg) {
        final Object m = next.pipeline.touch(ObjectUtil.checkNotNull(msg, "msg"), next);
        EventExecutor executor = next.executor();
        if (executor.inEventLoop()) {
            next.invokeChannelRead(m);
        } else {
            executor.execute(new Runnable() {
                @Override
                public void run() {
                    next.invokeChannelRead(m);
                }
            });
        }
    }	
}
```
