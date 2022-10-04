# WebSocket基本原理

结合Go或Java WebSocket库源码以及wireshark抓取的数据包理解。

源码包：

go: golang.org/x/net/websocket
Java: javax.websocket-api

抓包：

先启动本地WebSocket服务端（port:9002）。

抓包接口: Loopback:io 
抓包过滤器：port 9002
显示过滤器：暂不设置



## WebSocket 与 HTTP、TCP

HTTP是请求-响应模式的通信协议。

HTTP1.0 每个请求都需要一个单独的连接。

HTTP1.1 引入可重用连接（长连接，Connection： keep-alive，默认开启）在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟（其他优化和本文关系不大，暂不赘述）。

HTTP是半双工通信，同一时刻数据只能单向流动，服务端无法主动建立连接发送数据。这种特点在某些实时场景（比如请求获取一个需要一定时间才能返回到结果；或者获取某个不定时发生的事件信息，如股价变动、天气变化、web网页游戏状态刷新等），就很难处理，HTTP有几种方式解决问题：轮询、长轮询、流化，但是没有很完美的解决方法。

> 轮询：隔一段时间请求一次看看有没有想要的数据；
>
> 长轮询：连接保持较长时间不关闭，等待直到服务端返回数据或者超时；
>
> 流化：客户端发送一个请求，服务器发送并维护一个持续更新和保持打开（可以是无限或者规定的时间段）的开放响应。每当服务器有需要交付给客户端的信息时，它就更新响应。由于服务器从不发出完成HTTP响应的请求，因此连接一直保持打开。

为解决HTTP上述实时通信问题，引入了WebSocket协议，是全双工、双向、单套接字的连接，也是基于TCP的协议。

WebSocket相较于HTTP是单连接的，可以节约带宽、CPU资源、减少延迟、通信复杂度更低。

> 更多比较：WebRTC 、HTTP2.0。

WebSocket第一次是Http握手，握手成功之后才是WebSocket通信，WebSocket通信简言之主要就是基于WebSocket帧协议的数据编解码以及TCP数据传输。

> 为何WebSocket还要使用HTTP进行一次握手，而不是直接建立TCP连接？
>
> TODO？

## WebSocket 协议

### 连接握手

测试是用的Postman发起连接的`ws://127.0.0.1:9002`，客户端编码实现参考各WebSocket实现包的接口API文档。

![](../docs/imgs/连接握手.png)

**前3行**：TCP 3 次握手；

**第4行**：客户端发起WebSocket连接握手(HTTP/1.1协议的请求)；

握手数据：

```properties
Hypertext Transfer Protocol
    GET / HTTP/1.1\r\n
    Sec-WebSocket-Version: 13\r\n
    Sec-WebSocket-Key: pNgZr2dChVwcNts30DxZiQ==\r\n
    Connection: Upgrade\r\n
    Upgrade: websocket\r\n
    Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits\r\n
    Host: 127.0.0.1:9002\r\n
```

> 握手请求选项
>
> Sec-WebSocket-Protocol （子协议选择）、Sec-WebSocket-Extensions （拓展列表）?
>
> Origin: WebSocket 客户端是Web浏览器时需要提供此字段，用于同源策略检查。

**第5行**：ACK消息，即对前一行消息的确认消息；

**第6行**：握手成功；

握手成功返回数据：

```properties
Hypertext Transfer Protocol
    HTTP/1.1 101 Switching Protocols\r\n		# 101 状态码
    upgrade: websocket\r\n
    connection: upgrade\r\n
    sec-websocket-accept: W3UNJiLyRMXlwQ2lsHe8hNFegRU=\r\n
```

**第7行**：前一行的确认消息。

详细参考：RFC6455第4章节。

#### 安全模型

WebSocket协议在web网页中应用时基于web浏览器的同源策略模型；非浏览器页面的客户端中使用时可以接受任意源数据。

#### URI规范

```
ws-URI = "ws:" "//" host [ ":" port ] path [ "?" query ]
wss-URI = "wss:" "//" host [ ":" port ] path [ "?" query ]
```

wss是指整合了TLS的WebSocket协议，默认的"ws"端口是80，而默认的"wss"端口是443。。

> WebSocket URI 的 path、query 都有什么用？项目中只在节点负载均衡的时候用过。

#### 使用代理



#### TLS握手



### 数据传输

#### 数据帧

帧协议格式：

      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-------+-+-------------+-------------------------------+
     |F|R|R|R| opcode|M| Payload len |    Extended payload length    |
     |I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
     |N|V|V|V|       |S|             |   (if payload len==126/127)   |
     | |1|2|3|       |K|             |                               |
     +-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
     |     Extended payload length continued, if payload len == 127  |
     + - - - - - - - - - - - - - - - +-------------------------------+
     |                               |Masking-key, if MASK set to 1  |
     +-------------------------------+-------------------------------+
     | Masking-key (continued)       |          Payload Data         |
     +-------------------------------- - - - - - - - - - - - - - - - +
     :                     Payload Data continued ...                :
     + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
     |                     Payload Data continued ...                |
     +---------------------------------------------------------------+
+ 操作码（opcode）

   %x0 表示一个持续帧；表明这是一个数据分片；

   %x1 表示一个文本帧；表明这是一个文本协议编码的数据帧；

   %x2 表示一个二进制帧；表明这是一个二进制协议编码的数据帧；

   %x3-7 预留给以后的非控制帧；

   %x8 表示一个连接关闭包；

   %x9 表示一个ping包；

   %xA 表示一个pong包；

   %xB-F 预留给以后的控制帧。

  > 二进制编码 vs 文本协议编码

+ 掩码（Masking-key）

  为了防止早期版本的协议中存在的代理缓存污染攻击（proxy cache poisoning attacks）等问题。原理（TODO）？

#### 控制帧

+ 关闭帧（Close）

+ Ping / Pong

  Ping / Pong 是 WebSocket 的心跳。

#### 消息收发

建立连接后使用postman发送一个消息，如`{"userId":10001,"code":1001,"data":{}}`。

```
WebSocket
    1... .... = Fin: True						//这是消息的最后一个片段，由于数据较短这个消息只有一个片段
    .000 .... = Reserved: 0x0		
    .... 0001 = Opcode: Text (1)		//这是一个文本帧
    1... .... = Mask: True					//“有效负载数据”有添加掩码，“有效负载数据”是指“扩展数据”和“应用数据”
    .011 1001 = Payload length: 57	//“有效负载数据”的长度
    Masking-Key: a0fe8b23						//32bit的掩码
    Masked payload									//经过掩码编码后的Payload
    Payload
Line-based text data (5 lines)
    {\n
        "userId": 10001,\n
        "code": 1001,\n
        "data": {}\n
    }
```



### Ping/Pong & Keep-Alive & 心跳保活

Ping / Pong 用于 WebSocket 的心跳保活。

注意区分HTTP Keep-Alive机制与TCP Keep-Alive机制，还有心跳保活的区别。

HTTP Keep-Alive 主要是说**连接复用**；TCP Keep-Alive是指**TCP协议本身的心跳包机制**（配置：SO_KEEPALIVE），开启后通过抓包工具可以抓到对应的心跳包；心跳保活则更多地指**应用层自定义的发送心跳保持活跃连接**。

```
164	1314.700264	::1	::1	TCP	64	[TCP Keep-Alive] 63201 → 9002 [ACK] Seq=228 Ack=158 Win=407616 Len=0
165	1314.700319	::1	::1	TCP	76	[TCP Keep-Alive ACK] 9002 → 63201 [ACK] Seq=158 Ack=229 Win=1048576 Len=0 TSval=915809934 TSecr=915254833
```

**TCP Keep-Alive原理**：通过socket接口方法 setsockopt() 将 SOL_SOCKET.SO_KEEPALIVE 设置为1开启TCP心跳包机制，可以设置三个参数tcp_keepalive_time / tcp_keepalive_probes / tcp_keepalive_intvl，分别表示连接闲置多久开始发keepalive的ACK包、发几个ACK包不回复才当对方死了、两个ACK包之间间隔多长。连接闲置到指定时间（tcp_keepalive_time ）TCP协议会向对方发一个带有ACK标志的空数据包（KeepAlive探针），对方在收到ACK包以后，如果连接一切正常，应该回复一个ACK；如果连接出现错误了（例如对方重启了，连接状态丢失），则应当回复一个RST；如果对方没有回复，服务器每隔 tcp_keepalive_intvl 时间再发ACK，如果连续tcp_keepalive_probes 个包都被无视了，说明连接被断开了。连接异常或断开后会关闭连接。

TCP Keep-Alive机制，设计初衷自动关闭死亡的连接，机制已经固定灵活性受限，不适合实时性高的场合；很多资料都不推荐开启，应该在应用层自行实现心跳保活机制。

> 1）开启TCP Keep-Alive后，客户端发起的[TCP Keep-Alive]心跳包，为何 WireShark 可以抓到？Netty ChannelPipeline 抓不到？
>
> 推测是因为Keep-Alive是TCP内部机制，心跳包既然是给内部机制处理的，当然不需要再传给应用层。
>
> 2）服务端挂了，客户端的 TCP 连接还在吗？
>
> 首先看有没有自定义心跳保活、有没有开启TCP Keep-Alive机制；如果有自定义心跳保活，服务端挂掉后，客户端根据自定义心跳保活策略处理；如果有开启了TCP Keep-Alive，会根据Keep-Alive机制处理。都没有的话客户端TCP连接会一直存在。



### 关闭连接

测试中服务端30s没有收到消息就断开连接。

![](../docs/imgs/断开连接.png)

就只是TCP的四次挥手，这里是服务端触发的，和书上怎么有点不一样。

#### 状态码

| 状态码 | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| 1000   | 表示一个正常的关闭，意味着连接建立的目标已经完成了。         |
| 1001   | 表示终端已经“走开”，例如服务器停机了或者在浏览器中离开了这个页面。 |
| 1002   | 表示终端由于协议错误中止了连接。                             |
| 1003   | 表示终端由于收到了一个不支持的数据类型的数据（如终端只能怪理解文本数据，但是收到了一个二进制数据）从而关闭连接。 |
| 1004   | 保留字段。                                                   |
| 1005   | 是一个保留值并且不能被终端当做一个关闭帧的状态码。这个状态码是为了给上层应用表示当前没有状态码。 |
| 1006   | 是一个保留值并且不能被终端当做一个关闭帧的状态码。这个状态码是为了给上层应用表示连接被异常关闭如没有发送或者接受一个关闭帧这种场景的使用而设计的。 |
| 1007   | 表示终端因为收到了类型不连续的消息（如非 UTF-8 编码的文本消息）导致的连接关闭。 |
| 1008   | 表示终端是因为收到了一个违反政策的消息导致的连接关闭。这是一个通用的状态码，可以在没有什么合适的状态码（如 1003 或者 1009）时或者可能需要隐藏关于政策的具体信息时返回。 |
| 1009   | 表示终端由于收到了一个太大的消息无法进行处理从而关闭连接。   |
| 1010   | 表示终端（客户端）因为预期与服务端协商一个或者多个扩展，但是服务端在 WebSocket 握手中没有响应这个导致的关闭。需要的扩展清单应该出现在关闭帧的`原因（reason）`字段中。 |
| 1011   | 表示服务端因为遇到了一个意外的条件阻止它完成这个请求从而导致连接关闭。 |
| 1015   | 是一个保留值，不能被终端设置到关闭帧的状态码中。这个状态码是用于上层应用来表示连接失败是因为 TLS 握手失败（如服务端证书没有被验证过）导致的关闭的。 |



## 参考

协议官方RFC文档：[RFC6455](https://www.rfc-editor.org/rfc/rfc6455)

Github上的中文翻译文档：[WebSocket协议RFC文档](https://github.com/HJava/myBlog/tree/master/WebSocket%20%E5%8D%8F%E8%AE%AE%20RFC%20%E6%96%87%E6%A1%A3)

