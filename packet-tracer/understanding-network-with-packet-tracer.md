# 从 PocketTracer 看网络通信原理

组网常用技术：

+ DHCP
+ 动态IP

+ 动态路由

  + OSPF

+ 高可靠

  + HSRP
  + VRRP

+ 生成树 STP

+ VPN

+ NAT

+ Bridge

+ 网络安全

  + ACL
  + 防火墙



## 附录

### 基本概念

+ PDU

  Protocol Data Unit，协议数据单元，指的是在不同网络层次间传递的数据单位，每个层次都会根据自己的协议来定义PDU的格式和用途。

+ LAN

  Local Area Network，局域网。

+ VLAN：

  Virtual Local Area Network，虚拟局域网。

+ 集线器

  Hub,  工作在物理层的设备，集线器接收到一个网络设备发送的信号时，它会重新复制这个信号，然后广播到所有其他端口。这意味着集线器并不识别或处理数据帧中的地址信息，它简单地将所有接收到的数据帧转发到除了源端口之外的所有其他端口。

  集线器是半双工通信设备，即可以双向传输数据但是同一时间只能发送或接收数据。

+ 交换机

  Switch，工作在数据链路层（第二层）。

  + 实现MAC地址和硬件端口（比如 Ethernet0/1）的映射，即根据数据包中的MAC地址信息，决定将数据包转发到哪个端口；
  + 实现 VLAN，分割大的广播域，提升网络带宽利用率，各子VLAN是不同的网段；TODO?
  + 拓展局域网设备数量，比如 2950-24 只有24个Ethernet端口，最多支持24台机器，但是可以通过多层交换机，拓展局域网设备数量。

+ 网桥

  Bridge，工作在数据链路层（第二层），主要用于连接两个或多个局域网（LAN）段。

+ 中继器

  Repeater，工作在物理层（第一层），其主要功能是扩展网络的距离和增强信号，补偿信号在传输介质（如双绞线或光纤）中因距离增加而产生的信号衰减。

+ 碰撞域与广播域

  碰撞域，Collision Domain（又名冲突域），指两个或多个网络设备同时尝试在同一通信媒介上发送数据时可能发生冲突的区域。在计算机和计算机通过设备互联时，会建立一条通道，如果这条通道只允许瞬间一个数据报文通过，那么在同时如果有两个或更多的数据报文想从这里通过时就会出现冲突了。

  广播域，Broadcast Domain，指一个数据帧或包被广播时能够到达的所有设备组成的区域。

  集线器既不隔离冲突域（半双工通信，两台设备通过集线器同时发送数据可能发生冲突），也不隔离广播域；
  交换机隔离冲突域（全双工通信），不隔离广播域；
  路由器既隔离冲突域，也隔离广播域。

+ 无线网络中的概念：

  + AP: Access Point，接入点，如智能手机、笔记本电脑等，可通过Wi-Fi连接到网络。

  + BSS：Basic Service Set，基本服务集，无线网络中的一个基本单元，由一个AP和所有与之通信的无线设备组成。

+ PSTN：Public Switched Telephone Network，公共交换电话网络。

+ ARP

  Address Resolution Protocol，地址解析协议，用于将网络层的IP地址映射为链路层的物理地址（即MAC地址）。

+ ICMP

+ 子网掩码

  用于区分IP地址中的网络部分和主机部分。比如 
  192.168.16.1/25，网络号是 192.168.16.0，主机号是 1；

  192.168.16.130/25，网络号是 192.168.16.128，主机号是 2。

+ 三层交换机

  是一种可以在OSI模型第三层即网络层上工作的网络设备。与二层交换机只能通过MAC地址进行数据帧交换不同，三层交换机还能根据IP地址进行数据包的转发，实现更高级别的路由功能。

  三层交换机通过学习网络中不同设备的MAC地址和IP地址，建立一个称为“MAC地址表”和“路由表”的数据结构。当接收到一个数据包时，三层交换机会先检查目标MAC地址是否在MAC地址表中，如果在，则直接将数据包转发到目标设备；如果不在，则会根据目标IP地址查询路由表，找到最佳的路径，并将数据包转发到相应的网络。

  三层交换机的端口有两种工作模式：交换、路由，当使用某个端口连接外部网络时需要将端口切换为路由模式， 使用`no switchport`。

  > 交换机可以实现同局域网内设备通信，也可以设置虚拟局域网，实现隔离；
  >
  > 不过无法实现不同局域网的通信，需要使用三层交换机或者路由器实现。

+ 动态路由协议

  参考：[动态路由协议](dynamic-routing-protocols.md)

+ 高可靠网络

  参考：[高可靠网络](high-reliable-network.md)

### 注意事项

+ PocketTracer 模拟的虚拟设备加入网络后也需要时间启动，网线上的两个三角符号闪绿灯表示两个设备连接成功。
+ PocketTracer 路由器、三层交换机路由模式的硬件端口需要手动开启。
+ PocketTracer 路由表中 Type 值中，C 表示 “Connected”即连接的网络, L 表示 Local 即本地路由，指本机 IP 地址。

### Pocket Tracer 常用命令

```shell
# 切换到管理员权限
enable
# 开启配置终端
configure terminal
# 开启端口配置
interface GigabitEthernet0/1
# 用于将交换机的接口从数据链路层模式（即交换模式）更改为网络层模式（即路由模式）。这通常在配置三层交换机时使用，允许交换机在该接口上进行路由功能。
no switchport
# 配置端口IP、子网掩码
ip address 10.0.11.1 255.255.255.0
# 启用设备的路由功能
ip routing
# 开启配置终端 end
# 展示路由表内容,其中除了 C、L、还有R类型（表示通过RIP协议到达某个非直连网络的路由条目）
show ip route
# 查看 RIP 协议动态更新路由表过程。 
debug ip rip
no debug ip rip
# 关闭 RIP 协议，并删除RIP路由信息
no router rip
# router rip 的子命令
# 选用 RIPv2 协议
version 2
# 关闭路由信息自动汇整，可以查看内部路由信息更新详细流程
no auto-summary
# router rip 的子命令 end
```

### 参考资料

+ [Packet Tracer 官方教程](https://tutorials.ptnetacad.net/)

+ [Packet Tracer 中文手册 ](https://cisco-packet-tracer-help.yue.zone/Simplified%20Chinese/index.htm)

  不过并不完整，官方手册没找到。

+ [动手做计算机网络仿真实验——基于Packet Tracer](https://www.bilibili.com/video/BV1Y14y1r7TE/?spm_id_from=333.999.0.0&vd_source=e085f6b3e74d1e9c35fe18734cac42f7)

  电子版书籍找不到资源可以看这个视频，入门 Packet Tracer。

+ [网络工程师从基础入门到进阶必学教程](https://www.bilibili.com/video/BV1PV4y1y7e4?p=18&spm_id_from=pageDriver&vd_source=e085f6b3e74d1e9c35fe18734cac42f7)