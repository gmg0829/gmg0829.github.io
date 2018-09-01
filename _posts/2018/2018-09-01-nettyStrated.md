---
layout:     post
title:      "Netty入门"
subtitle:   " \"Netty入门\""
date:       2018-09-01 10:50:00
author:     "gmg"
header-img: "img/post/sayBye2017.jpg"
catalog: true
tags:
    - 工作
---

> " Netty入门 ”

## Netty入门
1、简介
Netty是一个异步事件驱动的网络应用程序框架用于快速开发可维护的高性能协议服务器和客户端。Netty是一个NIO客户端服务器框架，可以快速轻松地开发协议服务器和客户端等网络应用程序。 它极大地简化并简化了TCP和UDP套接字服务器等网络编程。Netty经过精心设计，具有丰富的协议，如FTP，SMTP，HTTP以及各种二进制和基于文本的传统协议。 架构图如下：  
![](
  https://gmg0829.top/assets/2018//netty-core.png) 
2、设计  
(1)适用于各种传输类型的统一API - 阻塞和非阻塞套接字  
(2)基于灵活且可扩展的事件模型，可以清晰地分离关注点  
(3)高度可定制的线程模型 - 单线程，一个或多个线程池，如SEDA  
(4)真正的无连接数据报套接字支持（自3.1起）

3、性能  
吞吐量更高，延迟更低  
减少资源消耗  
最小化不必要的内存复制  
 4、安全  
完全支持SSL/TSL

5、Netty线程模型  
netty线程模型采用“服务端监听线程”和“IO线程”分离的方式，与多线程Reactor模型类似。  
5.1 串行化设计避免线程竞争  
Netty采用了串行化设计理念，从消息的读取、编码以及后续Handler的执行，始终都由IO线程NioEventLoop负责，这就意外着整个流程不会进行线程上下文的切换，数据也不会面临被并发修改的风险，对于用户而言，甚至不需要了解Netty的线程细节。  
一个NioEventLoop聚合了一个多路复用器Selector，因此可以处理成百上千的客户端连接，Netty的处理策略是每当有一个新的客户端接入，则从NioEventLoop线程组中顺序获取一个可用的NioEventLoop，当到达数组上限之后，重新返回到0，通过这种方式，可以基本保证各个NioEventLoop的负载均衡。一个客户端连接只注册到一个NioEventLoop上，这样就避免了多个IO线程去并发操作它。   
5.2 定时任务与时间轮算法 
在Netty中，有很多功能依赖定时任务，比较典型的有两种：   
（1）客户端连接超时控制；  
（2）链路空闲检测。  
6、Reactor模型  
 - 6.1 单线程模型  

Reactor单线程模型，指的是所有的IO操作都在同一个NIO线程上面完成，NIO线程的职责如下：  

1）作为NIO服务端，接收客户端的TCP连接；

2）作为NIO客户端，向服务端发起TCP连接；

3）读取通信对端的请求或者应答消息；

4）向通信对端发送消息请求或者应答消息。  
Reactor单线程模型示意图如下所示：  
![](
  https://gmg0829.top/assets/2018//singleTHread.png)   
  - 6.2 多线程模型  

Rector多线程模型与单线程模型最大的区别就是有一组NIO线程处理IO操作。  
Reactor多线程模型的特点： 

1）有专门一个NIO线程-Acceptor线程用于监听服务端，接收客户端的TCP连接请求；

2）网络IO操作-读、写等由一个NIO线程池负责，线程池可以采用标准的JDK线程池实现，它包含一个任务队列和N个可用的线程，由这些NIO线程负责消息的读取、解码、编码和发送；

Reactor多线程模型示意图如下所示：  
![](
 https://gmg0829.top/assets/2018//multiThread.png)   
- 6.3 主从多线程模型    

主从Reactor线程模型的特点：  
服务端用于接收客户端连接的不再是个1个单独的NIO线程，而是一个独立的NIO线程池。Acceptor接收到客户端TCP连接请求处理完成后（可能包含接入认证等），将新创建的SocketChannel注册到IO线程池（sub reactor线程池）的某个IO线程上，由它负责SocketChannel的读写和编解码工作。Acceptor线程池仅仅只用于客户端的登陆、握手和安全认证，一旦链路建立成功，就将链路注册到后端subReactor线程池的IO线程上，由IO线程负责后续的IO操作。  
线程模型如下图所示： 
![](
 https://gmg0829.top/assets/2018/masterClusterThread.png)   

**参考**： 
(1) http://www.infoq.com/cn/articles/netty-threading-model  

(2) https://netty.io/
