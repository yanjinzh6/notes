---
title: NIO 简介
date: 2019-12-21 20:28:54
tags: 'Java'
categories:
  - ['开发', 'Java', 'IO']
permalink: java-nio
---

## NIO

### NIO 的新特性

java 中的 IO 和 NIO 的区别主要有 3 点:

1. IO 是面向流的, NIO 是面向缓冲的;
2. IO 是阻塞的, NIO 是非阻塞的;
3. IO 是单线程的, NIO 是通过选择器来模拟多线程的;

NIO 在基础的 IO 流上发展出新的特点, 分别是: 内存映射技术, 字符及编码, 非阻塞 I/O 和文件锁定

<!-- more -->

#### 内存映射

这个功能主要是为了提高大文件的读写速度而设计的. 内存映射文件(memory-mappedfile)能让你创建和修改那些大到无法读入内存的文件.

NIO 中内存映射主要用到以下两个类:

- java.nio.MappedByteBuffer
- java.nio.channels.FileChannel

#### 字符及编码

##### 字符编码方案

编码方案定义了如何把字符编码的序列表达为字节序列. 字符编码的数值不需要与编码字节相同, 也不需要是一对一或一对多个的关系. 原则上, 把字符集编码和解码近似视为对象的序列化和反序列化.

##### 字符集编码器和解码器

NIO 中提供了两个类 CharsetEncoder 和 CharsetDecoder 来实现编码转换方案.

#### 非阻塞 IO

一般来说 I/O 模型可以分为: 同步阻塞, 同步非阻塞, 异步阻塞, 异步非阻塞 四种 IO 模型.

NIO 的非阻塞 I/O 机制是围绕 选择器和 通道构建的. Channel 类表示服务器和客户机之间的一种通信机制. Selector 类是 Channel 的多路复用器. Selector 类将传入客户机请求多路分用并将它们分派到各自的请求处理程序. NIO 设计背后的基石是反应器(Reactor)设计模式.

Reactor 负责 IO 事件的响应, 一旦有事件发生, 便广播发送给相应的 handler 去处理. 而 NIO 的设计则是完全按照 Reactor 模式来设计的. Selector 发现某个 channel 有数据时, 会通过 SelectorKey 来告知, 然后实现事件和 handler 的绑定.

在 Reactor 模式中, 包含如下角色:

- Reactor 将 I/O 事件发派给对应的 Handler
- Acceptor 处理客户端连接请求
- Handlers 执行非阻塞读/写

多路复用 IO 避免了线程的阻塞, 提高了连接的数量, 一个线程就可以管理多个 socket, 只有当 socket 真正有读写事件发生才会占用资源来进行实际的读写操作.
多线程+ 阻塞 IO, 每个 socket 对应一个线程, 这样会造成很大的资源占用, 并且尤其是对于长连接来说, 线程的资源一直不会释放, 如果后面陆续有很多连接的话, 就会造成性能上的瓶颈.
非阻塞 IO 不断地询问 socket 状态时通过用户线程去进行的, 而在多路复用 IO 中, 轮询每个 socket 状态是内核在进行的, 这个效率要比用户线程要高的多.

#### 文件锁定

NIO 中的文件通道 (FileChannel) 在读写数据的时候主 要使用了阻塞模式, 它不能支持非阻塞模式的读写, 而且 FileChannel 的对象是不能够直接实例化的, 他的实例只能通过 getChannel()从一个打开的文件对象上边读取 (RandomAccessFile,  FileInputStream, FileOutputStream) , 并且通过调用 getChannel()方法返回一个 Channel 对象去连接同一个文件, 也就是针对同一个文件进行读写操作.

文件锁的出现解决了很多 Java 应用程序和非 Java 程序之间共享文件数据的问题

### 读数据和写数据方式

- 从通道进行数据读取 : 创建一个缓冲区, 然后请求通道读取数据.
- 从通道进行数据写入 : 创建一个缓冲区, 填充数据, 并要求通道写入数据.

### NIO 核心组件

- Channels
- Buffers
- Selectors

#### Buffer(缓冲区)

##### 介绍

Java NIO Buffers 用于和 NIO Channel 交互. 从 Channel 中读取数据到 buffers 里, 从 Buffer 把数据写入到 Channels.
Buffer 本质上就是一块内存区, 可以用来写入数据, 并在稍后读取出来. 这块内存被 NIO Buffer 包裹起来, 对外提供一系列的读写方便开发的接口.

###### 读写数据

1. 把数据写入 buffer;
1. 调用 flip;
1. 从 Buffer 中读取数据;
1. 调用 buffer.clear()或者 buffer.compact().

当写入数据到 buffer 中时, buffer 会记录已经写入的数据大小. 当需要读数据时, 通过 flip() 方法把 buffer 从写模式调整为读模式; 在读模式下, 可以读取所有已经写入的数据.

当读取完数据后, 需要清空 buffer, 以满足后续写入操作. 清空 buffer 有两种方式: 调用 clear() 或 compact() 方法. clear 会清空整个 buffer, compact 则只清空已读取的数据, 未被读取的数据会被移动到 buffer 的开始位置, 写入位置则近跟着未读数据之后.

###### 属性

- capacity 容量
- position 位置
- limit 限制

###### 常见方法

- Buffer clear()
- Buffer flip()
- Buffer rewind()
- Buffer position(int newPosition)

###### 使用方式/方法介绍

分配缓冲区 (Allocating a Buffer) :

```java
ByteBuffer buf = ByteBuffer.allocate(28);//以ByteBuffer为例子
```

写入数据到缓冲区 (Writing Data to a Buffer)

####### 写数据到 Buffer 有两种方法

1. 从 Channel 中写数据到 Buffer, int bytesRead = inChannel.read(buf); //read into buffer.
2. 通过 put 写数据, buf.put(127);

#### Channel (通道)

1. Channel (通道) 介绍

- 通常来说 NIO 中的所有 IO 都是从 Channel (通道) 开始的.
- NIO Channel 通道和流的区别：

1. FileChannel 的使用
1. SocketChannel 和 ServerSocketChannel 的使用
   ️1. DatagramChannel 的使用
1. Scatter / Gather

- Scatter: 从一个 Channel 读取的信息分散到 N 个缓冲区中(Buufer).
- Gather: 将 N 个 Buffer 里面内容按照顺序发送到一个 Channel.

1. 通道之间的数据传输

- 在 Java NIO 中如果一个 channel 是 FileChannel 类型的, 那么他可以直接把数据传输到另一个 channel.
- transferFrom() :transferFrom 方法把数据从通道源传输到 FileChannel
- transferTo() :transferTo 方法把 FileChannel 数据传输到另一个 channel

#### Selector (选择器)

1. Selector (选择器) 介绍

- Selector 一般称 为选择器 , 当然你也可以翻译为 多路复用器 . 它是 Java NIO 核心组件中的一个, 用于检查一个或多个 NIO Channel (通道) 的状态是否处于可读, 可写. 如此可以实现单线程管理多个 channels,也就是可以管理多个网络链接.
- 使用 Selector 的好处在于： 使用更少的线程来就可以来处理通道了, 相比使用多个线程, 避免了线程上下文切换带来的开销.

1. Selector (选择器) 的使用方法介绍

- Selector 的创建

```java
Selector selector = Selector.open();
```

- 注册 Channel 到 Selector(Channel 必须是非阻塞的)

```java
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, Selectionkey.OP_READ);
```

- SelectionKey 介绍

一个 SelectionKey 键表示了一个特定的通道对象和一个特定的选择器对象之间的注册关系.

- 从 Selector 中选择 channel(Selecting Channels via a Selector)

选择器维护注册过的通道的集合, 并且这种注册关系都被封装在 SelectionKey 当中.

- 停止选择的方法

wakeup()方法 和 close()方法.

1. 模板代码

有了模板代码我们在编写程序时, 大多数时间都是在模板代码中添加相应的业务代码.
