# Socket 网络编程
计算机网络概述：

   利用通信线路将地理分散的、具有独立功能得计算机系统和通信设备按不同得形式连接起来，以功能完善得网络软件及协议实现资源共享和信息传递的系统。

计算机网络体系结构：

   网络数据的传输是以包的形式进行，其工作流程就是拼包和拆包。在数据传输过程中，包是全能性术语，除此还有帧用于表示数据链路层中包的单位、片是 IP 中数据的单位、段则表示 TCP 数据流中的信息、消息表示应用协议中数据的单位。首先在应用层获取应用协议传过来的数据、经过传输层附加 TCP 包首部、经过网络层附加IP包首部、再经过数据链路层附加以太网包首部、最后经过物理层传输到达目标点进行逆向解包，最终展示给用户。

[drawio](k83yzg19ZunrClTf2vON2tclUzTu-fdGMmkovdTSXa0.svg)

|TCP/IP 模型|OSI 模型|TCP/IP 协议族|功 能|
| ----- | ----- | ----- | ----- |
| |应用层|TFTP、HTTP、SNMP、FTP、DNS、Telnet|文件传输、电子邮件、文件服务、虚拟终端|
|应用层|表示层| |数据格式转换、代码转换、数据加密|
| |会话层| |接触或建立与别的接点的联系|
|传输层|传输层|TCP、UDP|提供端对端的接口|
|网络层|网络层|IP、ICMP、RIP、OSPF、BGP、IGMP|为数据包选择路由|
|链路层|链路层|SLIP、CSLIP、PPP、ARP、PARP、MTU|传输有地址的帧以及错误检测功能|
| |物理层| |以二进制数据形式在物理媒体上传输数据|

TCP/IP 协议族：

  TCP/IP 协议簇是 Internet 的基础，也是当今最流行的组网形式。TCP/IP 是一组协议的代名词，包括许多别的协议，组成了 TCP/IP 协议簇。其中比较重要的有 SLIP协议、PPP 协议、IP 协议、ICMP 协议、ARP 协议、TCP 协议、UDP 协议、FTP 协议、DNS 协议、SMTP 协议等。

协议介绍：

（1）.TCP :面向连接可靠的协议流( HTTP )，有以下优点：面向连接、可靠性、RTT (往返连接 Round-Trip Time )和 RTO (重传超时 Retransmission TimeOut )、一个数据包从一端发送到另一端收到回包后都会重新计算重传超时时间、数据排序、流量控制(滑动窗口、固定发送频率)、全双工。

（2）. UDP :面向无连接的通讯协议(在线视频)。

（3）. UDT :基于 UDP、在应用层自己实现连接、重传的机制( HTTP3 )。

TCP/IP 三次握手:

（1）.第一次握手:客户端向服务端发送一个 SYN 报文( TCP 报文中含有 SYN 标识位的报文、SYN=1 seq= 24316(序列号))、客户端进入存储在 TCP 中为 SYN\_SENT 状态。
（2）.第二次握手:服务收到客户端报文、得知客户端需要建立连接、服务端发出应答报文( SYN=1 ACK=1(标志位)、服务端进入SYN\_RECEIVED 状态 ack=24317(为客户端 seq+1、表示服务端收到客户端消息) seq=6478 (序列号))。
（3）. 第三次握手:客户端收到服务端报文、检查 ACK 是否等于1 ack 是否等于客户端 seq+1、而后发送 ACK 报文( ACK=1 seq=6479 )应答服务器、客户端进入连接、服务端收到报文也进入连接状态。

[drawio](R8eBWpDyWovF6eZl62PcSfk9Gugip2tOUs6gx1JNCdM.svg)

TCP/IP 四次挥手:

（1）.第一次挥手:客户端向服务端发送一个 FIN 报文( FIN=1 seq=28356(序列号))、客户端进入 FIN\_WAIT\_1 状态(客户端不再发送数据)。
（2）.第二次挥手:服务端收到客户端 FIN 报文、服务端发出应答报文 ACK ( ACK=1 seq=28357(序列号))，服务端进入 CLOSE\_WAIT 状态(知道客户端不再发数据)，客户端进入 FIN\_WAIT 状态。
（3）.第三次挥手:服务端将所有数据发送给客户端后、发送一个 FIN ( FIN=1 seq=783469)报文通知客户端，服务端进入 CLOSE 状态。服务端的 ACK 报文和 FIN 报文可能合并进行发送(服务端将所有数据发送完毕后)。
（4）.第四次挥手:客户端收到服务端FIN报文后进入TIME\_WAITING状态，并发出 ACK ( ACK=1 seq=783470) 应答报文，服务端收到客户端应答后进入 CLOSED 状态。
  客户端在进入 TIME\_WAITING 状态后在经过 2\*MSL (最长报文段寿命，存活最长时间 RFC : 2分钟)时间后才会进入 CLOSED 状态(①传输丢包情况下、保证服务端迟来的报文有足够的时间被识别和解析。②确保可靠的终止 TCP/IP 连接)。

网络通信：

   Socket 是应用层语 TCP/IP 协议族通信的中间软件抽象层、它是一组接口，其实是一个门面模式。TCP 用主机的IP地址加上主机上的端口号作为 TCP 连接的端点，这种端点就叫套接字( Socket )。

[drawio](7PpR9wuSpX5WCcddUOaucHWw4LgK6W3hBzTr5z-EOAk.svg)

原生 JDK 网络编程--BIO :

    阻塞式网络通讯( socket 没有读取到网络数据就阻塞当前线程)。服务端 BIO 实现 : 首先创建 ServerSocket 对象绑定当前端口、启动接口监听，然后接收客户端的连接并返回 Socket 对象，和客户端连接成功后创建一个新的线程进行阻塞式通信( bind() -> accept() ->开启一个 Thread { Socket-> Write 和 Read })，accept 是一个阻塞方法。客户端:获取服务端IP和端口号并用 socket 进行连接，然后进行数据的读写。

[drawio](vp5aJBwwU7u8_B48I04JBtWg2Zb79IZ4MCAUYpoDGu8.svg)

```kotlin
/** 客户端BIO实现 */
class SocketTest(private val socket: Socket) {
    lateinit var inetSocketAddress: InetSocketAddress
    private var inputStream: ObjectInputStream? = null
    private var outStream: ObjectOutputStream? = null
    fun socketTask() {
        try {
            inetSocketAddress = InetSocketAddress("127.0.0.1", 8080)
            socket.connect(inetSocketAddress)
            inputStream = ObjectInputStream(socket.getInputStream())
            outStream = ObjectOutputStream(socket.getOutputStream())
            outStream?.writeUTF("hello")
            outStream?.flush()
            inputStream?.readUTF()
        } catch (e: Exception) {
            e.printStackTrace()
        } finally {
            inputStream?.close()
            outStream?.close()
            socket.close()
        }
    }
}
```
原生 JDK 网络编程--NIO：

   多路复用非阻塞式编程(一个线程服务多个客户端)、其三大核心 : Selector 选择器、Channel 管道和 Buffer 缓冲区。服务端 NIO 实现:首先创建 Selector 对象和 ServerSocketChannel 对象、并让 ServerSocketChannel 在 Selector 中注册一个关注客户端连接的 SelectionKey 事件。当有客户端连接进来后，Selector 会通知 ServerSocketChannel 有客户端连接事件而后产生 SocketChannel 对象，SocketChannel 在 Selector 中注册网络上关注读 SelectionKey 事件。当客户端往服务器发送数据时，Selector 会通知相对应的 SocketChannel 有读事件进来，SocketChannel 获取网络数据后会将数据缓存在 Buffer 中，然后由 Buffer 输出到相应的业务逻辑处理程序，当有数据从业务逻辑产生时也会先缓存在Buffer中，在由 Buffer 输入到 SocketChannel 中推送给客户端。当有多个客户端连接时， ServerSocketChannel 会根据客户端产生相对应的 SocketChannel，并注册 SelectionKey 事件监听网络上的读事件，最后通过 Buffer 进行数据输入输出。

三大核心组件：

（1）.ServerSocketChannel : 绑定某个端口并接收连接(对应BIO中的 ServerSocket )。
（2）.Selector: ① ServerSocketChannel 在 Selector 注册客户端的连接 SelectionKey 事件 ② 客户端连接服务端、Selector 感知客户端连接事件后、通知  ServerSocketChannel 和客户端进行连接。
（3）.SocketChannel : ServerSocketChannel 连接成功后产生 SocketChannel，并在 Selector 注册关注读或写的 SelectionKey 事件(对应 BIO 中 Socket )。

Buffer 缓存：

   客户端请求服务器进行读写时，Selector 感知读写事件通知 SocketChannel 进行读、写并把数据写入 Buffer 中、逻辑程序在 Buffer 中读、写。Buffer 有三个重要属性: capacity、position、limit。Buffer 进行数据写入时，position 代表内存下一个写入的位置、limit 表示可以写入的最大长度(一般来说 limit 等于capacity)。当 SocketChannel 从 Buffer 中读取数据时、Buffer 中有个读写模式切换的 flip() 函数、Buffer 切换到读模式 position 的位置会移动到数据开始位置、而 limit 则移动到写入数据时 position 位置、最后将数据从 Buffer 中读出并写入 SocketChannel 中。

[drawio](tf7hsgb2g0S4BGrQz0Yj7uuJIgpmDDvZg29inXGoJOg.svg)

```kotlin
/** 类说明：nio通信客户端处理器 */
class NioClientHandle(private val host: String, private val port: Int) : Runnable {
    @kotlin.jvm.Volatile
    private var started = false
    private var selector: Selector? = null
    private var socketChannel: SocketChannel? = null
    fun stop() {
        started = false
    }

    override fun run() {
        try {
            doConnect()
        } catch (e: IOException) {
            e.printStackTrace()
            exitProcess(1)
        }

        while (started) {
            try {
                /*无论是否有读写事件发生，selector每隔1s被唤醒一次*/
                selector?.select(1000)
                /*获取当前有哪些事件可以使用*/
                val keys: Set<SelectionKey> = selector?.selectedKeys()?:return
                val it: Iterator<SelectionKey> = keys.iterator()
                var key: SelectionKey? = null
                while (it.hasNext()) {
                    key = it.next()
                    /*我们必须首先将处理过的 SelectionKey 从选定的键集合中删除。如果我们没有删除处理过的键，
                    那么它仍然会在主集合中以一个激活的键出现，这会导致我们尝试再次处理它*/
                    it.remove()
                    try {
                        handleInput(key)
                    } catch (e: Exception) {
                        if (key != null) {
                            key.cancel()
                            if (key.channel() != null) {
                                key.channel().close()
                            }
                        }
                    }
                }
            } catch (e: Exception) {
                e.printStackTrace()
                exitProcess(1)
            }
        }
        /*selector关闭后会自动释放里面管理的资源*/
        if (selector != null) try {
            selector?.close()
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }

    @kotlin.Throws(IOException::class)
    private fun doConnect() {
       /*connect非阻塞，当他返回的时候，连接不一定完成了，如果返回值为true，表示连接完成、
       返回值为false，表示连接没完成，还在三次握手的过程中*/
        if (socketChannel?.connect(InetSocketAddress(host, port))==true) {
            socketChannel?.register(selector, SelectionKey.OP_READ)
        } else {
            /*selector要告诉我，连接以及完成*/
            socketChannel?.register(selector, SelectionKey.OP_CONNECT)
        }
    }

    @kotlin.Throws(IOException::class)
    private fun handleInput(key: SelectionKey) {
        if (key.isValid) {
            /*获得关心当前事件的channel*/
            val sc: SocketChannel = key.channel() as SocketChannel
            if (key.isConnectable) {
                if (sc.finishConnect()) {
                    socketChannel?.register(selector, SelectionKey.OP_READ)
                } else {
                    exitProcess(1)
                }
            }
            if (key.isReadable) {
                /*创建ByteBuffer，并开辟一个1M的缓冲区*/
                val buffer: ByteBuffer = ByteBuffer.allocate(1024)
                /*读取请求码流，返回读取到的字节数*/
                val readBytes: Int = sc.read(buffer)
                /*读取到字节，对字节进行编解码*/
                if (readBytes > 0) {
                    /*将缓冲区当前的limit设置为position，position=0，用于后续对缓冲区的读取操作*/
                    buffer.flip()
                    /*根据缓冲区可读字节数创建字节数组*/
                    val bytes = ByteArray(buffer.remaining())
                    /*将缓冲区可读字节数组复制到新建的数组中*/
                    buffer.get(bytes)
                    val result = String(bytes)
                    println("客户端收到消息：$result")
                } else if (readBytes < 0) {
                    key.cancel()
                    sc.close()
                }
            }
        }
    }

    @kotlin.Throws(IOException::class)
    private fun doWrite(channel: SocketChannel?, request: String) {
        /*将消息编码为字节数组*/
        val bytes: ByteArray = request.toByteArray()
        /*根据数组容量创建ByteBuffer*/
        val writeBuffer: ByteBuffer = ByteBuffer.allocate(bytes.size)
        /*将字节数组复制到缓冲区*/
        writeBuffer.put(bytes)
        writeBuffer.flip()
        /*发送缓冲区的字节数组、关心事件和读写网络并不冲突*/
        channel?.write(writeBuffer)
    }

    /*写数据对外暴露的API*/
    @kotlin.Throws(Exception::class)
    fun sendMsg(msg: String) {
        doWrite(socketChannel, msg)
    }

    init {
        try {
            /*创建选择器的实例*/
            selector = Selector.open()
            /*创建ServerSocketChannel的实例*/
            socketChannel = SocketChannel.open()
            /*设置通道为非阻塞模式*/
            socketChannel?.configureBlocking(false)
            started = true
        } catch (e: IOException) {
            e.printStackTrace()
        }
    }
}
```
