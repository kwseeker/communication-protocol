# 高可靠网络

+ 第一跳冗余协议（FHRP）
  + 热备份路由器协议 HSRP 
  + 虚拟路由器冗余协议 VRRP
  + 网关负载均衡协议 GLBP
  
+ 负载均衡

  + MSTP 

    MSTP提供了一定的负载均衡能力。


## 第一跳冗余协议（FHRP）

### 热备路由协议 HSRP

Hot Standby Router Protocol，热备份路由协议。

它通过将多个路由器组成一个冗余组，提供默认网关的冗余功能。在HSRP中，有一个活动路由器和一个或多个备用路由器。活动路由器负责转发流量，而备用路由器则处于待命状态。

HSRP使用虚拟IP地址（主路由器和备份路由器共同虚拟成一个路由器，对外提供一个虚拟IP地址）和虚拟MAC地址作为冗余组的标识。活动路由器被选举为主路由器，并负责将所有流量转发到虚拟IP地址。备用路由器监听主路由器的状态，并在主路由器故障时接管流量转发。

配置方法：

```shell
# 假设有一主一备两台路由器，通过HSRP协议对外提供虚拟局域网vlan1, 虚拟网关IP 192.168.0.254
# 路由器1
interface vlan1 # 其实没有必要设置虚拟局域网
  # vlan1 接口地址, 虚拟IP在此路由器上实际的IP地址
  ip address 192.168.1.252 255.255.255.0
  # standby 10 表示HSRP组号为10，ip 192.168.1.254 表示模拟的虚拟网关IP
  standby 10 ip 192.168.1.254
  # 当前服务器的优先级设置为了120,默认是100，值越大优先级越高
  standby 10 priority 120
  #standby 10 preempt
  
# 查看 standby 状态
show standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Gig0/0/0    10   120   Active   local           unknown         192.168.1.254 
  
# 路由器2
interface vlan1
  ip address 192.168.1.253 255.255.255.0
  standby 10 ip 192.168.1.254
  #standby 10 priority 100
  # 启用了优先级抢占功能
  standby 10 preempt
  
# 在路由器1中再次查看 standby 状态，因为路由器1没有开启优先级抢占，并不会被抢占
show standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Gig0/0/0    10   120   Active   local           192.168.1.253   192.168.1.254 
  standby 10 preempt
# 在路由器2中查看 standby 状态
show standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Gig0/0/0    10   100 P Standby  192.168.1.252   local           192.168.1.254
```

详细配置流程参考：[cisco packet tracer配置HSRP](https://www.bilibili.com/video/BV1w8411a7mL/?spm_id_from=333.337.search-card.all.click&vd_source=e085f6b3e74d1e9c35fe18734cac42f7) 与 `high-reliability-net.pkt`。

`high-reliability-net.pkt`展示了`HSRP+静态路由`和 `HSRP+OSPF`两种配置实现。

### 虚拟路由器冗余协议 VRRP

VRRP是一种开放标准的第一跳冗余协议，工作原理和HSRP基本相同，用于提供冗余的默认网关。

配置方法也基本相同。

不再复述。

### 各方案对比

- HSRP和VRRP在主路由器故障时提供备份，而GLBP提供负载均衡和冗余。
- HSRP和VRRP的备用路由器通常处于闲置状态，而GLBP可以充分利用所有路由器的资源。
- GLBP配置较为复杂，且仅在Cisco设备上可用，而HSRP和VRRP更通用。

## 负载均衡

TODO

## 生成树

生成树（Spanning Tree）实现交换机之间冗余连接的同时避免网络环路的出现，实现网络的高可用性。

