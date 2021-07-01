

参考：

* https://bbs.gameres.com/thread_842984_1_1.html
* [linux内核之：深度理解 epoll 本质](https://www.cnblogs.com/yougewe/articles/12355610.html)





# IO 模型

IO模型就是说用什么样的通道进行数据的发送和接收，Java共支持3种网络编程IO模式：**BIO，NIO，AIO**  

中间件底层模型都是  Client <-------> Server



## IO历史

要解决服务器可以支持多个客户端连接



最早期，因为客户端非常少，采用方案是为每个客户端连接，都 fork 一个进程/线程去接收并且处理请求



1983年，发明「**IO多路复用**」的模型，这种模型的好处就是「**没必要开那么多条线程和进程了**」，一个线程/进程就搞定了。对应模型是遍历全部已注册的文件描述符，来找到那个需要处理信息的文件描述符，如果已经注册了几万个文件描述符，那会因为遍历这些已经注册的文件描述符，导致cpu爆炸。具体实现是 **select 函数**



2002年，数以千万计的请求在全世界范围内发来发去，服务器大爆炸，人们通过改进「**IO多路复用**」模型，进一步的优化，发明了一个叫**epoll的方法**，底层是**单线程的事件驱动**。

缺点是

* 1）单线程因任何原因发生阻塞，都会使得这个模型不如多线程；
* 2）单线程模型不能很好的利用多核cpu；
* 3）一旦使用多线程会出现==回调==

解决

* 利用多进程，每个进程单条线程去利用多核CPU。**但是这又引入了新的问题：进程间状态共享和通信**。但是对于提升的性能来说，可以忽略不计。
* 协程





## IO多路复用





# BIO模型

**代码 1.0**

```java
public class SocketServer {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(9000);
        while (true) {
            System.out.println("等待连接");
            // 阻塞方法，等待客户端的连接
            Socket clientSocket = serverSocket.accept();
            System.out.println("有客户端连接了..");
            handler(clientSocket);
        }
    }

    private static void handler(Socket clientSocket) throws IOException {
        byte[] bytes = new byte[1024];
        System.out.println("准备read..");
        // 阻塞方法：等待客户发送数据
        int read = clientSocket.getInputStream().read(bytes);
        System.out.println("read done..");
        if (read != -1) {
            System.out.println("接收客户端的数据：" + new String(bytes,0,read));
        }
    }
}
```



客户端测试：

`telnet localhost 9000`



代码1.0解释：主线程阻塞，一次只能处理一个客户端连接



**代码 1.1**

改造 main 方法的 handler

```java
public static void main(String[] args) throws IOException {
    ServerSocket serverSocket = new ServerSocket(9000);
    while (true) {
        System.out.println("等待连接");
        // 阻塞方法，等待客户端的连接
        Socket clientSocket = serverSocket.accept();
        System.out.println("有客户端连接了..");
        new Thread(()->{
            try {
                handler(clientSocket);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```



代码1.1解释：主线程可以循环接收客户端连接，然后通过线程异步执行。



## BIO问题

==BIO 不适用于高并发场景==：典型的 C10K，C10M 场景（C 表示 connect 连接，10K 表示 1万个连接；10M 表示 1千万个连接）



方案一：使用千万个线程

* 线程占内存的，导致内存不够用 OOM
* 频繁切换上下文，性能不高



方案二：一定数量的线程池

由于是BIO，很多线程可能阻塞中等待资源，线程的利用率不高，不能满足高并发的用户连接。





# NIO模型

同步非阻塞，服务器实现模式为**一个线程可以处理多个请求(连接)**，客户端发送的连接请求都会注册到**多路复用器selector**上，多路复用
器轮询到连接有IO请求就进行处理，JDK1.4开始引入。



**应用场景：**

NIO方式适用于连接数目多且连接比较短（轻操作） 的架构， 比如聊天服务器， 弹幕系统， 服务器间通讯，编程比较复杂  



**代码 1.0**

```java
public class NioServer {
    static List<SocketChannel> channelList = new ArrayList<>();

    public static void main(String[] args) throws IOException {
        // 创建 NIO ServerSocketChannel 与 BIO 的 serverSocket 类似
        // 获取serverSocker
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.socket().bind(new InetSocketAddress(9000));
        // 设置ServerSocketChannel为非阻塞
        serverSocketChannel.configureBlocking(false);
        System.out.println("服务启动成功");
        while (true) {
            // 非阻塞式 accept 方法不会阻塞
            // 获取 clientSocket
            SocketChannel socketChannel = serverSocketChannel.accept();
            if (socketChannel != null) {
                System.out.println("连接成功");
                // 设置客户端SocketChannel 为非阻塞
                socketChannel.configureBlocking(false);
                // 保存客户端连接在List中
                channelList.add(socketChannel);
            }
            // 遍历连接进行数据读取
            Iterator<SocketChannel> iterator = channelList.iterator();
            while (iterator.hasNext()) {
                SocketChannel sc = iterator.next();
                ByteBuffer byteBuffer = ByteBuffer.allocate(128);
                // 非阻塞模式 read 不会阻塞
                int len = sc.read(byteBuffer);
                if (len > 0) {
                    System.out.println("接收到消息：" + new String(byteBuffer.array()));
                } else if (len == -1) { // 如果客户端断开，把socket从集合中去掉
                    iterator.remove();
                    System.out.println("客户端断开连接");
                }
            }
        }

    }
}
```



代码1.0 解释：NIO 单线程（主线程），每秒可以处理上万连接。原理与 redis 类似，redis 做了很多优化。



代码1.0 问题：

1. ？？？单线程模型下，忌讳一次读取的数量非常大，redis 也同样如此。

2. 假设有10万的线程，只有少数几个线程会读取，那么大量无法退出一直在空耗



==代码1.0思考==

针对问题2，如果我们能区分会发生读写的连接，只遍历有效的连接，那么程序的性能就能提升



**代码1.1**

```java
public class NioSelectorServer {
    public static void main(String[] args) throws IOException {
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.socket().bind(new InetSocketAddress(9000));
        serverSocketChannel.configureBlocking(false);
        // 打开Selector处理Channel，即创建epoll
        Selector selector = Selector.open();
        // 把ServerSocketChannel注册到selector上，并且selector对客户端accept连接操作感兴趣
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        System.out.println("服务启动成功");

        while (true) {
            // 阻塞等待需要处理的事件发生
            selector.select();
            // 获取selector中注册的全部事件的 SelectionKey 实例
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            Iterator<SelectionKey> iterator = selectionKeys.iterator();
            // 遍历SelectionKey对事件进行处理
            while (iterator.hasNext()) {
                SelectionKey key = iterator.next();
                // 如果是 OP_ACCEPT 事件，则进行连接获取和事件注册
                if (key.isAcceptable()) {
                    ServerSocketChannel server = (ServerSocketChannel) key.channel();
                    SocketChannel socketChannel = server.accept();
                    socketChannel.configureBlocking(false);
                    // 这里只注册了读事件，如果需要给客户端发送数据可以注册写操作
                    socketChannel.register(selector, SelectionKey.OP_READ);
                    System.out.println("客户端连接成功");
                } else if (key.isReadable()) { // 如果是 OP_READ事件，则进行读取和打印
                    SocketChannel socketChannel = (SocketChannel) key.channel();
                    ByteBuffer byteBuffer = ByteBuffer.allocate(128);
                    int len = socketChannel.read(byteBuffer);
                    // 如果有数据，把数据打印出来
                    if (len > 0) {
                        System.out.println("接收到消息：" + new String(byteBuffer.array()));
                    } else if (len == -1) { // 如果客户端断开，把socket从集合中去掉
                        iterator.remove();
                        System.out.println("客户端断开连接");
                    }
                }
                // 从事件集合里删除本次处理的 key，防止下次select重复处理
                iterator.remove();
            }
        }
    }
}
```



代码1.1 解释：增加了 Selector 多路复用器，将服务端和客户端都注册到 Selector 中，并指定感兴趣的 IO 事件，优化了程序性能







## Selector IO事件

SelectionKey 中定义了 IO 事件类型

* OP_WRITE
* OP_READ
* OP_CONNECT
* OP_ACCEPT

Selector 注册感兴趣的 IO 类型，并监听事件的发生



## NIO 事件模型

NIO 有三大核心组件： **Channel(通道)， Buffer(缓冲区)，Selector(多路复用器)**

1、channel 类似于流，每个 channel 对应一个 buffer缓冲区，buffer 底层就是个数组
2、channel 会注册到 selector 上，由 selector 根据 channel 读写事件的发生将其交由某个空闲的线程处理
3、NIO 的 Buffer 和 channel 都是既可以读也可以写  



![image-20210626100823028](Java%20IO.assets/image-20210626100823028.png)





硬件接收数据会触发操作系统的中断程序，中断会执行我们的事件回调





## NIO底层 Epoll 实现

涉及 3 个 native 函数，聚焦在 Linux 下的实现



EPollSelectorImpl 文件

理解文件描述符



native 方法（通过 JNI ）

* epollCreate()  创建实例，一种c实现的数据存储结构

* epollCtl
* epollWait



arraypoll  - channel

event - 

wait 监听





I/O多路复用底层主要用的Linux 内核函数（select，poll，epoll）来实现，windows不支持epoll实现，windows底层是基于winsock2的
select函数实现的(不开源)  

|          | select（JDK 1.4）                              | poll                                           | epoll（JDK 1.5）                                             |
| -------- | ---------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| 操作方式 | 遍历                                           | 遍历                                           | 回调                                                         |
| 底层实现 | 数组                                           | 链表                                           | 哈希表                                                       |
| IO效率   | 每次调用都进行线性遍历，<br />时间复杂度为O(n) | 每次调用都进行线性遍历，<br />时间复杂度为O(n) | 事件通知方式，每当有IO事件就绪，<br />系统注册的回调函数就会被调用，时间复杂度O(1) |
| 最大连接 | 有上限（1024）                                 | 无上限                                         | 无上限                                                       |







### Redis线程模型 

Redis就是典型的基于epoll的NIO线程模型(nginx也是)，epoll实例收集所有事件(连接与读写事件)，由一个服务端线程连续处理所有事件
命令。  



调用 epoll 操作系统函数来实现的 epoll_create、epoll_ctl、epoll_wait 。





 







### Netty 线程模型 















# AIO(NIO 2.0)

异步非阻塞， 由操作系统完成后回调，通知服务端程序启动线程去处理， 一般适用于连接数较多且连接时间较长的应用

**应用场景**：
AIO方式适用于连接数目多且连接比较长(重操作)的架构，JDK7 开始支持  











**BIO、 NIO、 AIO 对比：**

|          | BIO      | NIO                    | AIO        |
| -------- | -------- | ---------------------- | ---------- |
| IO模型   | 同步阻塞 | 同步非阻塞（多路复用） | 异步非阻塞 |
| 编程难度 | 简单     | 复杂                   | 复杂       |
| 可靠性   | 差       | 好                     | 好         |
| 吞吐量   | 低       | 高                     | 高         |





**为什么Netty使用NIO而不是AIO？**

在Linux系统上，AIO的底层实现仍使用Epoll，没有很好实现AIO，因此在性能上没有明显的优势，而且被JDK封装了一层不容易深度优
化，Linux上AIO还不够成熟。==Netty是异步非阻塞框架，Netty在NIO上做了很多异步的封装==。  











# 同步、异步、阻塞、非阻塞

**同步异步与阻塞非阻塞**(段子)
老张爱喝茶，废话不说，煮开水。
出场人物：老张，水壶两把（普通水壶，简称水壶；会响的水壶，简称响水壶）。

1 老张把水壶放到火上，立等水开。==（同步阻塞）==
老张觉得自己有点傻

2 老张把水壶放到火上，去客厅看电视，时不时去厨房看看水开没有。==（同步非阻塞）==
老张还是觉得自己有点傻，于是变高端了，买了把会响笛的那种水壶。水开之后，能大声发出嘀~~~~的噪音。

3 老张把响水壶放到火上，立等水开。==（异步阻塞）==
老张觉得这样傻等意义不大

4 老张把响水壶放到火上，去客厅看电视，水壶响之前不再去看它了，响了再去拿壶。==（异步非阻塞）==
老张觉得自己聪明了。



> 同步、异步

所谓同步异步，只是对于水壶而言。
普通水壶，同步；响水壶，异步。
虽然都能干活，但响水壶可以在自己完工之后，提示老张水开了。这是普通水壶所不能及的。
同步只能让调用者去轮询自己（情况2中），造成老张效率的低下。



> 阻塞、非阻塞

所谓阻塞非阻塞，仅仅对于老张而言。
立等的老张，阻塞；看电视的老张，非阻塞  





# Epoll

epoll 是linux 实现的







## 为什么性能好？



## 应用场景







# 协程







