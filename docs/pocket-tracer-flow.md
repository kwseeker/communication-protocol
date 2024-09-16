# 从 PocketTracer 看网络通信原理



## 附录

### 注意事项

+ PocketTracer 模拟的虚拟设备加入网络后也需要时间启动，网线上的两个三角符号闪绿灯表示两个设备连接成功。

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