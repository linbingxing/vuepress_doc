#  Netty详解

## 1.Netty是什么？

Netty 是一款用于高效开发网络应用的 NIO 网络框架，它大大简化了网络应用的开发过程。提供了对TCP、UDP和文件传输的支持。

## 2. Netty 特点是什么？ 

- 高并发：Netty 是一款基于 NIO（Nonblocking IO，非阻塞IO）开发的网络通 信框架，对比于 BIO（Blocking I/O，阻塞IO），他的并发性能得到了很大提高。 
- 传输快：Netty 的传输依赖于零拷贝特性，尽量减少不必要的内存拷贝，实现了 更高效率的传输。 
- 封装好：Netty 封装了 NIO 操作的很多细节，提供了易于使用调用接口。 

## 3. Netty 的优势有哪些？ 

- 使用简单：封装了 NIO 的很多细节，使用更简单。 
- 功能强大：预置了多种编解码功能，支持多种主流协议。 
- 定制能力强：可以通过 ChannelHandler 对通信框架进行灵活地扩展。 
- 性能高：通过与其他业界主流的 NIO 框架对比，Netty 的综合性能优。 
- 稳定：Netty 修复了已经发现的所有 NIO 的 bug，让开发人员可以专注于业务 本身。 
- 社区活跃：Netty 是活跃的开源项目，版本迭代周期短，bug 修复速度快。

## 4. Netty 的应用场景有哪些？ 

典型的应用有：阿里分布式服务框架 Dubbo，默认使用 Netty 作为基础通信组 件，还有 RocketMQ 也是使用 Netty 作为通讯的基础。 

## 5. Netty 高性能表现在哪些方面？ 

- IO 线程模型：同步非阻塞，用少的资源做更多的事。 
- 内存零拷贝：尽量减少不必要的内存拷贝，实现了更高效率的传输。 
- 内存池设计：申请的内存可以重用，主要指直接内存。内部实现是用一颗二叉查 找树管理内存分配情况。 
- 串形化处理读写：避免使用锁带来的性能开销。
- 高性能序列化协议：支持 protobuf 等高性能序列化协议。 

## 6. Netty的线程模型？

Netty采用Reactor模型，基于I/O多路复用器接收并处理用户请求，内部实现了两个线程池，Boss线程池和Work线程池，其中Boss线程池的线程负责处理请求的accept事件，当接收到accept事件的请求时，把对应的socket封装到一个NioSockerChannel中，并交给Work线程池，其中Work线程池负责请求的read和write事件，由对应的Handler处理。 在Netty中，EventLoopGroup 是 Netty Reactor 线程模型的具体实现方式，Netty 通过创建不同的 EventLoopGroup 参数配置，就可以支持 Reactor 的三种线程模型：

- 单线程模型：EventLoopGroup 只包含一个 EventLoop，Boss 和 Worker 使用同一个EventLoopGroup；

- 多线程模型：EventLoopGroup 包含多个 EventLoop，Boss 和 Worker 使用同一个EventLoopGroup；

- 主从多线程模型：EventLoopGroup 包含多个 EventLoop，Boss 是主 Reactor，Worker 是从 Reactor，它们分别使用不同的 EventLoopGroup，主 Reactor 负责新的网络连接 Channel 创建，然后把 Channel 注册到从 Reactor。

## 7.Netty 整体结构、 逻辑架构

1. **整体结构**

![netty整体架构](https://gitee.com/linbingxing/image/raw/master/netty整体架构.png)

1. Core 核心层
Core 核心层是 Netty 最精华的内容，它提供了底层网络通信的通用抽象和实现，包括可扩展的事件模型、通用的通信 API、支持零拷贝的 ByteBuf 等。

2. Protocol Support 协议支持层
协议支持层基本上覆盖了主流协议的编解码实现，如 HTTP、SSL、Protobuf、压缩、大文件传输、WebSocket、文本、二进制等主流协议，此外 Netty 还支持自定义应用层协议。Netty 丰富的协议支持降低了用户的开发成本，基于 Netty 我们可以快速开发 HTTP、WebSocket 等服务。

3. Transport Service 传输服务层
传输服务层提供了网络传输能力的定义和实现方法。它支持 Socket、HTTP 隧道、虚拟机管道等传输方式。Netty 对 TCP、UDP 等数据传输做了抽象和封装，用户可以更聚焦在业务逻辑实现上，而不必关系底层数据传输的细节。

Netty 的模块设计具备较高的通用性和可扩展性，它不仅是一个优秀的网络框架，还可以作为网络编程的工具箱。Netty 的设计理念非常优雅，值得我们学习借鉴。

**逻辑架构**

Netty 的逻辑处理架构为典型网络分层架构设计，共分为网络通信层、事件调度层、服务编排层，每一层各司其职。图中包含了 Netty 每一层所用到的核心组件。我将为你介绍 Netty 的每个逻辑分层中的各个核心组件以及组件之间是如何协调运作的。

![Netty逻辑架构](https://gitee.com/linbingxing/image/raw/master/netty逻辑架构.png)

## 8.Netty 中有哪种重要组件？



## 9.什么是 Netty 的零拷贝？



## 10.Netty解决TCP 粘包/拆包方法？

  