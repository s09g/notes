## 1. 架构图
在看具体每个 Netty 的 Maven 项目之前，我们还是先来看看 Netty 的整体架构图。

![架构图](https://raw.githubusercontent.com/ZhangShiqiu1993/notes/master/Netty/5.Netty%20%E7%AE%80%E4%BB%8B1-%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84/assets/01.png)

+ **Core** ：核心部分，是底层的网络通用抽象和部分实现。
    + Extensible Event Model ：可拓展的事件模型。Netty 是基于事件模型的网络应用框架。
    + Universal Communication API ：通用的通信 API 层。Netty 定义了一套抽象的通用通信层的 API 。
    + Zero-Copy-Capable Rich Byte Buffer ：支持零拷贝特性的 Byte Buffer 实现。
+ **Transport Services** ：传输( 通信 )服务，具体的网络传输的定义与实现。
Socket & Datagram ：TCP 和 UDP 的传输实现。
HTTP Tunnel ：HTTP 通道的传输实现。
In-VM Piple ：JVM 内部的传输实现。
Protocol Support ：协议支持。Netty 对于一些通用协议的编解码实现。例如：HTTP、Redis、DNS 等等。

## 2. 项目依赖图
Netty 的 Maven 项目之间主要依赖如下图：

![依赖图](https://raw.githubusercontent.com/ZhangShiqiu1993/notes/master/Netty/5.Netty%20%E7%AE%80%E4%BB%8B1-%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84/assets/02.png)

+ 本图省略**非主要依赖**。例如，`handler-proxy` 对 `codec` 有依赖，但是并未画出。
+ 本图省略**非主要的项目**。例如，`resolver`、`testsuite`、`example` 等等。


下面，我们来详细介绍每个项目。

## 3. common
`common` 项目，该项目是一个通用的工具类项目，几乎被所有的其它项目依赖使用，它提供了一些数据类型处理工具类，并发编程以及多线程的扩展，计数器等等通用的工具类。

![common 项目](https://raw.githubusercontent.com/ZhangShiqiu1993/notes/master/Netty/5.Netty%20%E7%AE%80%E4%BB%8B1-%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84/assets/03.png)

## 4. buffer
>该项目实现了 Netty 架构图中的 Zero-Copy-Capable Rich Byte Buffer 。

`buffer` 项目，该项目下是 Netty 自行实现的一个 Byte Buffer 字节缓冲区。该包的实现相对于 JDK 自带的 ByteBuffer 有很多优点：无论是 API 的功能，使用体验，性能都要更加优秀。它提供了 **一系列(多种)** 的抽象定义以及实现，以满足不同场景下的需要。

![buffer 项目](https://raw.githubusercontent.com/ZhangShiqiu1993/notes/master/Netty/5.Netty%20%E7%AE%80%E4%BB%8B1-%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84/assets/04.png)

## 5. transport
>该项是核心项目，实现了 Netty 架构图中 Transport Services、Universal Communication API 和 Extensible Event Model 等多部分内容。

`transport` 项目，该项目是网络传输通道的抽象和实现。它定义通信的统一通信 API ，统一了 JDK 的 OIO、NIO ( 不包括 AIO )等多种编程接口。

![transport 项目](https://raw.githubusercontent.com/ZhangShiqiu1993/notes/master/Netty/5.Netty%20%E7%AE%80%E4%BB%8B1-%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84/assets/05.png)

另外，它提供了多个子项目，实现不同的传输类型。例如：`transport-native-epoll`、`transport-native-kqueue`、`transport-rxtx`、`transport-udt` 和 `transport-sctp` 等等。

## 6. codec
>该项目实现了Netty 架构图中的 Protocol Support 。

`codec` 项目，该项目是协议编解码的抽象与部分实现：JSON、Google Protocol、Base64、XML 等等。

![codec 项目](https://raw.githubusercontent.com/ZhangShiqiu1993/notes/master/Netty/5.Netty%20%E7%AE%80%E4%BB%8B1-%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84/assets/06.png)

另外，它提供了多个子项目，实现不同协议的编解码。例如：`codec-dns`、`codec-haproxy`、`codec-http`、`codec-http2`、`codec-mqtt`、`codec-redis`、`codec-memcached`、`codec-smtp`、`codec-socks`、`codec-stomp`、`codec-xml` 等等。

## 7. handler
handler 项目，该项目是提供**内置的**连接通道处理器( ChannelHandler )实现类。例如：SSL 处理器、日志处理器等等。

![handler 项目](https://raw.githubusercontent.com/ZhangShiqiu1993/notes/master/Netty/5.Netty%20%E7%AE%80%E4%BB%8B1-%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84/assets/07.png)

另外，它提供了一个子项目 `handler-proxy` ，实现对 HTTP、Socks 4、Socks 5 的代理转发。

## 8. example
`example` 项目，该项目是提供各种 Netty 使用示例，良心开源项目。

![example 项目](https://raw.githubusercontent.com/ZhangShiqiu1993/notes/master/Netty/5.Netty%20%E7%AE%80%E4%BB%8B1-%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84/assets/08.png)
