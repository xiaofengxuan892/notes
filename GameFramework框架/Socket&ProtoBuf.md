[TOC]



### Socket通信：

#### 建立Socket连接通道：

##### 1).创建Socket对象：

```c#
Socket m_Socket = new Socket(ipAddress.AddressFamily, SocketType.Stream, ProtocolType.Tcp);
```

"ipAddress.AddressFamily"代表该socket通道的IPAddress的地址类型：“AddressFamily.InterNetwork” 代表“IPv4”，“AddressFamily.InterNetworkV6”代表“IPv6”

##### 2).使用socket.BeginConnect建立连接：

`IAsyncResult BeginConnect(IPAddress address, int port, AsyncCallback requestCallback, object state)`。该方法返回IAsyncResult异步类型变量，“state”通常可自由传递参数，但由于需要在“requestCallback”中使用“socket.EndConnect”等待“异步连接完成”，因此通常直接将该“m_Socket变量”进行传递(由于该参数为“object类型”，因此也可封装新类型 —— 该类型中包含Socket类型参数，然后使用该类型的实例对象以传递更多信息)

```c#
IAsyncResult connectResult = clientSocket.BeginConnect(ipAddress, port, m_ConnectCallback, m_Socket);
```

##### 3).在AsyncCallback中使用“socket.EndConnect(IAsyncResult ia)”等待“异步连接操作完成”：

```c#
m_Socket.EndConnect(connectResult);
```

**注意**：“EndConnect”并不是“终止Socket连接”，而是等待“BeginConnect”方法发起的“异步连接”完成

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231004222113730.png" alt="image-20231004222113730"  />

**<font color=red>真正可以“终止或结束某个Socket连接”的方法是“socket.Close()”</font>**，该方法作用如下：

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231004223727003.png" alt="image-20231004223727003"  />

但**<font color=red>使用Close方法关闭socket连接后，则不能再使用该socket进行通信或执行其他操作</font>**，只能再重新创建一个新的Socket对象才行



#### 使用Socket通道发送消息：

发送消息主要使用“BeginSend”和“EndSend”配合完成：

```c#
public class SocketAsyncState
{
    public Socket Socket { get; set; }
    public byte[] Buffer { get; set; }
}

..........................
// 创建一个客户端 Socket
Socket clientSocket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
// 连接到服务器
clientSocket.BeginConnect("127.0.0.1", 8888, ConnectCallback, null);

private static void ConnectCallback(IAsyncResult ar)
{
    // 完成连接操作
    clientSocket.EndConnect(ar);

    // 创建消息并转换为字节数组
    string message = "Hello, server!";
    byte[] buffer = Encoding.ASCII.GetBytes(message);

    // 创建异步状态对象
    SocketAsyncState state = new SocketAsyncState
    {
        Socket = clientSocket,
        Buffer = buffer
    };

    // 开始异步发送数据
    clientSocket.BeginSend(buffer, 0, buffer.Length, SocketFlags.None, SendCallback, state);
}

private static void SendCallback(IAsyncResult ar)
{
    // 获取异步状态对象
    SocketAsyncState state = (SocketAsyncState)ar.AsyncState;

    // 完成发送操作
    int bytesSent = state.Socket.EndSend(ar);
    Debug.LogFormat("Sent {0} bytes to server.", bytesSent);

    // 关闭 Socket 连接
    state.Socket.Shutdown(SocketShutdown.Both);
    state.Socket.Close();
}
```



#### 使用Socket通道接收消息：

**socket通道接收消息时采用“socket.BeginReceive”和“socket.EndReceive”配合完成**。详情如下：

##### BeginReceive和EndReceive：

`BeginReceive(byte[] buffer, int offset, int size, SocketFlags socketFlags, AsyncCallback callback, object state)`

<img src="https://gitee.com/kakaix892/image-host/raw/main/Typora/image-20231005170438044.png" alt="image-20231005170438044" style="zoom:80%;" />

示例说明：

```c#
public class StateObject
{
    public Socket workSocket = null;
    public const int BufferSize = 1024;
    public byte[] buffer = new byte[BufferSize];
}

.................
//开始异步接收消息
StateObject state = new StateObject(); //新创建StateObject用于存储接收到的消息包数据
state.workSocket = m_Socket;
m_Socket.BeginReceive(state.buffer, 0, StateObject.BufferSize, SocketFlags.None, new AsyncCallback(ReceiveCallback), state);

public static void ReceiveCallback(IAsyncResult ar)
{
    // 从 StateObject 中获取异步操作的状态和 socket
    StateObject state = (StateObject)ar.AsyncState;
    Socket handler = state.workSocket;

    // 结束异步接收，并获取实际接收到的字节数
    int bytesRead = handler.EndReceive(ar);

    if (bytesRead > 0)
    {
        // 处理接收到的数据，将其转换成string形式输出
        string receivedData = Encoding.ASCII.GetString(state.buffer, 0, bytesRead);
        Console.WriteLine("Received data: " + receivedData);

        // 继续异步接收客户端发送的数据
        handler.BeginReceive(state.buffer, 0, StateObject.BufferSize, SocketFlags.None, new AsyncCallback(ReceiveCallback), state);
    }
    else
    {
        // 客户端关闭连接
        handler.Close();
    }
}
```





### ProtoBuf序列化和反序列化数据包：

**<font color=red>序列化和反序列化通常使用“ProtoBuf.Meta.RuntimeTypeModel”中的方法实现</font>**，但ProtoBuf内部已经进行了封装，即`Serializer.cs`，**<font color=red>其内部包含“Serialize —— 序列化header数据”， “SerializeWithLengthPrefix —— 序列化实际数据”，“Deserialize —— 反序列化Header数据”，“DeserializeWithLengthPrefix —— 反序列化实际数据”</font>**，共4个方法。**在(反)序列化数据包时，可以直接使用该方法，效果相同**

#### 序列化数据包中的Header和真实数据：

##### 序列化Header数据：

`RuntimeTypeModel.Default.Serialize(Stream destination, object instance)`， instance代表需要被实例化的对象本身

**使用方式**：

```c#
CSPacketHeader header = ReferencePool.Acquire<CSPacketHeader>();
RuntimeTypeMode.Default.Serialize(m_CachedStream, header);
//将实例对象header序列化到“m_CachedStream”中
```

**PS**：**<font color=red>ProtoBuf内部已提供专用的序列化Header的方法“ProtoBuf.Serializer.Serialize”</font>**(**该方法内部通过封装RuntimeTypeModel.Default.Serialize得到**)，因此也可以直接该方法序列化header数据

##### 序列化“真实数据”：

`RuntimeTypeModel.Default.SerializeWithLengthPrefix(Stream destination, object instance, Type type, PrefixStyle style, int fieldNumber)`

**使用方式**：

```c#
//“packetData”代表需要序列化的“真实数据”对象本身
RuntimeTypeModel model = RuntimeTypeModel.Default;
model.SerializeWithLengthPrefix(m_CachedStream, packetData, model.MapType(typeof(packetData)), PrefixStyle.Fixed32, 0);
```

**PS**：**<font color=red>ProtoBuf内部已提供专用的序列化“真实数据”的方法“ProtoBuf.Serializer.SerializeWithLengthPrefix”</font>**(**该方法内部通过封装RuntimeTypeModel.Default.SerializeWithLengthPrefix得到**)



#### 2.反序列化数据包Header和真实数据：

##### 反序列化Header数据：

`RuntimeTypeModel.Default.Deserialize(Stream source, object value, Type type)`

**使用方式**：

```c#
//“SCPacketHeader”代表“从服务器向客户端发送消息的Header”的一种实际的类型
//“PacketHeaderBase”代表“服务器和客户端相互发送消息时用到的Header”的基类，“SCPacketHeader”和“CSPacketHeader”均是继承“PacketHeaderBase”而来
SCPacketHeader headerInstance = ReferencePool.Acquire<SCPacketHeader>();
PacketHeaderBase header = (PacketHeaderBase)RuntimeTypeModel.Default.Deserialize(sourceStream, headerInstance, typeof(SCPacketHeader));
```

##### 反序列化“真实数据”(数据包中Header之后的数据)：

`RuntimeTypeModel.Default.DeserializeWithLengthPrefix(Stream source, object value,  Type type, PrefixStyle style, int fieldNumber)`

**使用方式**：

```c#
//首先获取到该数据包中“真实数据”的类型，例如“心跳协议”中“服务器的返回数据类型”为“SCHeartBeat”
Type packetType = GetServerToClientPacketType(....); //主要在于获取“真实数据的类型”
object packetDataInstance = ReferencePool.Acquire(packetType);

//反序列化数据
PacketBase packetData = (PacketBase)RuntimeTypeModel.Default.DeserializeWithLengthPrefix(sourceStream, packetDataInstance, packetType, PrefixStyle.Fixed32, 0);
```



#### 问题：在使用RuntimeTypeModel.Default.Deserialize执行反序列化后，stream中的数据会被删除吗？

**解答**：RuntimeTypeModel.Default.Deserialize方法是读取stream中的二进制数据，并根据protobuf编码来解析数据，然后返回相应的对象，即“反序列化”。该过程并不会修改原始stream中的数据，更不会删除stream中的内容。

如果需要删除stream中的数据，可以按照以下操作：首先使用stream.Position属性来定位流的位置，然后使用SetLength来截断流的长度

以下案例演示如何在“反序列化”后清除流中的数据：

```c#
// 创建一个示例对象
ExampleClass example = new ExampleClass { Id = 1, Name = "Example" };

// 将示例对象序列化到流中
MemoryStream stream = new MemoryStream();
Serializer.Serialize(stream, example);

// 反序列化对象
stream.Position = 0; // 将流的位置设置为起始位置
ExampleClass deserializedExample = (ExampleClass)RuntimeTypeModel.Default.Deserialize(stream, null, typeof(ExampleClass));

// 清空流中的数据
stream.SetLength(0);

// 输出反序列化后的对象
Console.WriteLine($"Deserialized object: Id = {deserializedExample.Id}, Name = {deserializedExample.Name}");
```

**解析**：由以上代码可知，在设置`stream.Position = 0`后反序列化该stream**<font color=red>并不会对该stream有任何影响</font>**，**<font color=red>包括该stream.Position的位置也没有任何改变</font>**，之后直接使用“stream.SetLength(0)”即可截断该stream

**注意**：**<font color=red>在序列化过程中，即使用ProtoBuf.Serializer.Serialize或RuntimeTypeModel.Default.Serialize方法执行序列化后，stream中的数据会被改变</font>**





















