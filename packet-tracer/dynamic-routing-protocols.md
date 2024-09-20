# 动态路由协议

## RIP

+ RIPv1

  RIPv1 是基于距离向量的，即路由规则是选择**距离最短**（准确说其实是**中间节点数最少**）的路由。

  负载均衡策略是等价负载均衡。

  RIP 协议需要手动配置（命令 `router rip`）直连网络地址 。

  会每隔30秒，向所有连接就绪状态的端口发送（即广播） RIPv1 协议报动态更新路由表，可以通过 `debug ip rip` （开启特权模式后即可执行此命令）查看详细流程。

  ```
  Router>enable
  Router#debug ip rip
  RIP protocol debugging is on
  RIP: sending  v1 update to 255.255.255.255 via GigabitEthernet0/0 (192.168.0.254)
  RIP: build update entries
        network 10.0.0.0 metric 1
        network 20.0.0.0 metric 1
        network 30.0.0.0 metric 2
        network 192.168.1.0 metric 2
  RIP: sending  v1 update to 255.255.255.255 via GigabitEthernet0/2 (20.0.0.1)
  RIP: build update entries
        network 10.0.0.0 metric 1
        network 192.168.0.0 metric 1
  RIP: sending  v1 update to 255.255.255.255 via GigabitEthernet0/1 (10.0.0.1)
  RIP: build update entries
        network 20.0.0.0 metric 1
        network 192.168.0.0 metric 1
        network 192.168.1.0 metric 2
  RIP: received v1 update from 20.0.0.2 on GigabitEthernet0/2
        30.0.0.0 in 1 hops
        192.168.1.0 in 1 hops
  RIP: received v1 update from 10.0.0.2 on GigabitEthernet0/1
        30.0.0.0 in 1 hops
        192.168.1.0 in 2 hops
  ```

  最终的路由表（拓扑图：router-protocols-ripv4.pkt）：

  ![](/home/arvin/mywork/communication-protocol/packet-tracer/imgs/routing-table-for-router1.png)

+ RIPv2

  RIPv1 是有类路由器协议，不支持变长子网掩码 VLSM，RIPv2是无类路由协议，支持VLSM。

  RIPv1 通过广播发送 RIP 更新报文， RIPv2 通过组播发送 RIP 更新报文。RIPv2的组播地址是 224.0.0.9，这个地址被所有运行 RIPv2 的路由器监听，以接收路由更新信息（内部原理暂略），以 router-protocols.pkt 的拓扑场景为例，所有路由器都启用RIPv2后，路由更新数据报就只会在路由器中间传输不会再传给PC端。

  通过在 `router rip` 页面配置 `version 2` 可以启用 RIPv2 协议。

  > 有类路由协议基于IP地址的分类，即A、B、C、D和E类地址；
  >
  > 无类路由协议在路由更新中包含子网掩码，这样它们可以支持任意长度的子网划分（Variable-Length Subnet Mask, VLSM），允许在同一个网络中使用不同大小的子网。
  >
  > 比如 192.168.16.1/25，这里的子网掩码是前面25位，即255.255.255.128。
  >
  > 组播：允许发送者将数据包发送到一组特定的接收者，而不是网络中的所有设备。只有加入组播组的设备才会接收和处理这些数据包。

## OSPF

OSPF 是基于链路状态的，其路由规则是选择”**路径代价最少**“的路由。

> 思科路由器中，OSPF协议计算代价的方法：
>
> 代价 = 参考带宽 / 接口带宽
>
> 默认的参考带宽值是100Mbps。通过使用`auto-cost reference-bandwidth`命令可以修改参考带宽值。
>
> 接口带宽取每个跃点出接口带宽的最小值。

负载均衡策略是等价负载均衡。

OSPF 也是**无类路由协议**，支持变长子网掩码 VLSM。

RIP 协议也需要手动配置（命令 `router ospf [pid]`）直连网络地址 。

```shell
# 先进入特权模式
enable
# 进入配置终端
configure terminal
router ospf 100
# 设置直连网络地址
# area 0：指定该网络应该属于OSPF中的哪个区域。区域ID为0通常被用作OSPF骨干区域（backbone area），所有其他区域都应该通过骨干区域连接。
network 10.0.0.0 0.0.0.3 area 0
network 30.0.0.0 0.0.0.3 area 0
# 清除 OSPF 协议配置
no router ospf 100

# 返回特权模式
# 展示配置
show running-config
show running-config | section ospf #只展示 ospf 的配置
# 查看 ospf 相关事件
debug ip ospf events
# 关闭 ospf 相关事件
no debug ip ospf events
# 展示路由表信息，如 router-protocols-ospf.pkt
show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is not set

     10.0.0.0/30 is subnetted, 1 subnets
O       10.0.0.0/30 [110/67] via 50.0.0.1, 00:05:17, GigabitEthernet0/0
     20.0.0.0/30 is subnetted, 1 subnets
O       20.0.0.0/30 [110/66] via 50.0.0.1, 00:05:17, GigabitEthernet0/0
     30.0.0.0/30 is subnetted, 1 subnets
O       30.0.0.0/30 [110/66] via 50.0.0.1, 00:05:17, GigabitEthernet0/0
     40.0.0.0/30 is subnetted, 1 subnets
O       40.0.0.0/30 [110/65] via 50.0.0.1, 00:05:17, GigabitEthernet0/0
     50.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       50.0.0.0/30 is directly connected, GigabitEthernet0/0
L       50.0.0.2/32 is directly connected, GigabitEthernet0/0
     192.168.16.0/24 is variably subnetted, 2 subnets, 2 masks
O       192.168.16.0/25 [110/67] via 50.0.0.1, 00:05:17, GigabitEthernet0/0
O       192.168.16.128/26 [110/66] via 50.0.0.1, 00:05:17, GigabitEthernet0/0
O    192.168.18.0/24 [110/2] via 50.0.0.1, 00:05:17, GigabitEthernet0/0
     192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.20.0/24 is directly connected, GigabitEthernet0/1
L       192.168.20.254/32 is directly connected, GigabitEthernet0/1
```

## BGP

Border Gateway Protocol，边界网关协议。

BGP vs OSPF:

- OSPF是一种内部网关协议（IGP），主要用于单个自治系统（AS）内部的路由。
- BGP是一种外部网关协议（EGP），设计用于在不同的自治系统之间交换路由信息。
- OSPF使用Dijkstra算法的最短路径优先（SPF）算法来计算到达目的地的最佳路径。
- BGP使用基于路径属性的路径矢量算法来选择最佳路径，这些属性包括路径长度、路由源的可靠性、策略决策等。
- OSPF定期发送路由更新，即使网络没有变化也会这样做。
- BGP仅在网络拓扑发生变化时发送更新，这使得BGP在稳定状态下更高效。
- OSPF仅携带到达目的地的路由信息。
- BGP携带完整的路由信息，包括到达目的地的完整路径（即AS路径）。

配置方法：

```shell
# 100 是当前自治系统的编号
router bgp 100
# 指定邻居路由器，需要指定IP和自治系统的编号
neighbor 10.0.0.2 remote-as 200
network 10.0.0.0 mask 255.255.255.0
end
```

+ OSPF 到 BGP 的再分发

  参考官方支持文档：[Understand the Redistribution of OSPF Routes into BGP](https://www.cisco.com/c/en/us/support/docs/ip/border-gateway-protocol-bgp/5242-bgp-ospf-redis.html)

  测试文件： router-protocols-bgp.pkt。

  核心配置：

  ```shell
  # R2
  router ospf 1
   log-adjacency-changes
   # 用于 OSPF 到 BGP的再分发
   redistribute bgp 200 subnets 
   network 172.10.0.0 0.0.0.255 area 0
  router bgp 200
   bgp log-neighbor-changes
   # 用于 BGP 到 OSPF 的再分发
   bgp redistribute-internal
   no synchronization
   neighbor 10.0.0.2 remote-as 100
   neighbor 20.0.0.2 remote-as 300
   redistribute ospf 1 
  # R3
  router ospf 2
   log-adjacency-changes
   redistribute bgp 300 subnets 
   network 172.10.1.0 0.0.0.255 area 0
  router bgp 300
   bgp log-neighbor-changes
   bgp redistribute-internal
   no synchronization
   neighbor 20.0.0.1 remote-as 200
   neighbor 30.0.0.2 remote-as 100
   redistribute ospf 2 
  ```

+ IGP EGP

  IGP（Interior Gateway Protocol，内部网关协议），适用于单一网络内部，如企业内部网络或ISP的内部网络。

  EGP（Exterior Gateway Protocol，外部网关协议），适用于跨网络的互联，如互联网服务提供商（ISP）之间的互联。