



# 中间件 IO 模型

中间件底层模型都是  Client <-------> Server



# BIO模型

**代码 v1.0**

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



代码解释：主线程阻塞，一次只能处理一个客户端连接



**代码 v1.1**

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



代码解释：主线程可以循环接收客户端连接，然后通过线程异步执行。



## BIO问题

==BIO 不适用于高并发场景==：典型的 C10K，C10M 场景（C 表示 connect 连接，10K 表示 1万个连接；10M 表示 1千万个连接）



方案一：使用千万个线程

* 线程占内存的，导致内存不够用 OOM
* 频繁切换上下文，性能不高



方案二：一定数量的线程池

由于是BIO，很多线程可能阻塞中等待资源，线程的利用率不高，不能满足高并发的用户连接。





# NIO模型

NIO：Non-block IO，JDK 1.4



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



代码解释：NIO 单线程，每秒可以处理上万连接。原理与 redis 类似，redis 做了很多优化。



==思考==：单线程模型下，忌讳一次读取的数量非常大，redis 也同样如此。































