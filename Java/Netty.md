# Netty

## ⭐ Netty是什么
> Netty 是一个**异步的、基于事件驱动**的网络应用框架，用以开发高性能，高可靠的网络IO程序  
> 主要针对在TCP协议下，面向 Clients 端 的高并发应用  
> 本质是一个 `NIO` 框架  

### 应用场景
高性能的RPC框架 ,Hadoop的通信和序列化组件 `Avro` 默认采用Netty 进行节点通信 

### 书籍
`Netty In Action`  `Netty 权威指南`


## ⭐ IO模型
> java共支持三种 网络编程 IO模型  BIO,NIO,AIO  


> `BIO` (Blocking)同步阻塞,  
> `NIO` (Non-Blocking)同步非阻塞,使用多路复用轮询,适用于连接数目多且连接比较短,聊天服务器,弹幕,  
> `AIO` 异步非阻塞,适用于连接数目多且连接比较长  

## BIO
- 传统java IO 相关接口和类在 `java.io`
- 同步阻塞,服务器实现模式为一个连接一个线程,可以通过线程池机制改善,实现多个客户连接服务器
- BIO方式适用于连接数目小且固定的架构

### 简单流程
- 服务器端启动一个ServerSocket
- 客户端启动Socket对服务器进行通信
- 客户端发出请求后,先咨询服务器是否有线程响应,如果没有就会进入等待或者被拒绝
- 如果有响应,客户端线程会等待请求结束后,在继续运行


### 代码示例
- 服务器端启动一个`ServerSocket`
- 使用BIO 模型编写一个服务器端,监听 `6666` 端口    
- 客户端通过命令 `telnet localhost 6666` 访问  

```java
public class BioDemo {
    public static void main(String[] args) throws IOException {

        ExecutorService threadPool = Executors.newCachedThreadPool();

        ServerSocket serverSocket = new ServerSocket(6666);

        System.out.println("服务器启动");

        while (true) {
            //监听等待客户端连接
            System.out.println("等待链接");
            final Socket accept = serverSocket.accept();
            System.out.println("连接到一个客户端");

            threadPool.execute(() -> {
                handler(accept);
            });
        }
    }

    public static void handler(Socket socket) {
        try {
            System.out.println("线程信息id" + Thread.currentThread().getId() + "名称" + Thread.currentThread().getName());
            byte[] bytes = new byte[1024];
            //通过socket获取输入流
            InputStream inputStream = socket.getInputStream();
            while (true) {
                System.out.println("线程信息id" + Thread.currentThread().getId() + "名称" + Thread.currentThread().getName());
                System.out.println("等待读取read...");
                int read = inputStream.read(bytes);
                if (read != -1) {
                    //输出客户端发送的数据
                    System.out.println(new String(bytes, 0, read));
                } else {
                    break;
                }
            }

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```


### BIO缺陷
- 每个请求都需要创建独立的线程,与对应的客户端进行数据通信
- 并发数较大时,就会创建大量的线程,系统资源占用大
- 连接建立后,若没有数据操作,该线程就会进入阻塞,造成资源浪费  


## ⭐ NIO
- `NIO` 是从`JDK1.4` 开始java提供的改进的输入输出的新特性,称为`NIO` 
- `NIO` 相关包都在 `java.nio` 包下,并对原 `java.io` 包中很多类进行改写
- `NIO` 有三大核心部分 `Channel` 通道, `Buffer` 缓冲区, `Selector` 选择器
- `NIO` 是面向 缓冲区,面向块编程的
- `NIO` 非阻塞模式,使一个线程从某个通道发送请求或者读取数据,但它仅能得到目前可用的数据,如果目前没有可用数据,就什么也不做,而不是保持线程阻塞


### NIO BIO 对比
- BIO 以流(Stream)的方式处理数据 ,NIO以块`Buffer`的方式处理数据,效率更高
- BIO是阻塞的,NIO是非阻塞的
- BIO基于字符和字节流进行操作
- NIO 基于 Channel 和Buffer 进行操作, Selector 用于监听多个通道的事件,因此使用单个线程就可以监听多个客户端通道  
  - 通俗理解：若有100个请求过来，根据实际情况分配10个或20个线程去处理，一个线程对应一个selector 处理多个请求，而不是像BIO那样分配100个线程处理

### NIO三大核心
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty1.png) 

关系图说明：  
- 每个 `channel` 都会对应一个Buffer
- `Selector` 对应一个线程, 一个线程对应多个 `channel`  
- 程序切换到那个 `channel` 是由事件 `Event` 决定的
- `Selector` 会根据不同的事件 在各个通道上切换 
- `Buffer` 是一个内存块,底层是一个数组
- 数据的读取写入都是通过`Buffer`
- `Buffer` 可以读取也可以写入,通过`flip` 方法切换
- `channel` 是双向的,可以返回底层操作系统的情况  

## ⭐ Buffer 缓冲区 （核心一）
> 缓冲区本质上是个可以读写数据的内存块,可以理解为一个容器对象,
> `Buffer` 类是一个顶层父类,子类有 `ByteBuffer` `ShortBuffer`... 但是没有 `BooleanBuffer`

```java
public class BufferDemo {
    public static void main(String[] args) {
        // ByteBuffer LongBuffer...
        IntBuffer allocate = IntBuffer.allocate(5);
        for (int i = 0; i < allocate.capacity(); i++) {
            allocate.put(i);
        }
        //转换 读写切换 写模式切换为读模式
        allocate.flip(); //重置position
        while (allocate.hasRemaining()){
            //每次get后指针后移获取下一个值
            System.out.println(allocate.get());
        }
    }
}
```

### Buffer类及其子类
> Buffer类定义了所有的缓冲区都具有的四个属性,来提供关于其所包含的数据元素的信息

```java
//源码
    private int mark = -1;
    private int position = 0;
    private int limit;
    private int capacity;
```
- `capacity` 容量,缓冲区创建时设定,不可改变
- `limit` 缓冲区当前终点,不能对缓冲区超过极限的位置进行读写,且极限可以修改
- `position` 位置,下一个要被读或写的元素的索引,每次读写都会改变值,为下次读写做准备
- `mark` 标记

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty2.png)

> 对于java基本类型(Boolean 除外),都有一个对应的Buffer类型,最常用的是`ByteBuffer`   


ByteBuffer 常用方法    
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty3.png)


## ⭐ Channel 通道（核心二）
> NIO 的通道类似于流,但有如下区别   

- 通道channel可以同时进行读和写(双向),而流只能读或者写
- 通道channel可以实现异步读写数据
- 通道channel可以从缓冲区读数据,也可以写数据到缓冲区  


常用的 Channel 子类有 :     
- `FileChannel`(文件读写)   
- `DatagramChannel`(UDP读写)    
- `ServerSocketChannel`  `SocketChannel`(TCP读写)

### Channel 基本操作 

> 文件写入内容

```java
public class WriteDemo1 {

    public static void main(String[] args) throws FileNotFoundException, IOException {
        String s = "hello,测试";
        //创建输出流
        FileOutputStream fileOutputStream = new FileOutputStream("d:\\file01.txt");
        //通过Stream 获取 对应的FileChannel
        FileChannel channel = fileOutputStream.getChannel();

        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        byteBuffer.put(s.getBytes());
        byteBuffer.flip();

        //将bytebuffer的数据 写入channel
        channel.write(byteBuffer);
        fileOutputStream.close();


    }
}
```


> 读取文件内容

```java
//通过 Channel读取文件内容
public class ReadDemo2 {

    public static void main(String[] args) throws FileNotFoundException, IOException {
        File file = new File("D:\\file01.txt");
        FileInputStream fileInputStream = new FileInputStream(file);
        FileChannel channel = fileInputStream.getChannel();
        //创建缓冲区
        ByteBuffer byteBuffer = ByteBuffer.allocate((int) file.length());
        //读取数据 并放入到缓冲区
        channel.read(byteBuffer);
        //将字节信息转换为字符串
        String s = new String(byteBuffer.array());
        System.out.println(s);
        fileInputStream.close();

    }
}
```

> 读取文件并拷贝

```java
public class ReadAndWriteDemo3 {

    public static void main(String[] args) throws FileNotFoundException, IOException {
        File file = new File("D:\\file01.txt");
        FileInputStream fileInputStream = new FileInputStream(file);
        FileChannel channel = fileInputStream.getChannel();
        //创建缓冲区
        ByteBuffer byteBuffer = ByteBuffer.allocate((int) file.length());
        //创建输出流
        FileOutputStream fileOutputStream = new FileOutputStream("d:\\file02.txt");
        //通过Stream 获取 对应的FileChannel
        FileChannel channel1 = fileOutputStream.getChannel();
        while (true){
            byteBuffer.clear();
            //读取数据 并放入到缓冲区
            int read = channel.read(byteBuffer);
            if(read ==-1){
                break;
            }
            byteBuffer.flip();
            //将bytebuffer的数据 写入channel
            channel1.write(byteBuffer);
        }
        fileInputStream.close();
        fileOutputStream.close();
    }
}
```

> 使用 `transferFrom\transferTo` 拷贝文件 

```java
public class TransDemo {
    //拷贝图片
    public static void main(String[] args) throws IOException {
        FileInputStream fileInputStream1 = new FileInputStream("d:\\a1.png");
        FileOutputStream fileOutputStream = new FileOutputStream("d:\\a2.jpg");
        FileChannel channel1 = fileInputStream1.getChannel();
        FileChannel channel2 = fileOutputStream.getChannel();

        channel2.transferFrom(channel1,0,channel1.size());
//        channel1.transferTo(0,channel2.size(),channel2);

        channel1.close();
        channel2.close();
        fileInputStream1.close();
        fileOutputStream.close();

    }
}

```

### ⭐ 关于Buffer 和 Channel 的注意事项
- ByteBuffer 支持类型化put 和get ,put放入什么类型 get就应该使用什么类型来取出

```java
        ByteBuffer allocate = ByteBuffer.allocate(64);
        allocate.putInt(11);
        allocate.putLong(112);
        allocate.putChar('可');

        allocate.flip();
        //get 顺序要一致
        System.out.println(allocate.getInt());
        System.out.println(allocate.getLong());
        System.out.println(allocate.getChar());
```

- 可以将一个Buffer 转为 只读buffer `buffer.asReadOnlyBuffer` 返回的Buffer 只能读不能修改,此时返回的 buffer 类型为 `class java.nio.HeapByteBufferR` 后缀 `R` 一般为只读标记   
- `MappedByteBuffer` 可以让文件直接在内存 (堆外的内存)中进行修改,操作系统不需要拷贝一次

```java
public class MappedByteDemo {
    public static void main(String[] args) throws IOException {
        //hello,测试 
        //r 只读  rw 读写 
        RandomAccessFile randomAccessFile = new RandomAccessFile("d:\\file01.txt","rw");
        FileChannel channel = randomAccessFile.getChannel();
        //读写模式
        //可以直接修改的起始位置
        // 映射到内存的大小,将文件的5个字节映射到内存
        // 可修改 的索引为 0-4 共五个
        MappedByteBuffer mappedByteBuffer = channel.map(FileChannel.MapMode.READ_WRITE, 0, 5);
        mappedByteBuffer.put(0,(byte) '0');
        mappedByteBuffer.put(1,(byte) '9');
        //改为 09llo,测试
        randomAccessFile.close();

    }
}
```

- Buffer 的 分散和聚合 `Scattering`  `Gathering`  
  - `Scattering` 将数据写入到buffer时,可以采用buffer数组,依次写入
  - `Gathering` 从buffer 读取数据时,可以采用buffer数组,依次读取
 
```java
public class ScatterAndGatherDemo {

    public static void main(String[] args) throws IOException {
        //使用ServerSocketChannel
        ServerSocketChannel socketChannel = ServerSocketChannel.open();
        InetSocketAddress inetSocketAddress = new InetSocketAddress(7000);
        //绑定端口到socket 并启动
        socketChannel.socket().bind(inetSocketAddress);
        //创建buffer 数组
        ByteBuffer[] byteBuffers = new ByteBuffer[2];
        byteBuffers[0] = ByteBuffer.allocate(5);
        byteBuffers[1] = ByteBuffer.allocate(3);

        SocketChannel accept = socketChannel.accept();
        int messageLength = 8;
        while (true) {
            int byteRead = 0;
            while (byteRead < messageLength) {
                long read = accept.read(byteBuffers);
                byteRead += read;
                System.out.println("byteRead=" + byteRead);
                Arrays.stream(byteBuffers)
                        .map(bu -> "postion=" + bu.position() + ",limits=" + bu.limit()).forEach(System.out::println);
            }
            //将所有的buffer 进行flip
            Arrays.stream(byteBuffers).forEach(ByteBuffer::flip);
//            将数据读出
            int byteWrite = 0;
            while (byteWrite < messageLength) {
                long write = accept.write(byteBuffers);
                byteWrite += write;
            }
            //将所有buffer 进行clear
            Arrays.stream(byteBuffers).forEach(ByteBuffer::clear);
            System.out.println("byteRead=" + byteRead + ",byteWrite=" + byteWrite);
        }
    }
}
```

## ⭐ Selector 选择器 （核心三）
- NIO 使用 Selector 用一个线程处理多个客户端连接
- Selector 能够检测多个注册的通道(Channel)上是否有事件发生 
- 只有在连接/通道 有真正读写事件发生时,才会进行读写,就大大减少了系统开销
- 避免多线程间上下文切换

### Selector 类相关方法 

```java
public abstract class Selector implements Closeable {
    
    //得到一个选择器对象
    public static Selector open() throws IOException {
        return SelectorProvider.provider().openSelector();
    }

    //监控所有的注册通道,当有IO操作可以进行时,将对应的SelectionKey 加入到内部集合中并返回

    //参数用来设置超时时间 ,阻塞
    public abstract int select() throws IOException;
    //阻塞直到超时
    public abstract int select(long timeout) throws IOException;


    //从内部结合中得到所有的 SelectionKey
    public abstract Set<SelectionKey> selectedKeys();

    //唤醒selector
    public abstract Selector wakeup();

    //不阻塞,立刻返回
    public abstract int selectNow() throws IOException;
}
```


## ⭐ NIO 原理分析
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty4.png)

- 当有客户端连接时,会通过 `ServerSocketChannel` 得到 `SocketChannel` `ServerSocketChannel.open()`
- 将 `socketChannel` 注册到 `Selector` 上 ,通过 `SelectableChannel.register(Selector sel,int ops)` ,一个 `selector` 可以注册多个 `SocketChannel`
  - `ops` 可传入 值表示监听事件类型 如: `OP_READ = 1 << 0`(1)  `OP_WRITE = 1 << 2`(4) `OP_CONNECT = 1 << 3` (8)...
- 注册后返回 `SelectionKey` ,会和该 `Selector` 关联
- `Selector` 进行监听 `select` 方法,返回有事件发生的通道的个数
- 进一步得到各个有事件发生的 `SelectionKey` 
- 通过 `SelectionKey` 反向获取 `SelectionKey.SocketChannel channel()`
- 通过得到的 `channel` 完成业务


## NIO 案例

> 服务器 

```java
public class NIOServer {
    public static void main(String[] args) throws IOException {
        //创建serversocketchannel
        ServerSocketChannel socketChannel = ServerSocketChannel.open();
        //创建一个selector
        Selector selector = Selector.open();
        socketChannel.socket().bind(new InetSocketAddress(7000));
        //设置为非阻塞 与Selector一起使用时，Channel必须处于非阻塞模式下。
        socketChannel.configureBlocking(false);

        //把 serverSocketChannel 注册到 selector 关心事件为  OP_ACCEPT
        socketChannel.register(selector, SelectionKey.OP_ACCEPT);

        //循环等待客户端连接
        while (true) {
            if (selector.select(1000) == 0) {//没有事件发生
                System.out.println("服务器等待了1s 无连接");
                continue;
            }
            //返回值不为0 返回相关的selectionKeys集合
            //如果返回>0 表示已经获取到关注的事件
            //selector.selectedKeys 返回关注事件的集合
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            Iterator<SelectionKey> iterator = selectionKeys.iterator();
            while (iterator.hasNext()) {
                SelectionKey key = iterator.next();
                //根据key 对应的通道发生的事件,做响应的处理
                if (key.isAcceptable()) {//对应 SelectionKey.OP_ACCEPT
                    //给该客户端生成 SocketChannel
                    SocketChannel accept = socketChannel.accept();
                    accept.configureBlocking(false);
                    System.out.println("客户端连接成功,生成一个SocketChannel" + accept.hashCode());
                    //将当前的socketChannel注册到selector ,关注事件 OP_READ ,关联一个Buffer
                    accept.register(selector, SelectionKey.OP_READ, ByteBuffer.allocate(1024));
                }

                if (key.isReadable()) {//发生了读事件OP_READ
                    //通过key 反向获取对应channel
                    SocketChannel channel = (SocketChannel) key.channel();
                    //获取到该channel关联的buffer
                    ByteBuffer buffer = (ByteBuffer) key.attachment();
                    channel.read(buffer);
                    System.out.println("客户端发送的数据:" + new String(buffer.array()));

                }
                //手动从当前集合中移除当前的selectionKey,防止重复操作
                iterator.remove();

            }
        }


    }
}
```

> 客户端

```java
public class NIOClient {
    public static void main(String[] args) throws IOException {
        //得到一个网络通道
        SocketChannel socketChannel = SocketChannel.open();

        socketChannel.configureBlocking(false);
        //提供服务器端的ip和端口
        InetSocketAddress inetSocketAddress = new InetSocketAddress("127.0.0.1", 7000);
        //连接服务器
        if (!socketChannel.connect(inetSocketAddress)) {
            while (!socketChannel.finishConnect()) {
                System.out.println("因为连接需要时间客户端不会阻塞,可以做其他工作...");
            }
        }
        //如果连接成功就发送数据
        String str = "hello 测试";
        ByteBuffer byteBuffer = ByteBuffer.wrap(str.getBytes());
        //发送数据,将buffer 写入channel
        socketChannel.write(byteBuffer);
        System.in.read();

    }
}
```

## SelectionKey API 
> `Selector` 的子类 `SelectorImpl` 维护了一个 `HashSet<SelectionKey> keys`    
> 当 `ServerSocketChannel.register()`进行注册时 放入keys中,是当前所有向 `Selector` 注册的 `SelectionKey` 的集合  


> 查看源码发现还有一个 `Set<SelectionKey> selectedKeys` 集合  
> 这个代表相关事件已经被 `Selector` 捕获的 `SelectionKey` 的集合， `Selector` 的 `selectedKeys()` 方法返回该集合  

`SelectionKey` ，表示 `Selector` 和网络通道的注册关系, 共四种:  
- `int OP_READ = 1 << 0` `1` 读操作 对应判断方法 `isReadable()`
- `int OP_WRITE = 1 << 2` `4` 写操作 对应`isWritable()`
- `int OP_CONNECT = 1 << 3` `8` 连接已经建立  `isConnectable()` 和下面的 `isAcceptable()` 完全相同
- `int OP_ACCEPT = 1 << 4` `16` 有新的网络可以accept 对应`isAcceptable()`  


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty5.png)

## ServerSocketChannel  
> `ServerSocketChannel` 在服务器端监听新的客户端 Socket 连接

- `open()` 得到一个 `ServerSocketChannel` 通道
- `configureBlocking()` 设置阻塞或者非阻塞模式
- `bind(SocketAddress local)` 设置服务器端口号
- `register(Selector sel, int ops)` 注册选择器并设置监听事件  

```java
public abstract class ServerSocketChannel extends AbstractSelectableChannel implements NetworkChannel{

    //得到一个 ServerSocketChannel 通道
    public static ServerSocketChannel open()
    //设置服务器端端口号
    public final ServerSocketChannel bind(SocketAddress local)
    //设置阻塞或非阻塞模式，取值 false 表示采用非阻塞模式
    public final SelectableChannel configureBlocking(boolean block)
    // 接受一个连接，返回代表这个连接的通道对象
    public SocketChannel accept()
    // 注册一个选择器并设置监听事件
    public final SelectionKey register(Selector sel, int ops)
}
```

## SocketChannel 
> 网络IO 通道,负责具体的读写操作,把缓冲区的数据写入通道或者读取通道数据写入缓冲区

```java
public abstract class SocketChannel
extends AbstractSelectableChannel
implements ByteChannel, ScatteringByteChannel, GatheringByteChannel, NetworkChannel{
    public static SocketChannel open();//得到一个 SocketChannel 通道
    public final SelectableChannel configureBlocking(boolean block);//设置阻塞或非阻塞模式，取值 false 表示采用非阻塞模式
    public boolean connect(SocketAddress remote);//连接服务器
    public boolean finishConnect();//如果上面的方法连接失败，接下来就要通过该方法完成连接操作
    public int write(ByteBuffer src);//往通道里写数据
    public int read(ByteBuffer dst);//从通道里读数据
    public final SelectionKey register(Selector sel, int ops, Object att);//注册一个选择器并设置监听事件，最后一个参数可以设置共享数据
    public final void close();//关闭通道
}
```

## NIO群聊系统   
见代码


## ⭐ NIO与零拷贝
> 零拷贝是网络编程的关键,很多性能优化都需要零拷贝  
> 在 Java 程序中,常用的零拷贝有 `mmap` 和 `sendFile`   

> 传统IO拷贝,繁琐,多次拷贝,包括 CPU拷贝,DMA拷贝

> `mmap` 优化:  
> `mmap` 通过内存映射,将文件映射到内核缓冲区,同时,用户空间可以共享内核空间的数据,这样,在进行网络传输时,就可以减少内核空间到用户空间的拷贝次数


> `sendFile` 优化:  
> `Linux 2.1` 版本提供了 `sendFile` 函数直接从内核缓冲区,进入到 `SocketBuffer`   
> `Linux 2.4` 版本优化了`sendFile`,避免了 `内核到Buffer` 直接拷贝到协议栈   


> 零拷贝 是从操作系统角度来看,并不是完全零拷贝
> 零拷贝 可以减少数据复制次数,提高性能

### 实例
> 使用 `FileChannel.transferTo()` 方法  
> Linux 系统下 `transferTo` 方法可以完成传输
> Windows 系统下 `transferTo` 方法只能发送 8Mb ,超过8Mb 的文件需要进行分段传输  

```java
    FileChannel fileChannel = new FileInputStream(filename).getChannel();
    //底层用了零拷贝,返回值为传输文件大小
    long transfer = fileChannel.transferTo(0, fileChannel.size(), socketChannel);
```

## AIO
> `JDK 7` 引入了`AsynchronusI/O` ,即AIO,在进行IO编程时,常用到两种模式: `Reactor` 和 `Proactor`
> `NIO` 就是 `Reactor` 模式
> `AIO` 即 `NIO2.0` 是异步不阻塞的IO,使用 `Proactor` 模式   


## Netty概述  
> Netty 是由 JBOSS 提供的一个 Java 开源框架。Netty 提供异步的、基于事件驱动的网络应用程序框架，用以快速开发高性能、高可靠性的网络IO程序  
> Netty 是目前最流行的 NIO 框架，Netty 在互联网领域、大数据分布式计算领域、游戏行业、通信行业等获得了广泛的应用，知名的Elasticsearch 、Dubbo框架内部都采用了 Netty  
> Netty 对 JDK 自带的 NIO 的 API 进行了封装   
> Netty 版本分为 netty3.x 和 netty4.x、~~netty5.x(废弃)~~


## ⭐ 线程模型基本介绍
- 传统阻塞IO服务模型
  - 用户线程进行read操作时，需要等待操作系统执行实际的read操作，此期间用户线程是被阻塞的，无法执行其他操作
- 非阻塞
  - 用户线程在一个循环中一直调用read方法，若内核空间中还没有数据可读，立即返回,循环往复
- 多路复用
  - Java中通过 `Selector` 实现多路复用

- Reactor 模式
  - 单Reactor 单线程
  - 单Reactor 多线程
  - 主从 Reactor 多线程


> Netty 线程模式主要就是基于 主从Reactor多线程模型进行了改进   

### 传统阻塞IO模型
 
- 采用阻塞IO模式获取输入的数据  
- 每个连接都需要独立的线程完成数据的输入,业务处理,数据返回   
- 并发加大时,线程过多,系统资源耗尽   
- 创建连接后,如果暂时没有数据可读,线程会阻塞在read操作,线程资源浪费

> 总结:无法处理高并发请求,资源浪费   


## ⭐ Reactor 模式
`反应器模式` `分发者模式` `通知者模式`     

> 基于**IO复用模型**,多个连接共用一个阻塞对象   
> 基于**线程池复用**线程资源   

> Reactor 在一个单独的线程中运行，负责响应IO事件，当检测到一个新的事件，将其发送给相应的Handler去处理；新的事件包含连接建立就绪、读就绪、写就绪等   
> Handler 将自身（handler）与事件绑定，负责事件的处理，完成channel的读入，完成处理业务逻辑后，负责将结果写出channel


### 单线程Reactor
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty8.png)  

- 模型简单,没有多线程,线程竞争,进程通信
- 性能问题,只有一个线程
- 不可靠

### 单Reactor多线程
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty9.png)  


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty11.png)  

- `Reactor` 对象通过 `select` 监控请求事件,收到事件后通过 `dispatch` 进行分发
- 如果是连接请求,使用 `acceptor` 处理并创建 Handler对象
- 若不是连接请求,通过 `reactor` 调用对应 `handler` 处理
- `handler` 只用做响应事件,不做具体业务处理
- 读取数据后 使用线程池分配独立线程完成真正的任务,并将结果返回给 `handler`
- `handler` 收到响应后,通过 `send` 将结果返回给 `client`  

> 可以充分利用多核cpu处理能力   
> 多线程间数据共享和访问比较复杂,`reactor` 处理所有的事件监听和响应,高并发场景容易出现性能问题  

### 主从 Reactor多线程
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty10.png)


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty12.png)

- `Reactor` 主线程 `MainReactor` 通过 `select` 监听连接事件,收到事件后,通过 `Acceptor` 处理连接事件  
- `Acceptor` 处理连接事件后, `MainReactor` 将连接分配给 `SubReactor`  
- `subReacotr` 将连接加入队列进行监听,并创建 `handler` 进行事件处理
- 当有新事件发生时, `subreactor` 就会调用对应的 `handler` 处理
- `handler` 读取数据,分发给worker线程池处理,
- worker 进行业务处理并返回结果 ,handler 收到响应结果后通过send 将结果返回给client  
- MainReactor 可以关联多个 SubReactor  

> 优点: 主从线程交互简单职责明确     
> 缺点: 编程难度高     
> 应用实例: Nignx Memcached Netty

### Netty模型  
> netty 主要针对 主从模型 做了一定的改进,其中主Reactor模型中有多个Reactor   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty13.png)

- `BossGroup` 上维护的 `selector` 只关注 `accept` 事件
- 收到 `accept` 事件后,获取对应的 `SocketChannel` 封装成 `NIOSocketChannel` 注册到 `WorkerGroup` 的 `selector` 中  
- `WorkerGroup` 的 `selector` 监听到对应事件就开始处理   

## ⭐ Netty 模型 详细版  
- Netty 抽象出两组线程池  
  - `BossGroup` 负责接收连接
  - `WorkerGroup` 负责网络读写
- `BossGroup` `WorkerGroup` 类型都是 `NioEventLoopGroup` 
- `NioEventLoopGroup` 相当于一个事件循环组,组中包含多个事件循环 `NioEventLoop` 
- `NioEventLoop` 表示一个不断循环的执行处理任务的线程(每个 `NioEventLoop`都是串行处理 ),每个`NioEventLoop` 都有一个 `selector` ,用于监听绑定在其上的socket网络通讯
- `NioEventLoopGroup` 包含多个 `NioEventLoop`  
- 每个`BossNioEventLoop` 有三个步骤
  - 轮询 `accept` 事件
  - 处理 `accept` 事件,与 `client` 建立连接,生成 `NioSocketChannel` ,并将其注册到某个 `WorkerNioEventLoop` 上的 `selector`
  - 处理任务队列的任务, `runAllTasks` ?
- 每个 `WorkerNioEventLoop`执行的步骤
  - 轮询处理 `read/write` 事件
  - 处理 IO 事件,即read write 事件,在对应的 `NioSocketChannel` 处理
  - 处理任务队列的任务 `runAllTasks`
- `pipeline` 本质是个双向链表，每个 Channel 都有一个 ChannelPipeline
  - 可以反向得到 `channel` 二者可以理解为相互包含的关系

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/nettyModel.png)

### ⭐ 事件循环(EventLoop)中的TaskQueue

任务队列的Task 3种典型使用场景    
- 用户程序自定义的普通任务
- 用户自定义定时任务
- 非当前Reactor线程调用Channel 的各种方法  


> 例1(自定义的普通任务):若有一个非常耗时的异步任务,就可以将其提交到该channel 对应的 `NioEventLoop` 的 `taskQueue` 

代码片段    
```java
//(ChannelHandlerContext ctx)
ctx.channel().eventLoop().execute(()->{
    //异步任务执行位置
            try {
                Thread.sleep(10*1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            ctx.writeAndFlush(Unpooled.copiedBuffer("Hello,客户端111",CharsetUtil.UTF_8));
        });
//当前的异步任务是在同一个 eventLoop中,所以是一个线程,多个异步任务会顺序执行
```

> 例2(自定义定时任务):该任务会提交到 `scheduleTaskQueue` 中

```java
//5为延时时长 (ChannelHandlerContext ctx)
ctx.channel().eventLoop().schedule(()->{},5, TimeUnit.SECONDS);
```

> 例3 非当前Reactor调用,在server端, 获取客户端socketChannel 并将任务提交到 对应 channel 中的 `Task` 中即可    
> 可以使用一个集合管理 `SocketChannel` 在推送消息时,可以将业务加入到各个`ChannelInitializer`中添加 `channel` 对应的 `NioEventLoop` 的 `taskQueue` 或者 `scheduleTaskQueue`  

> 多个 `TaskQueue` 是串行,注意 `Queue` 的定义,依次调用 `Task`

### ⭐ 关于NioEventLoop
- `NioEventLoopGroup` 下包含多个 `NioEventLoop`
- 每个`NioEventLoop` 中包含有一个 `selector` 一个 `taskQueue` 
- 每个 `NioEventLoop` 的 `selector` 可以注册多个 `NioChannel`
- 每个 `NioChannel` 只会绑定在唯一的 `NioEventLoop` 上
- 每个 `NioChannel` 都绑定有一个自己的 `ChannelPipeline`



## 异步模型  
> 异步过程调用后,调用者不能立刻得到结果,实际处理这个调用的组件在完成后,通过状态,通知,和回调来通知调用者   

- Netty 的 IO 操作是异步的,包括 `Bind` `Write` `Connect` 都会返回一个 `ChannelFuture`   
- 调用者通过 `Future-Listener` 机制获取调用结果
- Netty 的异步模型是建立在 `future` 和 `callback` 上的,返回的 `Future` 后续可以通过 `Future` 去监控方法的处理过程,以及获取返回结果(`Future-Listener` 机制)  

> Future 表示异步执行的结果,可以通过提供的方法来检测是否完成   
> ChannelFuture 是一个接口,`public interface ChannelFuture extends Future<Void> {}`   
> 我们可以添加监听器 `addListener(s)` ,当监听的事件发生时,就会通知到监听器

### Future-Listener机制  
- 当Future 对象创建时,处于非完成状态,通过返回 `ChannelFuture` 来获取操作执行状态,注册监听函数来执行完成后的操作  
- 常见操作:
  - `isDone` 操作是否完成
  - `isSuccess` 是否成功
  - `getCause` 失败原因
  - `isCancelled` 是否被取消
  - `addListener` 注册监听器,当前操作`isDone` 会通知指定的监听器 

```java
            //绑定端口并且同步,启动,生成一个 ChannelFuture对象
            ChannelFuture cf = bootstrap.bind(6668).sync();
            cf.addListener(new ChannelFutureListener() {
                @Override
                public void operationComplete(ChannelFuture channelFuture) throws Exception {
                    if (cf.isSuccess()){
                        System.out.println("success");
                    }else {
                        System.out.println("fail");
                    }
                }
            });
```

### Promise
> Promise相当于一个容器，可以用于存放各个线程中的结果，然后让其他线程去获取该结果

```java
public class NettyPromise {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 创建EventLoop
        NioEventLoopGroup group = new NioEventLoopGroup();
        EventLoop eventLoop = group.next();

        // 创建Promise对象，用于存放结果
        DefaultPromise<Integer> promise = new DefaultPromise<>(eventLoop);

        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // 自定义线程向Promise中存放结果
            promise.setSuccess(50);
        }).start();

        // 通过get获取 主线程从Promise中获取结果
        System.out.println(Thread.currentThread().getName() + " " + promise.get());



        //通过 listener 获取
          promise.addListener(new GenericFutureListener<Future<? super Integer>>() {
            @Override
            public void operationComplete(Future<? super Integer> future) throws Exception {
                if (future.isSuccess()){
                    System.out.println("success");
                }else {
                    System.out.println("fail");
                }
            }
        });
    }
}
```

## Http实例/过滤请求
见代码 
> Server 通过 childHandler() 注册一个  `ChannelInitializer` 通过它的 `ChannelPipeline` 来  添加一个handler(ChannelInboundHandler)


```java
/**
 * SimpleChannelInboundHandler 是 ChannelInboundHandlerAdapter子类
 * HttpObject 客户端和服务器端相互通讯的数据
 */
public class TestHttpServerHandler extends SimpleChannelInboundHandler<HttpObject> {
    /**
     * 读取客户端数据
     *
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, HttpObject msg) throws Exception {
        if (msg instanceof HttpRequest) {
            System.out.println("msg类型" + msg.getClass());
            System.out.println("客户端地址" + ctx.channel().remoteAddress());

            HttpRequest httpRequest = (HttpRequest) msg;
            URI uri = new URI(httpRequest.uri());
            //拦截固定路径
            if("/favicon.ico".equals(uri.getPath())){
                System.out.println("请求了 favicon.ico");
                //这样操作会不做响应,浏览器一直padding
                return;
            }

            //回复信息给浏览器
            ByteBuf byteBuf = Unpooled.copiedBuffer("hello 我是服务器", CharsetUtil.UTF_8);
            //构造一个http响应,即HTTP response
            FullHttpResponse defaultFullHttpResponse = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK, byteBuf);
            defaultFullHttpResponse.headers().set(HttpHeaderNames.CONTENT_TYPE,"text/plain;charset=utf-8");
            defaultFullHttpResponse.headers().set(HttpHeaderNames.CONTENT_LENGTH,byteBuf.readableBytes());

            ctx.writeAndFlush(defaultFullHttpResponse);

        }

    }
}
```
## ⭐ Netty 核心模块

### BootStrap ServerBootStrap 
> Netty 通常由一个Bootstrap 开始,主要作用是配置整个Netty 程序,串联各个组件    
> Netty中 `Bootstrap` 类是客户端程序的启动引导类, `ServerBootStrap` 是服务端的启动引导类   

常见的方法有:  
- `public ServerBootstrap group(EventLoopGroup parentGroup, EventLoopGroup childGroup) ` 用于服务器端,设置两个EventLoop
- `public ServerBootstrap group(EventLoopGroup group)` 用于客户端,用来设置一个EventLoop  
- `public B channel(Class<? extends C> channelClass)` `AbstractBootstrap` 中用来设置一个服务器端的通道实现
  - 服务器端 `.channel(NioServerSocketChannel.class)`
  - 客户端 `.channel(NioSocketChannel.class)`  
- `public <T> B option(ChannelOption<T> option, T value) ` 用来给ServerChannel 添加配置
- `public <T> ServerBootstrap childOption(ChannelOption<T> childOption, T value)` 给接收到的通道添加配置
- `public ServerBootstrap childHandler(ChannelHandler childHandler) ` 用来设置业务处理类(自定义handler) 针对 `workerGroup`
  - `public B handler(ChannelHandler handler)` 设置处理类,针对 `bossGroup`
- `public ChannelFuture bind(int inetPort)` `AbstractBootstrap`用于服务器端 设置IP 端口 
- `public ChannelFuture connect(String inetHost, int inetPort) ` `Bootstrap`用于客户端 连接服务器


### Future ChannelFuture 
> Netty 中所有的IO操作都是异步的,不能立刻得知消息是否被正确处理,但是可以注册一个监听,通过 `Future` 和 `ChannelFuture` ,    
> 在操作执行成功或者失败自动触发注册的监听事件

### Channel
- Netty 网络通信组件,能够用于执行网络IO操作
- 通过 `Channel` 可以获得当前网络连接的通道状态
- 通过 `Channel` 可以获得当前连接的配置参数
- `Channel` 提供异步IO 操作,异步意味着任何IO调用都将立即返回,返回的是一个 `ChannelFuture` 实例,可以在IO操作成功,失败或者取消时通知调用方 
- 支持关联IO操作与对应的处理程序
- 不同协议不同阻塞类型都有不同的 `Channel` 类型与之对应 :
  - `NioScoketChannel` 异步客户端TCP Socket
  - `NioServerScoketChannel` 异步服务器端 TCP Socket
  - `NioDatagramChannel` 异步UDP
  - `NioSctpChannel` 异步客户端 Sctp连接
  - `NioSctpServerChannel` 异步服务端 Sctp连接

### Selector
- Netty 基于 `Selcetor` 对象实现**IO多路复用**,通过 `Selector` 一个线程可以监听多个连接的 `Channel` 事件  
- 一个 `Channel` 注册到 `Selector` 上后, `Selector` 内部的机制会自动不断查询注册的 `Channel` 是否有已经就绪的IO事件


### ChannelHandler 及其实现类  
- ChannelHandler 是一个接口,处理IO事件或拦截IO操作,并将其转发到器 ChannelPipeline 中的下一个处理程序
- ChannelHandler 本身并没有提供很多方法,但其有很多子类实现  
- `ChannelInboundHandlerAdapter` 用于处理入站 I/O 事件
- `ChannelOutboundHandlerAdapter` 用于处理出站 I/O 事件 
  - 如果事件的运动方向是从客户端到服务端的，那么我们称这些事件为出站的，反之为入站
- 我们经常需要自定义Handler 类去继承 `ChannelInboundHandlerAdapter` 然后通过重写相应方法实现业务逻辑  
  - `channelActive` 通道就绪
  - `channelRead` 通道读取数据事件
  - `channelReadComplete` 数据读取完成  


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/channelHandler.png)


### Pipeline  ChannelPipeline  
- 在 Netty 中每个 Channel 都有且仅有一个 ChannelPipeline 与之对应，它们的组成关系如下


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty14.png)  

- `ChannelPipeline` 是一个 `Handler` 的集合，它负责处理和拦截`inbound或者outbound` 的事件和操作，相当于一个贯穿 Netty 的链


- 在http实例中的部分代码debug过程可以看到 pileline 内部结构  

```java
public class TestServerInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        //得到管道
        ChannelPipeline pipeline = ch.pipeline();

        //加入一个netty提供的 httpServerCodec (codec 表示 coder 和 decoder)
        /**
         * HttpServerCodec netty提供的基于http的编,解码器
         */
        pipeline.addLast("MyHttpServerCodec",new HttpServerCodec());
        pipeline.addLast("MyHttpServerHandler",new TestHttpServerHandler());

    }
}
```

- 整体结构

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty15.png)


- pipeline 是一个双向链表,下图为head链 ,`next` 中为下一个 `handler`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty16.png)

- 同理,tail 链 ,方向和head 相反,`prev` 中为上一个 `handler`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty17.png)

> 一个 `Channel` 包含了一个 `ChannelPipeline` ,而 `ChannelPipeline` 中又维护了一个又 `ChannelHandlerContext` 组成的双向链表,   
> 兵器而每个 `ChannelHandlerContext` 中又关联着一个 `ChannelHandler`    

常用方法:  
- `addFirst` 把 handler 添加到链中第一个位置
- `addLast` 最后一个位置  

### ChannelHandlerContext  
- 保存 `Channel` 相关的所有上下文信息，同时关联一个 `ChannelHandler` 对象 
- 即 `ChannelHandlerContext` 中包含一个具体的事件处理器 `ChannelHandler` ，同时 `ChannelHandlerContext` 中也绑定了对应的 `pipeline` 和 `Channel` 的信息，方便对 `ChannelHandler` 进行调用.
- 常用方法:
  - `ChannelFuture close()` 关闭通道
  - `ChannelOutboundInvoker flush()`，刷新  
  - `ChannelFuture writeAndFlush(Object msg)` 将数据写入到 `ChannelPipeline` 中, 当前ChannelHandler 的下一个 ChannelHandler 开始处理

### ChannelOption 
- Netty 在创建 Channel 实例后,一般都需要设置 ChannelOption 参数 
  - `ChannelOption.SO_BACKLOG` ,初始化服务器可连接队列大小
  - `ChannelOption.SO_KEEPALIVE` ,一直保持连接活动状态

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty18.png)

### EventLoopGroup 和其实现类NioEventLoopGroup
- `EventLoopGroup` 是一组`EventLoop` 的抽象,Netty 为了利用多核资源,一般会有多个 `EvnetLoop` 同时工作,每个 `EventLoop` 维护一个 `Selector` 实例  
- `BossEventLoop` 负责接收客户端的连接并将 `SocketChannel` 交给 `WorkerEventLoopGroup` 来进行 IO 处理， 
  - `BossEventLoop` 通常是单线程的 `EventLoop` 
- `EventLoopGroup` 提供 `next` 接口，可以从组里面按照一定规则获取其中一个 `EventLoop` 来处理任务 
- 通常是OP_ACCEPT(OP_CONNECT)事件，然后将接收到的SocketChannel 交给WorkerEventLoopGroup
- `WorkerEventLoopGroup` 会由`next`选择其中一个 `EventLoop` 来将这个 `SocketChannel` 注册到其维护的 `Selector` 并对其后续的IO事件进行处理

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty19.png)

常用方法:  
- `public NioEventLoopGroup()` 构造方法
- `public Future<?> shutdownGracefully()` 断开连接，关闭线程

## Unpooled 类
> Netty 提供一个专门用来操作缓冲区(Netty 的数据容器)的工具类   
> `public static ByteBuf copiedBuffer(CharSequence string, Charset charset)` 通过指定字符串和编码,生成Bytebuf对象   
> bytebuf 不需要读写转换操作 `flip`  
> 其内部 有`readerIndex`  和 `writerIndex` ,`capacity` 来限定可操作区域    

部分方法参考   
```java
public static void main(String[] args) {
        ByteBuf byteBuf = Unpooled.copiedBuffer("hellow.world", CharsetUtil.UTF_8);

    if (byteBuf.hasArray()){
        byte[] content = byteBuf.array();

        System.out.println(new String(content,CharsetUtil.UTF_8));
        System.out.println("byteBuf="+ byteBuf);
        System.out.println(byteBuf.arrayOffset());
        System.out.println(byteBuf.readerIndex());
        System.out.println(byteBuf.writerIndex());
        System.out.println(byteBuf.capacity());

        //会影响readindex
        System.out.println(byteBuf.readByte());
        //不会影响 readindex
        System.out.println(byteBuf.getByte(0));

        //可读字节数
        int i = byteBuf.readableBytes();
        System.out.println(i);

        //读取 指定字符 从0开始 往后数4 个字符 注意这里
        CharSequence charSequence = byteBuf.getCharSequence(0, 4, CharsetUtil.UTF_8);
        System.out.println(charSequence);
    }
```


## 群聊实例
见代码 `nettyDemo1.com.ric.netty.groupchat`  

1. 服务器配置   

```java
public class GroupChatServer {


    private int port;

    public GroupChatServer(int port) {
        this.port = port;
    }

    public void run() throws InterruptedException {
        NioEventLoopGroup bossGroup = new NioEventLoopGroup(1);
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    //配置信息
                    .option(ChannelOption.SO_BACKLOG, 100)
                    .childOption(ChannelOption.SO_KEEPALIVE, true)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ChannelPipeline pipeline = ch.pipeline();
                            //解码器
                            pipeline.addLast("decoder", new StringDecoder());
                            //编码器
                            pipeline.addLast("encoder", new StringEncoder());
                            //处理器
                            pipeline.addLast(new GroupChatServerHandler());
                        }
                    });
            System.out.println("netty服务器启动");
            ChannelFuture sync = serverBootstrap.bind(port).sync();
            //检测关闭
            sync.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new GroupChatServer(7000).run();
    }
}
```

```java
public class GroupChatServerHandler extends SimpleChannelInboundHandler {


    //定义一个channel 组管理所有的channel
//    GlobalEventExecutor.INSTANCE  全局的事件执行器，单例
    private static ChannelGroup channelGroup = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);

    /**
     * 一旦连接建立 第一个执行
     * 将当前channel 加入到 channel组
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        Channel channel = ctx.channel();
        //加入聊天信息推送
        //该方法会将channelgroup中所有channel 遍历并发送消息
        channelGroup.writeAndFlush("[客户端]" + channel.remoteAddress() + "--" + LocalDateTime.now().format(DateTimeFormatter.ISO_DATE_TIME) + "加入聊天\n");
        channelGroup.add(channel);
    }


    /**
     * 断开连接响应
     * 会自动将当前连接 从chan nelGroup 中移除
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
        channelGroup.writeAndFlush(ctx.channel().remoteAddress() + "断开连接");
        System.out.println("channel Group Size" + channelGroup.size());
    }

    /**
     * 表示channel 处于一个活动的状态 判断上限
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush("服务指示您已上线");
        System.out.println(ctx.channel().remoteAddress() + "上线了~");
    }


    /**
     * channel 处于不活动状态 判断离线
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        System.out.println(ctx.channel().remoteAddress() + "下线了~");


    }

    @Override
    public void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {

        Channel channel = ctx.channel();
        //根据不同的情况回送不同的消息

        channelGroup.forEach(c -> {
            if (channel != c) {
                //不是当前channel 转发消息
                c.writeAndFlush("[客户]" + channel.remoteAddress() + "发送了消息" + msg + "\n");
            } else {
                c.writeAndFlush("[自己]" + channel.remoteAddress() + "发送了消息" + msg + "\n");
            }

        });


    }


    /**
     * 发生异常关闭
     *
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();

    }
}
```

2. 客户端配置  

```java
public class GroupChatClient {

    private final String host;
    private final int port;

    public GroupChatClient(String host, int port) {
        this.host = host;
        this.port = port;
    }

    public void run() {
        NioEventLoopGroup loopGroup = new NioEventLoopGroup();
        try {

            Bootstrap bootstrap = new Bootstrap().group(loopGroup)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ChannelPipeline pipeline = ch.pipeline();
                            //解码器
                            pipeline.addLast("decoder", new StringDecoder());
                            //编码器
                            pipeline.addLast("encoder", new StringEncoder());
                            pipeline.addLast(new GroupChatClientHandler());

                        }
                    });
            ChannelFuture channelFuture = bootstrap.connect(host, port).sync();
            Channel channel = channelFuture.channel();
            System.out.println("----"+channel.remoteAddress()+"----");
            //客户端需要输入信息
            Scanner scanner = new Scanner(System.in);
            while (scanner.hasNextLine()){
                String msg = scanner.nextLine();
                //通过channel 发送到服务器端
                channel.writeAndFlush(msg+ "\r\n");
            }

        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            loopGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) {
        new GroupChatClient("127.0.0.1",7000).run();
    }
}
```

```java
public class GroupChatClientHandler extends SimpleChannelInboundHandler<String> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
        System.out.println(msg.trim());
    }
}
```


## ⭐ Netty 心跳机制  

类 `IdleStateHandler` `new IdleStateHandler(3,5,7, TimeUnit.SECONDS)`的说明:   
1. `readerIdleTime` 多久没有读操作,发送一个心跳检测包检测是否连接
2. `writerIdleTime` 写操作
3. `allIdleTime` 读写操作

> 源码注释: Triggers an {@link IdleStateEvent} when a {@link Channel} has not performed read, write, or both operation for a while.

4. 当 `IdleStateEvent` 触发后,就会传递给管道的下一个Handler 进行处理,通过调用下一个 Handler 的 `userEventTriggered` (`ChannelInboundHandlerAdapter` 类中方法)  

```java
pipeline.addLast(new IdleStateHandler(3,5,7, TimeUnit.SECONDS));
//加入一个对空闲检测进一步处理的自定义的handler 处理心跳执行对应 的操作
pipeline.addLast(new MyServerHandler());
```

```java
//通过userEventTriggered 方法判断读写空闲 并进行处理
public class MyServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        if (evt instanceof IdleStateEvent){
            //将evt 转型
            IdleStateEvent event = (IdleStateEvent) evt;
            String eventType = null;
            switch (event.state()){
                case READER_IDLE:
                    eventType="读空闲";
                    break;
                case WRITER_IDLE:
                    eventType = "写空闲";
                    break;
                case ALL_IDLE:
                    eventType = "读写空闲";
                    break;
            }
            System.out.println(ctx.channel().remoteAddress()+"超时发生 ++"+eventType);
            ctx.channel().close();
        }
    }
}
```

## Netty 通过WebSocket 编程实现服务器和客户端长连接  
```java
public static void main(String[] args) {

        NioEventLoopGroup bossGroup = new NioEventLoopGroup(1);
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup,workerGroup)
                    .channel(NioServerSocketChannel.class)
                    //在boss中添加日志处理器
                    .handler(new LoggingHandler(LogLevel.INFO))
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ChannelPipeline pipeline = ch.pipeline();
                            pipeline.addLast(new HttpServerCodec());
                            //块处理器
                            pipeline.addLast(new ChunkedWriteHandler());

                            /*
                            * http数据传输中是分段的, HttpObjectAggregator 可以将多段聚合
                            * */
                            pipeline.addLast(new HttpObjectAggregator(8192));

                            /*
                            * 对于websocket 数据是以帧的形式传递
                            * 可以看到 WebsocketFrame 下面有六个子类
                            * 浏览器请求 ws://localhost:7000/
                            * 将http协议升级为ws 协议,保持长连接
                            *
                            * */
                            pipeline.addLast(new WebSocketServerProtocolHandler("/hello"));

                            //处理业务逻辑
                            pipeline.addLast(new WebSocketFrameHandler());

                        }
                    });
            ChannelFuture channelFuture = serverBootstrap.bind(9000).sync();
            channelFuture.channel().closeFuture().sync();


        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
```


## ⭐ 编码和解码基本介绍
 - 编写网络应用程序时,因为数据在网络中传输都是二进制字节码数据,在发送数据时需要编码,在接收数据时需要解码
 - `codec` 编码器 的组曾部分有两个, `decoder`(字节码数据->业务数据) 解码器 和 `encoder`(业务数据->字节码数据) 编码器

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty20.png)

- netty 自带一些编/解码器
  - `StringEncoder/StringDecoder` 字符串编码器
  - `ObjectEncoder/ObjectDecoder` java对象编码器

> Netty 本身自带的编解码器 `ObjectEncoder/ObjectDecoder` 可以用来实现POJO 对象或者各种业务对象的编解码,  
> 底层使用的仍是Java 序列化技术,存在如下问题  
> 1. 无法跨语言  
> 2. 序列化后体积过大   
> 3. 序列化性能过低   
> 4. 使用 `GoogleProtoBuf` 

## Google ProtoBuf

> 是一种轻便高效的结构化数据存储格式,可以用于结构化数据序列化,适合做RPC ,支持跨平台,跨语言  
> 性能高,可靠性高  
> protobuf 编译器能自动生成代码，  
> 后通过 protoc.exe 编译器根据.proto 自动生成.java 文件   


### 安装
ProtoBuf插件需要独立下载安装到IDEA
https://github.com/ksprojects/protobuf-jetbrains-plugin/releases/tag/v0.8.0

ProtoBuf 类型对应表  
https://developers.google.com/protocol-buffers/docs/proto3#scalar

ProtoBuf 下载（编译程序 protoc-3.19.4-win64.zip）   
https://github.com/protocolbuffers/protobuf/releases/tag/v3.19.4

### 使用 
1. 编写 模板文件 `Student.proto`

```proto
syntax = "proto3"; //版本
//生成的外部类名,同时也是文件名
option java_outer_classname = "StudentPOJO";
message Student{ //会在 StudentPOJO 生成一个内部类Student 是真正发送的POJO对象
    int32 id = 1; //Student类中有一个属性,名字为id 类型为 int32 1表示属性序号,不是值
    string name =2;
}
```

2. 使用proto软件生成java文件,cmd命令`protoc.exe --java_out=. Student.proto`   
3. 会生成文件 `StudentPOJO.java`  

### 代码
客户端传送数据给服务器端:    
- 客户端 pipeline 添加编码器 `pipeline.addLast(new ProtobufEncoder());`
- 服务器端 pipeline 添加解码器`pipeline.addLast(new ProtobufDecoder(StudentPOJO.Student.getDefaultInstance()))`
- 客户端发送方法参考  

```java
public class NettyClientHandler extends ChannelInboundHandlerAdapter {
    //通道就绪时就会触发该方法
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {

        StudentPOJO.Student student = StudentPOJO.Student.newBuilder().setId(4).setName("张三").build();
        ctx.writeAndFlush(student);
    }
}
```
- 服务器端接收方法
- 方法一 继承`ChannelInboundHandlerAdapter`  强转消息类型 

```java
public class NettyServerHandler extends ChannelInboundHandlerAdapter {

    //读取数据
    //ChannelHandlerContext 上下文对象,含有 管道pipeline 通道 channel 地址
    //Object msg 客户端发送的数据
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

        StudentPOJO.Student student = (StudentPOJO.Student) msg;
        System.out.println("客户端发送的数据"+ student.getId()+"--"+student.getName());

    }
}
```

- 方法二 继承 `SimpleChannelInboundHandler<I>` 直接指定消息类型

```java
public class NettyServerHandler2 extends SimpleChannelInboundHandler<StudentPOJO.Student> {


    @Override
    protected void channelRead0(ChannelHandlerContext ctx, StudentPOJO.Student student) throws Exception {
        System.out.println("客户端发送的数据"+ student.getId()+"--"+student.getName());
    }
}
```

### 随机发送两种对象,服务器判断并接收

参考代码位置 `nettyDemo1.com.ric.codec2`   

```proto
syntax = "proto3"; //版本
option optimize_for = SPEED; //加快解析
option java_package="com.ric.codec2";
option java_outer_classname="MyDataInfo";

//protobuf 可以使用message 管理其他 message
message MyMessage{
    //定义一个枚举
    enum DataType{
        StudentType = 0;
        WorkerType = 1;
    }

    //用datatype 来标识传的是哪一个枚举类型
    DataType data_type = 1;
    //标识每次枚举类型最多只能出现其中的一个,节省空间 下面两个类不能两个同时存在,不是student 就是 worker
    oneof dataBody {
        Student student = 2;
        Worker worker = 3;

    }
}
message Student{
    int32 id =1;
    string name =2;
}
message Worker{
    string name = 1;
    int32 age = 2;
}
```

## ⭐ Netty Handler机制

> 基本说明: Netty 主要组件有 `Channel` `EventLoop` `ChannelFuture` `ChannelHandler` `ChannelPipe` 

> ChannelHandler 充当了处理入站和出站数据的应用程序逻辑的容器 

### ⭐ 出站入站 机制
> 出站,入站 是相对客户端和服务端的名称  

- 客户端:
  - 发出到服务端为出站
  - 从服务端接收的为入站
- 服务端:
  - 发出到客户端为出站
  - 从客户端接收的为入站

> 出站入站对应 Handler 中的 Outbound类型的Handler 和Inbound类型Handler    
> 出站消息 需要进行编码(编码器是`ChannelOutboundHandlerAdapter` 的子类)   
> 入站消息 需要进行解码(解码器是 `ChannelInboundHandlerAdapter` 的子类)


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty21.png)

> 上图的重点:   
> ChannelPipeline**责任链**设计模式,对于客户端来说,从 `head` 处理读操作,从 `tail` 处理写操作  
> 对于客户端来说: 读消息(入站) 不会经过 `OutboundHandler` 处理, 写消息(出站) 不会经过 `InboundHandler` 处理     
> 还有一种 同时继承和实现了两种Handler `ChannelDuplexHandler extends ChannelInboundHandlerAdapter implements ChannelOutboundHandler`

### 编解码器
> Netty 发送或者接收一个消息时,就会发生一次数据转换(编解码),入站消息会被解码,出站消息会被编码   

> Netty 提供一系列使用的编解码器,他们都实现了 `ChannelInboundHandler` 或者 `ChannelOutboundHandler`,这些类中的 `channelRead` 方法已经被重写了      
> 以入站为例: 对于每个从入站 `Channel` 读取的消息,会调用 `channelRead` 随后他将由解码器所提供的 `decode()` 方法进行解码,   
> 并将已经解码的字节转发给 `ChannelPipeline` 中的下一个 `ChannelInboundHandler`   

举例 `io.netty.handler.codec.string.StringDecoder` 和 `io.netty.handler.codec.string.StringEncoder`  关系图   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/StringEncoder继承关系图.png)

#### 解码器示例 ReplayingDecoder
- `public abstract class ReplayingDecoder<S> extends ByteToMessageDecoder `
- `ReplayingDecoder` 扩展了 `ByteToMessageDecoder` 类,使用这个类不必调用 `readableBytes()` 方法,参数S指定了用户状态管理的类型,其中 `Void` 代表不需要状态管理
- 其他编解码器:
  - `LineBasedFrameDecoder` 使用行尾控制字符作为分隔符 `\n`或者`\r\n` 来解析数据 
  - `DelimiterBasedFrameDecoder` 使用自定义分隔符
  - `HttpObjectDecoder` Http 数据的解码器  
  - `LengthFieldFrameDecoder` 指定长度来标识整包消息,可以自动的处理粘包和半包消息   



### Netty Handler的调用机制

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/encoderDecoder.png)

> 不论是 `编码器handler` 还是 `解码器handler` 即接收消息类型必须与待处理的消息类型一致,否则该handler不会执行    
> 在解码器进行数据解码时,需要判断缓存区(ByteBuf) 数据是否足够,否则接收到的结果和期望结果不一致   

## Log4j整合Netty
- 在 Maven 中添加对 Log4j 的依赖在 pom.xml  

```java
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.25</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.25</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.7.25</version>
    <scope>test</scope>
</dependency>

```

- 配置 Log4j，在 resources/log4j.properties

```properties
log4j.rootLogger=DEBUG,stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=[%p]%C{1}-%m%n

```



## ⭐ 粘包拆包  

> TCP 是面向连接,面向流的,提供高可靠性服务,收发两端都要有一一成对的socket,发送端为了将多个发给接收端的包更有效的发给对方    
> 使用了优化算法 `Nagle` 算法,将多次间隔较小且数据量较小的数据,合并成一个大的数据块,然后进行封包.   
> 这样虽然提高了效率,但是接收端就难于分辨出完整的数据包了   
> 因为:**面向流的通信时无消息保护边界的**   


> 由于TCP无消息保护边界,需要在接收端处理消息编辑问题,也就是粘包,拆包问题   


示意图  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty22.png)

1. 正常情况,接收端分两次读取两个包
2. 粘包
3. 拆包
4. 拆包

### 解决方案 
- 使用自定义协议 + 编码器来解决  
- 关键是 **服务器端每次读取数据长度的问题**
- 可以通过自定义 传输类来限定包大小和内容(同时传入包内容及长度),然后自定义编解码器进行编解码 
- 简单的可以通过 上述解码器按照一定规则拆包解码
  - `LineBasedFrameDecoder`
  - `LengthFieldFrameDecoder` 
  - ...

自定义类举例 
```java
public class MessageProtocol {
    //长度
    private int len;
    //内容
    private byte[] content;
}
```  

自定义编码器
```java
public class MessageEncoder extends MessageToByteEncoder<MessageProtocol> {
    @Override
    protected void encode(ChannelHandlerContext channelHandlerContext, MessageProtocol messageProtocol, ByteBuf byteBuf) throws Exception {
        System.out.println("encode");
        byteBuf.writeInt(messageProtocol.getLen());
        byteBuf.writeBytes(messageProtocol.getContent());
    }
}
```

自定义解码器  
```java
public class MessageDecoder extends ReplayingDecoder<Void> {
    @Override
    protected void decode(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf, List<Object> list) throws Exception {
        System.out.println("decode调用");
        //二进制字节码转为 对象
        int i = byteBuf.readInt();
        byte[] bytes = new byte[i];
        byteBuf.readBytes(bytes);

        MessageProtocol messageProtocol = new MessageProtocol();
        messageProtocol.setLen(i);
        messageProtocol.setContent(bytes);

        list.add(messageProtocol);
    }
}
```

案例代码 `nettyDemo1.com.ric.tcp`    

## Netty 核心源码  



## 使用Netty实现RPC调用

> RPC 远程过程调用   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty23.png)


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/netty24.png)






















 












# 期间疑问汇总

1. `ctx.writeAndFlush()` `ctx.channel().writeAndFlush()` 区别 https://blog.csdn.net/fishseeker/article/details/78447684   
2. `ChannelInboundHandlerAdapter`  `SimpleChannelInboundHandler` 区别 出站 入站
3. `ChannelInitializer` 理解 流程   
4. `channelFuture.channel().closeFuture().sync(); channelFuture.channel().close().sync()`
5. `ChannelInboundHandlerAdapter`  `SimpleChannelInboundHandler`  `ChannelInitializer` 区别 和核心方法



# 参考
> - [尚硅谷视频](https://www.bilibili.com/video/BV1DJ411m7NR)  
> - [博客](https://www.jianshu.com/p/6681bfa36c4f)
> - [一张图搞清楚什么是入站和出站](https://www.jianshu.com/p/01b9b8cff943)
> - [粘包拆包参考](https://www.cnblogs.com/rickiyang/p/12904552.html)