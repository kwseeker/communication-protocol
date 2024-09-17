# 从 PocketTracer 看网络通信原理



## 附录

### 基本概念

+ PDU：Protocol Data Unit，协议数据单元，指的是在不同网络层次间传递的数据单位，每个层次都会根据自己的协议来定义PDU的格式和用途。

+ LAN：Local Area Network，局域网。

+ VLAN：Virtual Local Area Network，虚拟局域网。

+ ARP：Address Resolution Protocol，地址解析协议，用于将网络层的IP地址映射为链路层的物理地址（即MAC地址）。

+ 交换机主要作用
  + 实现MAC地址和硬件端口（比如 Ethernet0/1）的映射，即根据数据包中的MAC地址信息，决定将数据包转发到哪个端口；
  + 实现 VLAN，分割大的广播域，提升网络带宽利用率，各子VLAN是不同的网段；TODO?
  + 拓展局域网设备数量，比如 2950-24 只有24个Ethernet端口，最多支持24台机器，但是可以通过多层交换机，拓展局域网设备数量。

+ 无线网络中的概念：

  + AP: Access Point，接入点，如智能手机、笔记本电脑等，可通过Wi-Fi连接到网络。

  + BSS：Basic Service Set，基本服务集，无线网络中的一个基本单元，由一个AP和所有与之通信的无线设备组成。

+ PSTN：Public Switched Telephone Network，公共交换电话网络。

+ 三层交换机

  是一种可以在OSI模型第三层即网络层上工作的网络设备。与二层交换机只能通过MAC地址进行数据帧交换不同，三层交换机还能根据IP地址进行数据包的转发，实现更高级别的路由功能。

  三层交换机通过学习网络中不同设备的MAC地址和IP地址，建立一个称为“MAC地址表”和“路由表”的数据结构。当接收到一个数据包时，三层交换机会先检查目标MAC地址是否在MAC地址表中，如果在，则直接将数据包转发到目标设备；如果不在，则会根据目标IP地址查询路由表，找到最佳的路径，并将数据包转发到相应的网络。
  
  > 交换机可以实现同局域网内设备通信，也可以设置虚拟局域网，实现隔离；
  >
  > 不过无法实现不同局域网的通信，需要使用三层交换机或者路由器实现。

### 注意事项

+ PocketTracer 模拟的虚拟设备加入网络后也需要时间启动，网线上的两个三角符号闪绿灯表示两个设备连接成功。
+ PocketTracer 路由器、三层交换机路由模式的硬件端口需要手动开启。

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
```

### 参考资料

+ [动手做计算机网络仿真实验——基于Packet Tracer](https://www.bilibili.com/video/BV1Y14y1r7TE/?spm_id_from=333.999.0.0&vd_source=e085f6b3e74d1e9c35fe18734cac42f7)

  电子版书籍找不到资源可以看这个视频，入门 Packet Tracer。