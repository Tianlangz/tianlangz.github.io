---
author: zty
title: VXLAN
date: 2024-04-18
description:  网络安全基础和IE考试
series: 
  - 网络安全基础和IE考试
tags : [
    网络基础,HCIE考试,虚拟园区网
]
categories : [
    ENSP,
    华为设备配置
]
series : [HCIE考试]
aliases : [数通基础]
---

<!--more-->
# 协议概述
## 概念
- VXLAN：虚拟扩展局域网（Virtual eXtensible Local Area Network）
- NVE：在那个设备上做的报文封装，也就是实现虚拟化功能的设备
- VTEP：Vxlan隧道的端点，封装在vxlan报文中
- VNI：VXLAN网络标识，封装在VXLAN头部，类似于vlan id，用于区分不同的vxlan段
- BD：
  - 桥接域，类似于VLAN划分广播域，但是vlan数量有限，无法满足大二层的网络，因此用BD标识一个大二层域，**`VNI与BD1:1映射`**
  - BD概念解析：相同网段互访时是通过VXLAN隧道跨三层实现相同或不同vlan的通讯，通过VXLAN隧道将隧道两端断掉的BD连接起来，因为BD关联了VXLAN就相当于将断掉的VLAN桥接起来，从而实现大二层网络的互通
  - 特点：隧道两端的BD可以不同，隧道两端的二层VNI必须相同，VLAN与BD与VNI是1:1:1的对应关系，可以用多个vlan对应一个BD
- VAP：虚拟接入点，实现VXLAN业务的接入
## 场景
- 早期应用在数据中心的技术，数据中心网络使用的技术有哪些：
  - 堆叠+链路聚合
  - M-LAG跨设备链路聚合
  - VXLAN
  - VXLAN EVPN
- 服务器虚拟化
  - 传统企业服务器集群虚拟化部署
    ![](/images/VXLAN传统企业服务器虚拟化.png)
  - 数据中心服务器虚拟化部署
    ![](/images/VXLAN数据中心服务器虚拟化.png)
  - VXLAN技术从数据中心延伸至园区网，实现一网多用
    ![](/images/VXLAN园区网服务器虚拟化.png)
## 报文结构
![](/images/VXLAN报文结构.png)
解析：外层以太网头
![](/images/VXLAN外层以太网头.png)
这一部分应该小于1500字节，原始报文
![](/images/VXLAN原始报文.png)
这一部分可能都达到1500字节了，再加上
![](/images/VXLAN外层以太网头剩下报文.png)
这一段，50字节，一共1550字节，超过了最大MTU值就要分片重组，带来设备的开销较大，因此需要开启接口最大MTU值超过1550，避免分片重组，最大接口MTU可设置为9600字节

### {数据|IP|MAC}|VXLAN|UDP|VTEP|外层MAC
- {数据|IP|MAC}
  - 原始数据
- VXLAN
  - 封装VNI
- UDP
  - 源端口
    - 生成方式：使用原始数据帧的以太网头进行哈希计算
    - 作用：使用原始数据帧中的头部进行哈希计算，通过哈希值除以负载分担条数余数是几就选第几条转发的路径，这样才能让流量选择不同的路径，更好的实现负载分担
  - 目标端口
    - 4789
- VTEP
  - 数据转发封装的外层IP
- 外层mac
## 隧道模式
- 静态VXLAN隧道，不引入控制平面
- 动态VXLAN隧道，引入控制平面
## 网关模式
### 集中式网关
可以通过建立静态隧道，也可以建动态隧道
- 场景：多用于园区网
- 不足
  1. 如果1.1需要访问2.1，CE2作为集中式网关，会存在次优路径的问题
  2. 网关arp表项负担过重
  ![](/images/VXLAN集中式网关拓扑.png)
### 分布式网关
#### 特点
只支持动态建立隧道
#### 场景
多用于数据中心，主要解决集中式网关场景中，跨网段互访出现的次优路径问题
#### 分类
##### 非对称式：
- 拓扑
  ![](/images/VXLAN分布式网关拓扑.png)
- 解析
  - 同一台CE交换机：

    网管部署在CE1上，PCV1直接访问PC4，不同网段的主机通过直连路由访问，解决了次优路径的问题
  - 不同CE交换机：
    - 去包

      192.168.1.1网段访问192.168.2.2网段，数据包到达CE1后根据目标地址查找路由表，发现2.0网络是自己的直连路由，查找MAC地址表，向CE3发送ARP广播，封装2.0网络的VNI`源IP2.254目的IP2.2|源MAC2.254的网关MAC目的MAC为全F（广播）|二层VNI|UDP|VTEP IP|二层MAC`数据包到达CE3后解封装，根据二层VNI直接桥接到2.0网络所在的BD，在BD下所有子接口泛洪arp广播流量，目标主机接收到该请求后，单播回复arp，CE1进行icmp报文的封装`源IP1.1目的IP2.2|源MAC2.254的网关MAC目的MAC为PC2|二层VNI|UDP|VTEP IP|二层MAC`
    - 回包

      CE3必须要有1.0网络和2.0网络的网关，PC2回包时先找到网关2.0，根据目标地址1.1查找路由表，匹配直连路由，`源IP2.2目的IP1.1|源MAC1.254目的MAC1.1|1.0的VNI|UDP|VTEP IP|二层MAC`因此CE3必须要配置1.0网关，即使该网关下没有主机，回包时封装的是1.0 网段的VNI值
    - 总结：本端CE先路由后桥接，对端CE只桥接
    - 非对称式网关特点
      1. 去包封装的VNI值和回包封装的VNI值不同，第一个不对称
      2. 在数据转发时CE1要先查路由表（L3表），再查MAC（L2表），数据到达CE3只需要查MAC表，第二个不对称
    - 非对称式网关的不足
      1. 占用设备资源大，本端CE查两次表项，对端查一次表项，工作负担不一致
      2. CE1、CE3上必须要有所有网段网关，即使该分布式网关节点下没有终端接入
##### 对称式
- 对称式网关特点
  1. 去包封装的三层VNI值和回包封装的VNI值相同
  2. 本端CE查路由表，对端CE也是查路由表，路由器行为相同
- 对比
  ![](/images/VXLAN对称式对比.png)
# 工作原理
## 不引入控制平面-同网段互访
### 拓扑
![](/images/VXLAN不引入控制平面同网段互访.png)
### 需求
Client1的IP地址为192.168.1.1，可以访问Client2的IP地址192.168.1.2，通过vxlan实现跨三层网络互通
### 步骤
1. 配置交换机S1、S2分别打上vlan100和vlan200，上行口为trunk，放行所有vlan
2. CE1、CE2、CE3部署OSPF，打通环回口地址
3. 创建BD，BD下配置VNI
4. 在NVE设备上配置二层网关
5. 配置子接口，子接口终结vlan，绑定BD
### 命令
```
CE2配置：
#
interface GE1/0/2.1 mode l2      
 encapsulation dot1q vid 10    //隧道两端可以不一致
 bridge-domain 10
#
bridge-domain 10               // 隧道两端可以不一致
 vxlan vni 100                  /隧道两端必须一致 
#
interface Nve1                // 创建二层网关，进行隧道封装
 source 10.10.2.2
 vni 100 head-end peer-list 10.10.3.3
            // 如果发现封装了vni 100的流量则向头端复制列表中的地址泛洪流量
#
--------------------------------------------------------------------------
CE3配置：
#
interface GE1/0/1.1 mode l2
 encapsulation dot1q vid 10   /隧道两端可以不一致
 bridge-domain 10
#
bridge-domain 10           /隧道两端可以不一致
 vxlan vni 100    /隧道两端必须一致 
#
interface Nve1
 source 10.10.3.3
 vni 100 head-end peer-list 10.10.2.2
--------------------------------------------------------------------------
 查看隧道信息：
 <ce2>display  vxlan tunnel 
Number of vxlan tunnel : 1
Tunnel ID   Source         Destination         State  Type     Uptime
--------------------------------------------------------------------------
4026531841  10.10.2.2        10.10.3.3          up     static   01:47:54  
 查看MAC地址：
[~ce2]display  mac-address 
Flags: * - Backup  
       # - forwarding logical interface, operations cannot be performed based 
           on the interface.
BD   : bridge-domain   Age : dynamic MAC learned time in seconds
---------------------------------------------------------------------------
MAC Address    VLAN/VSI/BD   Learned-From        Type                Age
----------------------------------------------------------------------------
fa34-367b-0051 1/-/-         GE1/0/2             dynamic                701
24bd-1620-50bb -/-/10        10.10.3.3           dynamic                574
c476-44e2-6246 -/-/10        GE1/0/2.1           dynamic                629
-----------------------------------------------------------------------------
```
### 转发原理解析
- 去程：PC1访问PC2发送arp`广播流量`，到达S1，打上标签vlan100，从CE2子接口接收后，终结掉vlan100，通过关联BD，BD又关联VNI，NVE网关发现VNI是100，`根据头端复制列表进行向具有相同的VNI100的VTEP目的地址泛洪流量`，封装外层VTEPIP，和外层MAC，CE3学习MAC地址，解封装露出VNI，通过VNI对应到BD，向BD下所有子接口泛洪流量，广播帧带上标签转发给S2，经过S2后剥离标签，流量到达PC2
![](/images/VXLAN不引入控制平面同网段互访转发原理解析去程.png)
- 回包：`因为回包是已知单播帧，已经形成了MAC地址表。根据目标地址查找MAC地址表转发`，根据MAC地址表中的信息可知，主机MAC属于BD10，BD10关联了VNI100，进行封装VNI100，该MAC来自10.10.2.2，在通过MAC地址表中的learned-from也就是VTEP的目的IP10.10.2.2进行VXLAN隧道的封装
![](/images/VXLAN不引入控制平面同网段互访转发原理解析回包1.png)
![](/images/VXLAN不引入控制平面同网段互访转发原理解析回包2.png)
到达隧道的终点解封装剥离VXLAN头部，漏出原始报文目标MAC地址，通过查找MAC地址表，转发至PC1
```
[~ce2]display  mac-address 
Flags: * - Backup  
       # - forwarding logical interface, operations cannot be performed based 
           on the interface.
BD   : bridge-domain   Age : dynamic MAC learned time in seconds
---------------------------------------------------------------------------
MAC Address    VLAN/VSI/BD   Learned-From        Type                Age
----------------------------------------------------------------------------
fa34-367b-0051 1/-/-         GE1/0/2             dynamic                701
24bd-1620-50bb -/-/10        10.10.3.3           dynamic                574
c476-44e2-6246 -/-/10        GE1/0/2.1           dynamic                629       //PC1的主机mac地址
-----------------------------------------------------------------------------

```
## 不引入控制平面-集中式网关
### 拓扑
![](/images/VXLAN不引入控制平面集中式网关拓扑.png)
### xvqiu 
1. client1可以访问同网段的client3
2. client1可以访问不同网段的client2
3. client3可以访问不同网段的client2
### 步骤
1. 配置交换机S1、S2分别打上vlan10和vlan20，上行口为trunk，放行所有vlan
2. CE1、CE2、CE3部署OSPF，打通环回口地址
3. 创建BD，BD下配置VNI
4. 在CE1、CE3设备上配置二层网关
5. 配置子接口，子接口终结vlan，绑定BD
6. 在CE2配置二层、三层网关
### 命令
```
预配：
==========================================================================
CE1配置:
sys
#
sysname ce1
#
user-interface con 0
 idle-timeout 1400
#
interface LoopBack0
 ip address 10.10.1.1 255.255.255.255
#
interface GE1/0/1
 undo portswitch
 ip address 10.1.12.1 255.255.255.0
#
ospf 1 router-id 1.1.1.1
 area 0.0.0.0
  network 10.1.12.1 0.0.0.0
  network 10.10.1.1 0.0.0.0
CE2配置：
sys
#
sysname ce2
#
user-interface con 0
 idle-timeout 1400
#
interface LoopBack0
 ip address 10.10.2.2 255.255.255.255  
#
interface GE1/0/1
 undo portswitch
 ip address 10.1.23.2 255.255.255.0
#
interface GE1/0/2
 undo portswitch
 ip address 10.1.12.2 255.255.255.0
#
ospf 1 router-id 2.2.2.2
 area 0.0.0.0
  network 10.1.12.2 0.0.0.0
  network 10.1.23.2 0.0.0.0
  network 10.10.2.2 0.0.0.0 
CE3配置：
sys
#
sysname ce3
# 
user-interface con 0
 idle-timeout 1400
#
interface LoopBack0
 ip address 10.10.3.3 255.255.255.255 
#
interface GE1/0/2
 undo portswitch
 ip address 10.1.23.3 255.255.255.0 
#
ospf 1 router-id 3.3.3.3
 area 0.0.0.0
  network 10.1.23.3 0.0.0.0
  network 10.10.3.3 0.0.0.0
 ===========================================================================
 CE1配置：
bridge-domain 10
 vxlan vni 100
#
interface Nve1
 source 10.10.1.1
 vni 100 head-end peer-list 10.10.2.2
 vni 100 head-end peer-list 10.10.3.3
#
interface GE1/0/2.1 mode l2
 encapsulation dot1q vid 10
 bridge-domain 10
#
--------------------------------------------------------------------------
CE3配置：
#
bridge-domain 10
 vxlan vni 100
#
bridge-domain 20
 vxlan vni 200
#
interface Nve1
 source 10.10.3.3
 vni 100 head-end peer-list 10.10.1.1
 vni 200 head-end peer-list 10.10.2.2
#
interface GE1/0/1.1 mode l2
 encapsulation dot1q vid 10
 bridge-domain 10
#
interface GE1/0/1.2 mode l2
 encapsulation dot1q vid 20
 bridge-domain 20
 -------------------------------------------------------------------------
 CE2配置：
 #
bridge-domain 10
 vxlan vni 100
#
bridge-domain 20
 vxlan vni 200
#
 interface Nve1
 source 10.10.2.2
 vni 100 head-end peer-list 10.10.1.1
 vni 200 head-end peer-list 10.10.3.3
#
interface Vbdif10
 ip address 192.168.1.254 255.255.255.0
#
interface Vbdif20
 ip address 192.168.2.254 255.255.255.0
 --------------------------------------------------------------------------
 查看隧道信息：
[ce2] display  vxlan tunnel 
Number of vxlan tunnel : 
Tunnel ID   Source                   Destination           State  Type     Uptime
----------------------------------------------------------------------------------------------------
4026531841  10.10.2.2             10.10.1.1               up     static   00:48:08  
4026531842  10.10.2.2             10.10.3.3               up     static   00:47:38   
 查看MAC地址：
<ce1>display  mac-address 
Flags: * - Backup  
       # - forwarding logical interface, operations cannot be performed based 
           on the interface.
BD   : bridge-domain   Age : dynamic MAC learned time in seconds
-------------------------------------------------------------------------------
MAC Address    VLAN/VSI/BD   Learned-From        Type                Age
-------------------------------------------------------------------------------
fa20-88d3-0051 1/-/-         GE1/0/2             dynamic               4155
ce00-56c8-2d4a -/-/10        GE1/0/2.1           dynamic               1634
-----------------------------------------------------------------------------
[ce2]display  mac-address 
Flags: * - Backup  
       # - forwarding logical interface, operations cannot be performed based 
           on the interface.
BD   : bridge-domain   Age : dynamic MAC learned time in seconds
-------------------------------------------------------------------------------
MAC Address    VLAN/VSI/BD   Learned-From        Type                Age
-------------------------------------------------------------------------------
ce00-56c8-2d4a -/-/10        10.10.1.1           dynamic               2877
2c70-df40-609a -/-/20        10.10.3.3           dynamic               1654
------------------------------------------------------------------------------
[ce3]display  mac-address 
Flags: * - Backup  
       # - forwarding logical interface, operations cannot be performed based 
           on the interface.
BD   : bridge-domain   Age : dynamic MAC learned time in seconds
-------------------------------------------------------------------------------
MAC Address    VLAN/VSI/BD   Learned-From        Type                Age
-------------------------------------------------------------------------------
fa20-88d3-0011 1/-/-         GE1/0/1             dynamic               5531
2c70-df40-609a -/-/20        GE1/0/1.2           dynamic               2201
fa20-88d3-0041 -/-/20        10.10.2.2           dynamic               2201
--------------
```
### 转发原理解析
1. 主机获取网关MAC

   client1访问client2，不同网段访问发送arp`广播`，获取网关MAC地址，到达S1，打上标签vlan10，从CE1子接口接收后，终结掉vlan10，通过关联BD，BD又关联VNI，NVE网关发现VNI是100，`根据头端复制列表泛洪`广播流量，冰灯装外层VTEPIP，和外层MAC，根据隧道的目的地址进行转发到CE2，CE2解封装漏出VNI，通过VNI对应到BD，找到vbdif10的网关，网关回复单播MAC地址给Client1，通过MAC地址表找到所属的BD，BD关联了VNI，封装目的VTEP10.10.1.1
2. 主机封装icmp报文

   主机封装网关MAC地址作为目的MAC地址，发送icmp请求，目标地址192.168.2.1，CE2收到后，剥离vxlan标签，解封装到网络层。发现目标IP为192.168.2.1，在vbdif20的直连路由下，因此封装BD20对应的VNI200。发送ARP广播报文获取client2的mac地址，根据头端复制列表泛洪，client2收到后，根据mac地址表转发，封装隧道目的地址，单播回复arp请求

   此时CE2重新封装数据包，源MAC为vbdif20的MAC，目的MAC为client2的MAC，根据MAC地址表找到BD，BD对应VNI以及隧道的目的地址进行封装和转发，最终转发到client3，client3单播回复icmp请求
## EVPN工作原理
### 场景
结合VXLAN使用，作为VXLAN的控制平面，通过EVPN传递主机路由和VTEP信息
### 概念
EVPN是一种用于二层网络互联的VPN技术
### 作用
用于解决VXLAN存在的问题，VXLAN本身没有控制平面，如果没有EVPN，VXLAN通过泛洪ARP报文的方式获取目标主机MAC，并进行VTEP发现和主机信息（包括IP地址、MAC地址、VNI、网关VTEPIP地址）的获取，导致广播流量较大。有了EVPN后可以传递EVPN路由，携带VTEP地址自动建立隧道和传递主机信息，减少了不必要的广播流量。
### 原理：
EVPN路由有三种路由类型
1. type2：用于通告MAC/IP型路由，Type2根据报文携带的内容，又分为：
  - MAC类型路由携带（二元组）：主机MAC地址+二层VNI；`相同网段通告`（通过MAC路由也可以动态建立隧道）
  - ARP类型路由携带（三元组）：主机MAC地址+主机IP地址+二层VNI；用于在集中式网关中生成ARP调优表，通过将广播变为单播，进行ARP广播报文的抑制，减少广播流量的泛洪
  - IRB类型路由携带（四元组）：主机MAC地址+主机IP地址+二层VNI+三层VNI。`不同网段通告`
2. type3：VTEP地址的自动发现和`动态建立隧道`
  - 携带二层VNI+本地VTEP IP地址信息，用于建立头端复制列表，通过头端列表泛洪BUM帧
  - 通过MP-REACH-NLRI字段中的tunnel标识地址动态建立隧道，通过VNI+VTEP生成组播泛洪表
3. type5：`外部路由的通告`
  - 携带路由信息+三层VNI
## 控制平面引入EVPN-同网段互访
### 拓扑
![](/images/VXLAN控制平面引入EVPN同网段互访.png)
### 需求
1. PC1的IP地址为192.168.1.1，，可以访问PC3的IP地址192.168.1.2，通过vxlan实现跨三层网络实现互通，evpn作为控制平面
2. PC4访问PC5，192.168.2.1访问192.168.2.2，通过vxlan实现跨三层网络实现互通，evpn作为控制平面
### 步骤
1. 配置交换机S1、S2分别打上vlan100和vlan200，上行口为trunk，放行所有vlan
2. CE1、CE2、CE3部署OSPF，打通环回口地址
3. 创建BD，BD下配置VNI，BD下创建VNI，BD下创建evpn实例，配置RD、RT
4. 在NVE设备上配置二层网关
5. 配置子接口，子接口终结vlan，绑定BD
### 命令
```
CE1配置：
evpn-overlay enable
#
bridge-domain 10
 vxlan vni 100
 evpn
  route-distinguisher 100:1
  vpn-target 2:2 export-extcommunity
  vpn-target 2:2 import-extcommunity
#
bridge-domain 20
 vxlan vni 200
 evpn
  route-distinguisher 300:1
  vpn-target 3:3 export-extcommunity /通过RT值决定，哪个BD下接收邻居通告的MAC地址
  vpn-target 3:3 import-extcommunity
#
bgp 100
 peer 10.10.3.3 as-number 100
 peer 10.10.3.3 connect-interface LoopBack0
#
 ipv4-family unicast
  peer 10.10.3.3 enable
#
 l2vpn-family evpn
  policy vpn-target
  peer 10.10.3.3 enable
#
interface Nve1
 source 10.10.1.1
 vni 100 head-end peer-list protocol bgp  
 vni 200 head-end peer-list protocol bgp
#
interface GE1/0/2.1 mode l2
 encapsulation dot1q vid 10
 bridge-domain 10
#
interface GE1/0/2.2 mode l2
 encapsulation dot1q vid 20
 bridge-domain 20
--------------------------------------------------------------------------
CE3配置：
evpn-overlay enable
#
bridge-domain 10
 vxlan vni 100
 evpn
  route-distinguisher 200:1
  vpn-target 2:2 export-extcommunity
  vpn-target 2:2 import-extcommunity
#
bridge-domain 20
 vxlan vni 200
 evpn
  route-distinguisher 400:1
  vpn-target 3:3 export-extcommunity  /通过RT值决定，哪个BD下接收邻居通告的MAC地址
  vpn-target 3:3 import-extcommunity
#
bgp 100
 peer 10.10.1.1 as-number 100
 peer 10.10.1.1 connect-interface LoopBack0
 #
 ipv4-family unicast
  peer 10.10.1.1 enable
 #
 l2vpn-family evpn
  policy vpn-target
  peer 10.10.1.1 enable
#
interface Nve1
 source 10.10.3.3
 vni 100 head-end peer-list protocol bgp
 vni 200 head-end peer-list protocol bgp
#
interface GE1/0/2.1 mode l2
 encapsulation dot1q vid 10
 bridge-domain 10
#
interface GE1/0/2.2 mode l2
 encapsulation dot1q vid 20
 bridge-domain 20
-------------------------------------------------------------------------
 查看隧道信息：
[ce1] display  vxlan tunnel 
Number of vxlan tunnel 
Tunnel ID   Source             Destination          State  Type     Uptime
-----------------------------------------------------------------------------
4026531848  10.10.1.1           10.10.3.3            up     dynamic  01:04:27    
[ce1] display  vxlan vni  100
VNI            BD-ID            State
---------------------------------------
100            10               up          
[ce1] display  vxlan vni  200
VNI            BD-ID            State
---------------------------------------
200            20               up   
[ce1]display  mac-address 
Flags: * - Backup  
       # - forwarding logical interface, operations cannot be performed based 
           on the interface.
BD   : bridge-domain   Age : dynamic MAC learned time in seconds
---------------------------------------------------------------------------
MAC Address    VLAN/VSI/BD   Learned-From        Type                Age
---------------------------------------------------------------------------
fa7d-40c3-0031 1/-/-         GE1/0/2             dynamic               4805
7e79-3cbc-71e3 -/-/10        GE1/0/2.1           dynamic               1018
a625-ad01-5652 -/-/10        10.10.3.3           evn                   1017
5ecd-075a-5559 -/-/20        10.10.3.3           evn                     13
a692-4f84-88a9 -/-/20        GE1/0/2.2           dynamic                156

[ce3]display  mac-address 
Flags: * - Backup  
       # - forwarding logical interface, operations cannot be performed based 
           on the interface.
BD   : bridge-domain   Age : dynamic MAC learned time in seconds
----------------------------------------------------------------------------
MAC Address    VLAN/VSI/BD   Learned-From        Type                Age
----------------------------------------------------------------------------
fa7d-40c3-0041 1/-/-         GE1/0/2             dynamic               4843
7e79-3cbc-71e3 -/-/10        10.10.1.1           evn                   1070
a625-ad01-5652 -/-/10        GE1/0/2.1           dynamic               1070
5ecd-075a-5559 -/-/20        GE1/0/2.2           dynamic                208
a692-4f84-88a9 -/-/20        10.10.1.1           evn                     75
---------------------------------------------------------------------------
Total items: 5
```
### 控制平面分析
- 通过type3类型的路由，携带本地的VETPIP地址自动建立VXLAN隧道，生成头端复制列表泛洪BUM帧
- 通过type2类型路由，通告主机MAC和二层VNI，生成MAC地址转发表
### 转发平面分析
- 控制平面转发过程不需要通过自身的泛洪机制获取目标主机的MAC地址，而是通过EVPNtype2的路由，传递过来的目标主机MAC
- 但是主机的ARP广播依旧会通过头端复制列表进行泛洪BUM帧，区别是这个头端列表是通过BGP的Type3的路由生成的
## 控制平面引入EVPN-集中式网关
### 拓扑
![](/images/VXLAN控制平面引入EVPN集中式网关拓扑.png)
### 需求
1. PC1的IP得知为192.168.1.1，可以访问PC3的IP地址192.168.1.100，通过vxlan实现跨三层网络实现互通，evpn作为控制平面
2. PC1访问PC2，192.168.1.1访问192.168.2.1，CE2作为集中式网关，通过vxlan实现跨三层网络互通，evpn作为控制平面
### 步骤
1. 创建BD，BD创建EVPN实例，绑定子接口
2. 建立BGP evpn邻居
3. 配置L2层网关、配置L3层网关
### 命令
```
注：ENSP可能不通，需要重启设备
CE1配置：
evpn-overlay enable
#
bridge-domain 10
 vxlan vni 100
#
 evpn
  route-distinguisher 100:1
  vpn-target 2:2 export-extcommunity
  vpn-target 2:2 import-extcommunity
#
interface GE1/0/2.1 mode l2
 encapsulation dot1q vid 10
 bridge-domain 10
#
bgp 100
 private-4-byte-as enable
 peer 10.10.2.2 as-number 100
 peer 10.10.2.2 connect-interface LoopBack0
 #
 ipv4-family unicast
  peer 10.10.2.2 enable
 #
 l2vpn-family evpn
  policy vpn-target
  peer 10.10.2.2 enable
#
interface Nve1
 source 10.10.1.1
 vni 100 head-end peer-list protocol bgp
--------------------------------------------------------------------------
CE2配置：
#
evpn-overlay enable
#
bridge-domain 10
 vxlan vni 100
 #
 evpn
  route-distinguisher 100:1
  vpn-target 2:2 export-extcommunity
  vpn-target 2:2 import-extcommunity
#
bridge-domain 20
 vxlan vni 200
#
 evpn
  route-distinguisher 100:2
  vpn-target 3:3 export-extcommunity
  vpn-target 3:3 import-extcommunity
# 
bgp 100
 private-4-byte-as enable
 peer 10.10.1.1 as-number 100
 peer 10.10.1.1 connect-interface LoopBack0
 peer 10.10.3.3 as-number 100
 peer 10.10.3.3 connect-interface LoopBack0
#
 ipv4-family unicast
  peer 10.10.1.1 enable
  peer 10.10.3.3 enable
#
 l2vpn-family evpn
  undo policy vpn-target
  peer 10.10.1.1 enable
  peer 10.10.1.1 reflect-client
  peer 10.10.3.3 enable
  peer 10.10.3.3 reflect-client
#
interface Nve1
 source 10.10.2.2
 vni 100 head-end peer-list protocol bgp
 vni 200 head-end peer-list protocol bgp
#
interface Vbdif10
 ip address 192.168.1.254 255.255.255.0
#
interface Vbdif20
 ip address 192.168.2.254 255.255.255.0
-------------------------------------------------------------------------
CE3配置：
#
evpn-overlay enable
#
bridge-domain 10
 vxlan vni 100
 #
 evpn
  route-distinguisher 100:1
  vpn-target 2:2 export-extcommunity
  vpn-target 2:2 import-extcommunity
#
bridge-domain 20
 vxlan vni 200
 #
 evpn
  route-distinguisher 100:2
  vpn-target 3:3 export-extcommunity
  vpn-target 3:3 import-extcommunity
#
interface GE1/0/1.1 mode l2
 encapsulation dot1q vid 20
 bridge-domain 20
#
interface GE1/0/1.2 mode l2
 encapsulation dot1q vid 10
 bridge-domain 10
#
bgp 100
 private-4-byte-as enable
 peer 10.10.2.2 as-number 100
 peer 10.10.2.2 connect-interface LoopBack0
 #
 ipv4-family unicast
  peer 10.10.2.2 enable
 #
 l2vpn-family evpn
  policy vpn-target
  peer 10.10.2.2 enable
#
interface Nve1
 source 10.10.3.3
 vni 100 head-end peer-list protocol bgp
 vni 200 head-end peer-list protocol bgp

```
## 控制平面引入EVPN-集中式网关-外联网络
### 拓扑
![](/images/VXLAN控制平面引入EVPN集中式网关外联网络.png)
### 需求
园区网内部实现一网多用，对接出口设备
### 命令
```
Y园区配置：
CE1配置：
#
evpn-overlay enable
#
bridge-domain 10
 vxlan vni 100
 evpn
  route-distinguisher 100:1
  vpn-target 1:1 export-extcommunity
  vpn-target 1:1 import-extcommunity
#
bgp 100
 peer 10.10.2.2 as-number 100
 peer 10.10.2.2 connect-interface LoopBack0
#
 l2vpn-family evpn
  policy vpn-target
  peer 10.10.2.2 enable
#
ospf 1 router-id 1.1.1.1
 area 0.0.0.0
  network 10.1.12.1 0.0.0.0
  network 10.10.1.1 0.0.0.0
#
interface LoopBack0
 ip address 10.10.1.1 255.255.255.255
#
interface Nve1
 source 10.10.1.1
 vni 100 head-end peer-list protocol bgp
#
interface GE1/0/2.1 mode l2
 encapsulation dot1q vid 10
 bridge-domain 10

CE3配置：
#
evpn-overlay enable
#
bridge-domain 20
 vxlan vni 200
 evpn
  route-distinguisher 100:2
  vpn-target 2:2 export-extcommunity
  vpn-target 2:2 import-extcommunity
#
bgp 100
 peer 10.10.2.2 as-number 100
 peer 10.10.2.2 connect-interface LoopBack0
#
 l2vpn-family evpn
  policy vpn-target
  peer 10.10.2.2 enable
#
ospf 1 router-id 3.3.3.3
 area 0.0.0.0
  network 10.1.23.3 0.0.0.0
  network 10.10.3.3 0.0.0.0
#
interface LoopBack0
 ip address 10.10.3.3 255.255.255.255
#
interface Nve1
 source 10.10.3.3
 vni 200 head-end peer-list protocol bgp
#
interface GE1/0/1.1 mode l2
 encapsulation dot1q vid 20
 bridge-domain 20

CE2配置：
#
vlan batch 100 200
#
evpn-overlay enable
#
ip vpn-instance OA
 ipv4-family
  route-distinguisher 100:1
  vpn-target 2:2 export-extcommunity
  vpn-target 2:2 import-extcommunity
#
ip vpn-instance RD
 ipv4-family
  route-distinguisher 100:2
  vpn-target 3:3 export-extcommunity
  vpn-target 3:3 import-extcommunity
#
bridge-domain 10
 vxlan vni 100
 evpn
  route-distinguisher 100:1
  vpn-target 1:1 export-extcommunity
  vpn-target 1:1 import-extcommunity
#
bridge-domain 20
 vxlan vni 200
 evpn
  route-distinguisher 100:2
  vpn-target 2:2 export-extcommunity
  vpn-target 2:2 import-extcommunity
#
interface LoopBack0
 ip address 10.10.2.2 255.255.255.255
#
interface Nve1
 source 10.10.2.2
 vni 100 head-end peer-list protocol bgp
 vni 200 head-end peer-list protocol bgp
#
bgp 100
 peer 10.10.1.1 as-number 100
 peer 10.10.1.1 connect-interface LoopBack0
 peer 10.10.3.3 as-number 100
 peer 10.10.3.3 connect-interface LoopBack0
 #
 l2vpn-family evpn
  undo policy vpn-target
  peer 10.10.1.1 enable
  peer 10.10.3.3 enable
#
ospf 1 router-id 2.2.2.2
 area 0.0.0.0
  network 10.1.12.2 0.0.0.0
  network 10.1.23.2 0.0.0.0
  network 10.10.2.2 0.0.0.0
#
ospf 100 router-id 22.22.22.22 vpn-instance OA
 vpn-instance-capability simple
 area 0.0.0.0
  network 10.1.25.2 0.0.0.0
  network 192.168.1.254 0.0.0.0
#
ospf 200 router-id 22.22.22.22 vpn-instance RD
 vpn-instance-capability simple
 area 0.0.0.0
  network 10.1.25.3 0.0.0.0
  network 192.168.2.254 0.0.0.0
#
interface Vbdif10
 ip binding vpn-instance OA
 ip address 192.168.1.254 255.255.255.0
#
interface Vbdif20
 ip binding vpn-instance RD
 ip address 192.168.2.254 255.255.255.0
#
interface Vlanif100
 ip binding vpn-instance OA
 ip address 10.1.25.2 255.255.255.0
#
interface Vlanif200
 ip binding vpn-instance RD
 ip address 10.1.25.3 255.255.255.0
#
interface GE1/0/0
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094

AR5配置：
#
ip vpn-instance OA
 ipv4-family
  route-distinguisher 100:1
  vpn-target 2:2 export-extcommunity
  vpn-target 2:2 import-extcommunity
#
ip vpn-instance RD
 ipv4-family
  route-distinguisher 100:2
  vpn-target 3:3 export-extcommunity
  vpn-target 3:3 import-extcommunity
#
ospf 100 router-id 5.5.5.5 vpn-instance OA
 import-route bgp
 area 0.0.0.0 
  network 10.1.25.5 0.0.0.0 
#
ospf 200 router-id 6.6.6.6 vpn-instance RD
 import-route bgp
 area 0.0.0.0 
  network 10.1.25.6 0.0.0.0 
#
interface GigabitEthernet0/0/0.1
 dot1q termination vid 100
 ip binding vpn-instance OA
 ip address 10.1.25.5 255.255.255.0 
 arp broadcast enable
#
interface GigabitEthernet0/0/0.2
 dot1q termination vid 200
 ip binding vpn-instance RD
 ip address 10.1.25.6 255.255.255.0 
 arp broadcast enable
#
interface GigabitEthernet0/0/1.1
 dot1q termination vid 10
 ip binding vpn-instance OA
 ip address 10.1.15.5 255.255.255.0 
 arp broadcast enable
#
interface GigabitEthernet0/0/1.2
 dot1q termination vid 20
 ip binding vpn-instance RD
 ip address 10.1.15.6 255.255.255.0 
 arp broadcast enable
#
bgp 100
 router-id 55.55.55.55
 #
 ipv4-family vpn-instance OA 
  import-route ospf 100
  peer 10.1.15.1 as-number 200 
 #
 ipv4-family vpn-instance RD 
  import-route ospf 200
  peer 10.1.15.2 as-number 200 

```
## 控制平面引入EVPN-分布式网关
### 拓扑
![](/images/VXLAN控制平面引入EVPN分布式网关拓扑.png)
### 需求
使用分布式网关，实现PC1访问个人PC2
### 步骤
1. 配置IGP
2. 建立BGP EVPN邻居
3. 创建二层实例，三层实例
4. 创建NVE
5. 创建分布式网关绑定实例
6. 创建子接口绑定BD
- 注意事项：
  - EVPN邻居要配置通告IRB路由，默认不通告
  - 在BD EVPN下要设置三层VPN实例的RT出向值
### 命令
```
CE1配置：
evpn-overlay enable
#
bgp 100
 peer 10.10.3.3 as-number 100
 peer 10.10.3.3 connect-interface LoopBack0
#
 ipv4-family unicast
  peer 10.10.3.3 enable
#
 l2vpn-family evpn
  policy vpn-target
  peer 10.10.3.3 enable
  peer 10.10.3.3 advertise irb
#
ip vpn-instance a
 ipv4-family
  route-distinguisher 100:1
  vpn-target 3:3 export-extcommunity evpn
  vpn-target 3:3 import-extcommunity evpn
 vxlan vni 5010
#
bridge-domain 10
 vxlan vni 100
 evpn
  route-distinguisher 100:1
  vpn-target 2:2 export-extcommunity
  vpn-target 3:3 export-extcommunity       //type 2携带的4元组信息，
  既有IP信息又有mac信息，IP信息放入对端的L3 vpn实例要匹配入向RT,
  MAC信息放入对端的L2 VPN实例匹配入向RT
  vpn-target 2:2 import-extcommunity
#
interface Nve1
 source 10.10.1.1
 vni 100 head-end peer-list protocol bgp  //传递Type 3路由,生成BUM帧的泛洪表
#
interface Vbdif10
 ip binding vpn-instance a
 ip address 192.168.1.254 255.255.255.0
 mac-address 0000-5e00-0100          //同一个实例mac地址需要配置成相同的，
 对于M-LAG双活网关场景来说，对外虚拟成一个网关
 范围：0000-5e00-0100～0000-5e00-02ff
 vxlan anycast-gateway enable        //开启分布式网关功能
 arp collect host enable             //收集并发布主机路由
#            
interface GE1/0/0.1 mode l2
 encapsulation dot1q vid 100
 bridge-domain 10
--------------------------------------------------------------------------
CE3配置：
evpn-overlay enable
#
bgp 100
 peer 10.10.1.1 as-number 100
 peer 10.10.1.1 connect-interface LoopBack0
#
 ipv4-family unicast
  peer 10.10.1.1 enable
#
 l2vpn-family evpn
  policy vpn-target
  peer 10.10.1.1 enable
  peer 10.10.1.1 advertise irb
#
ip vpn-instance a
 ipv4-family
  route-distinguisher 100:1
  vpn-target 3:3 export-extcommunity evpn
  vpn-target 3:3 import-extcommunity evpn
 vxlan vni 5010
#
bridge-domain 20
 vxlan vni 200
 evpn
  route-distinguisher 100:1
  vpn-target 2:2 export-extcommunity
  vpn-target 3:3 export-extcommunity
  vpn-target 2:2 import-extcommunity
#
interface Nve1
 source 10.10.3.3
 vni 200 head-end peer-list protocol bgp
#
interface Vbdif20
 ip binding vpn-instance a
 ip address 192.168.2.254 255.255.255.0
 mac-address 0000-5e00-0102
 vxlan anycast-gateway enable        //开启分布式网关功能
 arp collect host enable             //收集主机路由，不开启就没有IRB路由
 没有IRB路由，也就不会通告给邻居
#
interface GE1/0/1.1 mode l2
 encapsulation dot1q vid 200
 bridge-domain 20
 =========================================================================
 查找EVPN实例路由：display  bgp evpn vpn-instance 20 routing-table mac-route
                  20就是BD的名字
 查找VPN实例路由：display  bgp vpnv4 vpn-instance a routing-table
```
### 转发控制平面分析
#### 控制平面
主机2.1上线，发送自己的ARP表到CE3，CE3收到后将形成的ARP表项放入BGP中，通过EVPN路由的type2IRB路由的形式（PC2IP+PC2MAC+二层VNI+三层VNI）将路由反射给CE2，CE2将收到的EVPN中的三层VNI和IP地址放入到IP VRF表中，将MAC地址二层VNI放入MAC VRF表中。此时IP VRF路由表中将多一条去往2.1的主机路由体哦啊木，下一跳是CE3的VTEP IP10.1.3.3
![](/images/VXLAN控制平面引入EVPN分布式网关控制平面.png)
#### 转发平面
1.1访问2.1报文封装为`1.1|2.1|PC1 MAC|1.254网关MAC`，将报文发给CE1，CE1网关收到后，查找VRF路由表，发现去往该目的地址的下一跳是对端的VTEP的地址10.1.3.3，进行VXLAN封装，报文格式为`S:1.1 D:2.1|S:网关MAC D:对端NVE的MAC|三层VNI5010|S:10.1.1.1 D:10.1.3.3|S:外层MAC D:MAC查路由表转发`，报文转发至CE2，CE2解封装发现是三层VNI，根据三层VNI查找VRF路由表，发现是vbdif20的直连，出接口是vbdif20，查找MAC地址表，如果有目的MAC对应的出接口就转发，没有就泛洪，找到去往目标地址对应的出接口，最终将报文转发给PC2
![](/images/VXLAN控制平面引入EVPN分布式网关转发平面.png)
## 控制平面引入EVPN-分布式网关-外联网络
### 拓扑
![](/images/控制平面引入EVPN分布式网关外联网络拓扑.png)
### 需求
使用分布式网关实现PC1、PC2能够访问PC4
### 步骤
- 配置VXLAN分布式网关
- 在CE2设备
  - 配置外部势力，绑定接口，在实例下配置静态路由去往外联网络
  - 配置BGP与CE1、CE3建立EVPN邻居
  - 配置BGP外部实例下引入静态路由，通告EVPN路由
  - 建立NVE二层网关，指定隧道源
- CE1、CE3上与CE2建立EVPN邻居，通告IRB路由
### 命令
```
注意ENSP可能不通需要重启该设备
CE1配置：
evpn-overlay enable
#
evpn
#
ip vpn-instance a
 ipv4-family
  route-distinguisher 100:1
  vpn-target 2:2 export-extcommunity evpn
  vpn-target 2:2 import-extcommunity evpn
 vxlan vni 5010
#
bridge-domain 10
 vxlan vni 100
 evpn
  route-distinguisher 100:1
  vpn-target 3:3 export-extcommunity
  vpn-target 2:2 export-extcommunity
  vpn-target 3:3 import-extcommunity
#
interface Vbdif10
 ip binding vpn-instance a
 ip address 192.168.1.254 255.255.255.0
 vxlan anycast-gateway enable
 arp collect host enable
#
interface GE1/0/0.1 mode l2
 encapsulation dot1q vid 100
 bridge-domain 10
#
interface GE1/0/1
 undo portswitch
 undo shutdown
 ip address 10.1.12.1 255.255.255.0
#
interface LoopBack0
 ip address 10.10.1.1 255.255.255.255
#
bgp 100
 peer 10.10.2.2 as-number 100
 peer 10.10.2.2 connect-interface LoopBack0
 #
 l2vpn-family evpn
  policy vpn-target
  peer 10.10.2.2 enable
  peer 10.10.2.2 advertise irb
#
ospf 1 router-id 1.1.1.1
 area 0.0.0.0
  network 10.1.12.1 0.0.0.0
  network 10.10.1.1 0.0.0.0
 #
interface Nve1
 source 10.10.1.1 /在没有二层互访的情况下不需要配置vni ，也能自动生成vxlan隧道
--------------------------------------------------------------------------
CE3配置：
evpn-overlay enable
#
ip vpn-instance a
 ipv4-family
  route-distinguisher 100:3
  vpn-target 2:2 export-extcommunity evpn
  vpn-target 2:2 import-extcommunity evpn
 vxlan vni 5010
#
bridge-domain 20
 vxlan vni 200
 evpn
  route-distinguisher 100:3
  vpn-target 3:3 export-extcommunity
  vpn-target 2:2 export-extcommunity
  vpn-target 3:3 import-extcommunity
#
interface Vbdif20
 ip binding vpn-instance a
 ip address 192.168.2.254 255.255.255.0
 vxlan anycast-gateway enable
 arp collect host enable
#
interface GE1/0/0
 undo portswitch
 undo shutdown
 ip address 10.1.23.3 255.255.255.0
#
interface GE1/0/1.1 mode l2
 encapsulation dot1q vid 200
 bridge-domain 20
#
interface LoopBack0
 ip address 10.10.3.3 255.255.255.255
#
bgp 100
 peer 10.10.2.2 as-number 100
 peer 10.10.2.2 connect-interface LoopBack0
 #
 l2vpn-family evpn
  policy vpn-target
  peer 10.10.2.2 enable
  peer 10.10.2.2 advertise irb
#
ospf 1 router-id 3.3.3.3
 area 0.0.0.0
  network 10.1.23.3 0.0.0.0
  network 10.10.3.3 0.0.0.0
#
interface Nve1
 source 10.10.3.3
CE2配置：
evpn-overlay enable
#
ip vpn-instance out
 ipv4-family
  route-distinguisher 100:2
  vpn-target 2:2 export-extcommunity evpn
  vpn-target 2:2 import-extcommunity evpn
 vxlan vni 5010
#
interface GE1/0/0
 undo portswitch
 undo shutdown
 ip address 10.1.12.2 255.255.255.0
#
interface GE1/0/1
 undo portswitch
 undo shutdown
 ip address 10.1.23.2 255.255.255.0
#
interface GE1/0/2
 undo portswitch
 undo shutdown
 ip binding vpn-instance out
 ip address 10.1.21.2 255.255.255.0
#
interface LoopBack0
 ip address 10.10.2.2 255.255.255.255
#
interface Nve1
 source 10.10.2.2
#
bgp 100
 peer 10.10.1.1 as-number 100
 peer 10.10.1.1 connect-interface LoopBack0
 peer 10.10.3.3 as-number 100
 peer 10.10.3.3 connect-interface LoopBack0
 #
 ipv4-family vpn-instance out
  import-route static
  advertise l2vpn evpn
 #
 l2vpn-family evpn
  undo policy vpn-target
  peer 10.10.1.1 enable
  peer 10.10.1.1 advertise irb
  peer 10.10.1.1 reflect-client
  peer 10.10.3.3 enable
  peer 10.10.3.3 advertise irb
  peer 10.10.3.3 reflect-client
#
ospf 1 router-id 2.2.2.2
 area 0.0.0.0
  network 10.1.12.2 0.0.0.0
  network 10.1.23.2 0.0.0.0
  network 10.10.2.2 0.0.0.0
#
ip route-static vpn-instance out 172.16.1.0 255.255.255.0 10.1.21.1
```
# 特性
## ARP广播抑制
### 场景
- BGP EVPN集中式网关场景下，用于发送主机ARP信息，给邻居表生成ARP广播抑制表type2的（三元组）信息
- BGP EVPN分布式网管场景下，用于发布主机路由信息（四元组）
### 解决的问题
减少首次访问ARP广播流量大量泛洪问题
### 原理
当三层网关学习到其子网下的主机ARP时，生成主机信息（包括主机IP地址、主机MAC地址、二层VNI、网关VTEP IP地址），然后通过type2传递ARP类型路由将主机信息同步到二层网关上。这样当二层网关再收到ARP请求时，先查找是否存在目的IP地址对应的主机信息，如果存在，则直接将ARP请求报文中的广播MAC地址替换为目的单播MAC地址，实现广播变单播，起到ARP广播抑制的目的
### 命令
```
配置：
BGP作为控制平面集中式网关场景
CE2配置：
interface Vbdif10
 ip address 192.168.1.254 255.255.255.0
 arp collect host enable
#
interface Vbdif20
 ip address 192.168.2.254 255.255.255.0
 arp collect host enable
#
bgp 100
 l2vpn-family evpn
  policy vpn-target
  peer 10.10.1.1 advertise arp
  peer 10.10.3.3 advertise arp
 
 CE1配置：
 bridge-domain 10
 vxlan vni 100
 evpn
  route-distinguisher 100:1
  vpn-target 2:2 export-extcommunity
  vpn-target 2:2 import-extcommunity
 arp broadcast-suppress enable
 查看：
 [ce1]display  arp broadcast-suppress user bridge-domain  10

```
## ARP代答
### 场景
BGP建立EVPN隧道，相同网段访问
### 原理
使用ARP二层代答功能之后，设备会将从终端收集上来的主机信息传递给邻居，此时当设备在收到ARP报文时，会先查看自己能否获取到该ARP请求报文的目的用户的信息，如果能够获取就直接进行ARP代答，否则按照原先的转发流程转发该报文。
### 命令
```
CE1配置：
bridge-domain 10
 vxlan vni 100
 evpn
  route-distinguisher 100:1
  vpn-target 2:2 export-extcommunity
  vpn-target 2:2 import-extcommunity
 arp l2-proxy enable        开启二层代答
 arp collect host enable    收集主机信息传给邻居
```