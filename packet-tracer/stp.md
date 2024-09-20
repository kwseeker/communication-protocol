# 生成树协议

生成树协议（Spanning Tree Protocol）用于在网络中的交换机之间消除环路并提供路径冗余。

常用生成树算法：

+ STP

  STP 是最早出现的生成树协议，它通过计算生成树来消除网络中的环路。STP 会选择一个根桥，然后根据根桥的位置计算出各个端口的角色（根端口、指定端口、阻塞端口），从而实现无环拓扑。

+ RSTP

  RSTP（Rapid Spanning Tree Protocol）是 STP 的改进版本，它在保持 STP 基本原理的基础上，提高了收敛速度和对拓扑变化的响应能力。

+ MSTP

  MSTP 是基于 STP 和 RSTP 发展而来的多生成树协议，它允许在一个交换环境中运行多个生成树，每个生成树称为一个实例（instance）。这些实例彼此独立，可以为不同的VLAN提供不同的路径，从而实现负载均衡和优化网络资源的使用。MSTP通过设置VLAN映射表（即VLAN与MSTI的对应关系表），把VLAN和MSTP联系起来。每个VLAN只能对应一个MSTI，而一个MSTI可以对应多个VLAN。
