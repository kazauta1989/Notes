# TCP/UDP

## TCP 协议

：传输控制协议（Transmission Control Protocol）
- 规定了网络上的主机之间如何传输数据，属于传输层协议。
- 采用 C/S 架构，通信双方分别称为 server、client 。
  - 面向连接：通信双方在通信之前要先建立信道。
  - 是可靠的通信协议，可保证消息顺序，可进行差错控制、流量控制。
  - 可进行全双工通信。
- 大部分应用层协议都是基于 TCP 进行通信的，比如 HTTP、FTP、SMTP 等。

### 数据包结构

TCP 数据包包含以下信息：
- 源端口 ：16 bit
- 目标端口 ：16 bit
- seq ：序列号，32 bit ，用于保证消息顺序。
- ack ：确认号，32 bit ，表示期望收到的下一个序列号。
- ...
- flag ：标志符，6 bit ，每个 bit 代表一个标志位，默认为 0 。
  - URG=1 ：表示该数据是紧急数据，应该被优先处理。
  - ACK=1 ：表示确认。
  - PSH=1 ：表示发送方应该立即将该数据打包成一个 TCP 数据包发送，接收方也应该立即将该数据上报给上层程序。
    - TCP 模块在发送数据时，一般会等发送缓冲区满了，才打包成一个 TCP 数据包发送。同理，接收数据时也一般会等接收缓冲区满了才上报。
    - 一般一个应用层的报文会被切分成多个 TCP 数据包发送，最后一个 TCP 数据包会设置成 PSH=1 。
  - RST=1 ：用于重新建立 TCP 连接，也可用于拒绝连接。
  - SYN=1 ：用于建立 TCP 连接，开始同步。
  - FIN=1 ：用于断开 TCP 连接。
- checksum ：校验和，16 bit
- ...
- 数据

### 建立连接

建立 TCP 连接时需要经过三个步骤，称为“三次握手”：
1. client 发送一个 SYN=1 的 TCP 包，表示请求连接。
2. server 收到后，回复一个 ACK=1、SYN=1 的 TCP 包，表示允许连接。
3. client 收到后，发送一个 ACK=1 的 TCP 包，表示正式建立连接。

- 总之，通信双方都要发送一个 SYN、一个 ACK 。
- 主动建立 TCP 连接的一方是 client 。
- 建立连接之后，双方便可以进行通信。

### 断开连接

断开 TCP 连接时需要经过四个步骤，称为“四次分手”：
1. 主机 A 发送 FIN ，表示请求断开连接。
2. 主机 B 收到后，先回复 ACK ，表示同意断开连接。
   <br>准备好了之后也发送 FIN ，请求断开连接。
3. 主机 A 收到后，发送一个 ACK ，表示自己已经断开连接。
4. 主机 B 收到后，正式断开连接。

- 总之，通信双方都要发送一个 FIN、一个 ACK 。
- client、server 都可以主动断开连接，因此这里用主机 A、主机 B 代称。

## UDP 协议

：用户数据报协议（User Datagram Protocol）
- 与 TCP 协议类似，只是面向无连接。
- 通信双方分别称为 sender、receiver 。
- TCP 需要一直运行一个服务器，而 UDP 适合用作即时通信、广播消息。
- 少部分应用层协议是基于 UDP 进行通信的，比如 DHCP、DNS、SNMP、RIP 等。
