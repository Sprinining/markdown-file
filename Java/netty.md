# Netty

## Buffer

```java
IntBuffer intBuffer = IntBuffer.allocate(5);

for (int i = 0; i < intBuffer.capacity(); i++) {
    intBuffer.put(i);
}

// 读写切换
intBuffer.flip();
while (intBuffer.hasRemaining()) {
    // 每次get，指针都会后移
    System.out.println(intBuffer.get());
}
```

### 类型化放入

```java
public static void main(String[] args) throws Exception {
    ByteBuffer buffer = ByteBuffer.allocate(64);

    // 类型化方式放入数据
    buffer.putInt(100);
    buffer.putChar('*');
    buffer.putLong(1L);

    buffer.flip();

    // 取出的顺序必须一致
    System.out.println(buffer.getInt());
    System.out.println(buffer.getChar());
    System.out.println(buffer.getLong());
}
```

### 只读

```java
public static void main(String[] args) throws Exception {
    ByteBuffer buffer = ByteBuffer.allocate(64);

    for (int i = 0; i < buffer.capacity(); i++) {
        buffer.put((byte) i);
    }

    buffer.flip();

    // 只读
    ByteBuffer readOnlyBuffer = buffer.asReadOnlyBuffer();
    while (readOnlyBuffer.hasRemaining()) {
        System.out.println(readOnlyBuffer.get());
    }

    // 会抛异常java.nio.ReadOnlyBufferException
    readOnlyBuffer.put((byte) 1);
}
```

### MappedByteBuffer

```java
public static void main(String[] args) throws Exception {
    RandomAccessFile randomAccessFile = new RandomAccessFile("file", "rw");
    FileChannel channel = randomAccessFile.getChannel();

    /**
     * 可让文件直接在堆外内存修改，不需要OS拷贝一次
     * 读写模式，可以直接修改的起始位置，映射到内存的大小（字节）
     * 可以直接修改的位置[0,5)
     * 实际类型是DirectByteBuffer
     */
    MappedByteBuffer map = channel.map(FileChannel.MapMode.READ_WRITE, 0, 5);
    map.put(0, (byte) 'X');
    map.put(4, (byte) 'Y');
    randomAccessFile.close();
}
```

### Buffer数组

```java
public static void main(String[] args) throws Exception {
    ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
    InetSocketAddress inetSocketAddress = new InetSocketAddress(8888);
    // 绑定端口并启动
    serverSocketChannel.socket().bind(inetSocketAddress);

    ByteBuffer[] byteBuffers = new ByteBuffer[2];
    byteBuffers[0] = ByteBuffer.allocate(5);
    byteBuffers[1] = ByteBuffer.allocate(3);

    SocketChannel socketChannel = serverSocketChannel.accept();
    int count = 8;
    while (true) {
        int byteRead = 0;
        while (byteRead < count) {
            long l = socketChannel.read(byteBuffers);
            byteRead += l;
            System.out.println("byteRead:" + l);
            // 流打印
            Arrays.stream(byteBuffers)
                    .map(buffer -> "position:" + buffer.position()
                            + ", limit:" + buffer.limit())
                    .forEach(System.out::println);
        }

        // flip
        Arrays.asList(byteBuffers)
                .forEach(ByteBuffer::flip);

        // 读出数据显示在客户端
        long byteWrite = 0;
        while (byteWrite < count) {
            long l = socketChannel.write(byteBuffers);
            byteWrite += l;
        }

        // clear
        Arrays.asList(byteBuffers)
                .forEach(ByteBuffer::clear);
    }
}
```

## Channel

### ByteBuffer

```java
public static void main(String[] args) throws Exception {
    FileInputStream fileInputStream = new FileInputStream("file");
    FileChannel channel1 = fileInputStream.getChannel();

    FileOutputStream fileOutputStream = new FileOutputStream("file.txt");
    FileChannel channel2 = fileOutputStream.getChannel();

    ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
    while (true) {
    /*    public Buffer clear() {
            position = 0;
            limit = capacity;
            mark = -1;
            return this;
        }*/
        byteBuffer.clear();

        // 从channel读出，放入到byteBuffer
        int read = channel1.read(byteBuffer);
        if (read == -1) {
            break;
        }

        // 将byteBuffer写入channel2
        byteBuffer.flip();
        channel2.write(byteBuffer);
    }

    fileInputStream.close();
    fileOutputStream.close();
}
```

### transferTo&transferFrom

```java
public static void main(String[] args) throws Exception {
    FileInputStream fileInputStream = new FileInputStream("file");
    FileChannel sourceChannel = fileInputStream.getChannel();

    FileOutputStream fileOutputStream = new FileOutputStream("file.txt");
    FileChannel destChannel = fileOutputStream.getChannel();

    // destChannel.transferFrom(sourceChannel, 0, sourceChannel.size());
    sourceChannel.transferTo(0, sourceChannel.size(), destChannel);

    fileInputStream.close();
    fileOutputStream.close();
}
```

## Selector

- 一个EventLoopGroup 包含一个或者多个EventLoop；
- 一个EventLoop 在它的生命周期内只和一个Thread 绑定；
- 所有由EventLoop 处理的I/O 事件都将在它专有的Thread 上被处理；
- 一个Channel 在它的生命周期内只注册于一个EventLoop；
- 一个EventLoop 可能会被分配给一个或多个Channel。



- 有客户端连接时，会通过ServerSocketChannel得到SocketChannel
- 将SocketChannel注册到Selector上，`register(Selector sel, int ops, Object att)`，返回SelectionKey，会和该Selector上关联(集合)。一个Selector上可以注册多个SocketChannel
- Selector会用select方法进行监听，返回发生事件的通道个数
- 得到有事件发生的Channel的SelectionKey，然后得到该Channel

```java
selector.select(); // 阻塞
selector.select(100); // 阻塞100ms后返回
selector.wakeup(); // 唤醒
selector.selectNow();// 不阻塞，立刻返回
```

### 服务器

```java
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.nio.charset.StandardCharsets;
import java.util.Iterator;
import java.util.Set;

class NIOServer {
    public static void main(String[] args) throws Exception {
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

        Selector selector = Selector.open();

        serverSocketChannel.socket().bind(new InetSocketAddress(7899));
        // 设为非阻塞
        serverSocketChannel.configureBlocking(false);
        // 把serverSocketChannel注册到selector上，关心事件为OP_ACCEPT
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        // 等待连接
        while (true) {
            if (selector.select(1000) == 0) {
                System.out.println("服务器等待1s，无连接");
                continue;
            }

            // 返回关注事件的集合
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            // 迭代器遍历
            Iterator<SelectionKey> iterator = selectionKeys.iterator();
            while (iterator.hasNext()) {
                // 获取SelectionKey
                SelectionKey key = iterator.next();

                // 根据事件做相应处理
                if (key.isAcceptable()) {
                    // 新客户端连接
                    // accept实际不会阻塞，因为已经知道是OP_ACCEPT事件
                    SocketChannel socketChannel = serverSocketChannel.accept();
                    socketChannel.configureBlocking(false);
                    System.out.println("客户端连接成功" + socketChannel.hashCode());
                    // 注册到selector，关心事件为OP_READ，同时关联一个buffer
                    socketChannel.register(selector, SelectionKey.OP_READ, ByteBuffer.allocate(10));
                }

                if (key.isReadable()) {
                    // 通过key反向获取对应的Channel
                    SocketChannel channel = (SocketChannel) key.channel();
                    // 通过key获取Buffer
                    ByteBuffer byteBuffer = (ByteBuffer) key.attachment();
                    channel.read(byteBuffer);
                    System.out.println("客户端发来：" + new String(byteBuffer.array(), StandardCharsets.UTF_8));
                }

                // 从集合移动当前selectionKey，防止重复操作
                iterator.remove();
            }

        }
    }
}
```

### 客户端

```java
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;
import java.nio.charset.StandardCharsets;

class NIOClient {
    public static void main(String[] args) throws Exception {
        SocketChannel socketChannel = SocketChannel.open();
        // 设置非阻塞
        socketChannel.configureBlocking(false);
        InetSocketAddress inetSocketAddress = new InetSocketAddress("127.0.0.1", 7899);
        if (!socketChannel.connect(inetSocketAddress)) {
            while (!socketChannel.finishConnect()) {
                System.out.println("连接中，客户端不会阻塞，可以做其他事");
            }
        }

        // 连接成功后
        ByteBuffer byteBuffer = ByteBuffer.wrap("haha".getBytes(StandardCharsets.UTF_8));
        // 将buffer写入channel
        socketChannel.write(byteBuffer);
        System.in.read();
    }
}
```

