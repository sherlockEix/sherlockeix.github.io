---
title: netty-1
date: 2021-01-12 13:10:00
tags:
- netty
- java
---


### 什么是Netty
[Netty](https://netty.io/) 是一个开源的网络编程框架，Netty 封装了 Java 的 NIO 和 OIO，简化了java网络编程。可以让开发者快速的上手网络编程
由于 NIO 使用 Reactor 模式来设计，可以高效的处理网络请求。而 Netty 是对 NIO 的进一步封装和简化，所以 Netty 也是 Reactor 模式。

> Reactor 是一种广泛应用在服务器端开发的设计模式。是为了节省 CPU 的性能，不让 CPU 因为读取网络 IO 里面的数据而阻塞。一般的实现方式是注册一个回调
的实例(Handler)，然后由一个单独的线程不断的去检查 IO 操作是否就，如果就绪之后就会通知对应的 Handler 来处理。

### 最简单的一个例子
`本例子中会使用netty实现可以接收请求并且响应消息的服务端`

#### 创建服务端
- 启动类

```java
package com.java.netty.simple.demo;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import lombok.extern.slf4j.Slf4j;

/**
 * 服务端
 */
@Slf4j
public class SimpleServer {

    private static final int PORT = 9999;

    public static void main(String[] args) throws InterruptedException {
        // 创建监听线程，这个线程池会用来监听连接到客户端的连接，
        NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup();
        // 创建工作线程，用来处理具体的请求
        NioEventLoopGroup workEventLoopGroup = new NioEventLoopGroup();
        // 创建服务端的引导
        ServerBootstrap serverBootstrap = new ServerBootstrap();
        ChannelFuture sync = serverBootstrap.group(nioEventLoopGroup, workEventLoopGroup)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        ch.pipeline().addLast(new SimpleServerHandler());
                    }
                })
                .bind(PORT).addListener(future -> {
                    if (future.isSuccess()) {
                        log.info("端口{}绑定成功", PORT);
                    }
                }).sync();
        sync.channel().closeFuture().sync();
    }
}
```
- 服务端处理类

```java
package com.java.netty.simple.demo;

import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import lombok.extern.slf4j.Slf4j;

import static com.java.netty.Constants.UTF_8;

/**
 * 服务端处理
 */
@Slf4j
public class SimpleServerHandler extends SimpleChannelInboundHandler {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {
        // 在不加管道的情况下，默认的对象都是ByteBuf
        ByteBuf byteBuf = (ByteBuf) msg;
        log.info("服务器接收到:{}", byteBuf.toString(UTF_8));
    }
}
```
- 测试服务是否正常
  使用`telnet`命令可以测试一个端口是否有应用活动，运行`main`方法，结果如下
  
{% asset_img 2018-11-14-netty-server-1.png 应用启动 %}
{% asset_img 2018-11-14-netty-server-2.png telnet测试 %}
{% asset_img 2018-11-14-netty-server-3.png 服务器结果 %}
可以看到，服务端已经接收到了我们输入的内容。这就是使用netty创建的一个最简单的服务端,只需要十几行代码就可以实现网络编程的基础功能。

源码地址: [netty-demo](https://github.com/g5niusx/netty-demo/tree/master/src/main/java/com/java/netty/simple)
