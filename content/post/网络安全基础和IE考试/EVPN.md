---
author: zty
title: EVPN
date: 2024-04-09
description:  网络安全基础和IE考试
series: 
  - 网络安全基础和IE考试
tags : [
    网络基础,HCIE考试,核心网
]
categories : [
    ENSP,
    华为设备配置
]
series : [HCIE考试]
aliases : [数通基础]
---
EVPN（Ethernet Virtual Private Network），它本身就属于BGP的一个拓展簇，利用BGP拓展协议来传递二层或三层的可达性信息，EVPN不像传统L2VPN VPLS那样，通过数据平面泛红学习MAC地址的，而是引入控制平面学习MAC和IP，实现了转控分离

EVPN解决传统L2VPN的典型问题，实现双活、快速收敛
<!--more-->
# VPLS VPN
## 协议概述
- VPLS：（Virtual Private LAN Service）虚拟专用局域网业务，是mpls二层vpn技术
- AC：（Attachment Circuit）接入电路，连接CE和PE的链路
- VSI：（Virtual Switch Instance）虚拟交换实例，相当于mpls vpn中的三层instance
- PW（Pseudo Wire）虚链路，VSI之间建立虚链路，用于承载二层数据帧的转发
- VSI ID：两端的VSI值必须相同，才能建立PW，数据帧只向相同VSI值内的PW进行ARP广播泛洪
- PW Signaling：PW信令协议，用于创建和维护PW，PW信令协议主要有LDP和BGP，我们这里使用LDP协议为二层实例跨设备分配内层标签
## 原理及配置
### 拓扑
![](/images/EVPNVPLSVPN拓扑图.png)
### 需求
两台PC属于同网段，现要实现跨越 骨干网互访
### 步骤
1. 配置骨干网
    - 配置ospf
    - 部署MPLS，开启二层VPLS
      - 建立LDP本地会话
      - 建立LDP远端会话
2. 创建二层实例
    - 选择信令协议LDP
    - 配置vsi id
    - 手动指定远端邻居
3. 创建子接口，终结vlan，接口绑定二层实例
### 配置
```
NE1配置：
第1步：配置骨干网
ospf 1 router-id 1.1.1.1
 area 0.0.0.0
  network 10.1.12.1 0.0.0.0
  network 10.10.1.1 0.0.0.0
#
mpls lsr-id 10.10.1.1
#
mpls
#
mpls l2vpn
#
mpls ldp
#
mpls ldp remote-peer 10.10.3.3
 remote-ip 10.10.3.3
#
interface Ethernet1/0/1
  mpls
  mpls ldp
第2步：创建实例
vsi a
 pwsignal ldp
  vsi-id 100
  peer 10.10.3.3 
第3步：创建子接口
interface Ethernet1/0/0.1
 vlan-type dot1q 1000
 l2 binding vsi a
 ----------------------------------------------------------------------
AR2配置：
mpls lsr-id 10.10.2.2
mpls
#
mpls l2vpn
#
mpls ldp
#
interface GigabitEthernet0/0/0
 mpls
 mpls ldp
#
interface GigabitEthernet0/0/1
 mpls
 mpls ldp
--------------------------------------------------------------------------
NE3配置：
第1步：配置骨干网
ospf 1 router-id 3.3.3.3
 area 0.0.0.0
  network 10.1.23.3 0.0.0.0
  network 10.10.3.3 0.0.0.0
#
 mpls lsr-id 10.10.3.3
#
mpls
#
mpls l2vpn
#
mpls ldp
#
mpls ldp remote-peer 10.10.1.1
 remote-ip 10.10.1.1
#
interface Ethernet1/0/1
  mpls
  mpls ldp
第2步：创建实例
vsi a
 pwsignal ldp
  vsi-id 100
  peer 10.10.1.1
第3步：创建子接口
interface Ethernet1/0/1.1
 vlan-type dot1q 500
 l2 binding vsi a
 ---------------------------------------------------------------------
1）查看内层标签信息
 [NE1]display vpls forwarding-info verbose
Total Number   : 1,        1  up,  0  down

**Vsi-Name      : a
  Vsi Index     : 2                    PwState      : UP
  Peer IP       : 10.10.3.3            VcOrSiteId   : 100
  InVcLabel     : 48125 我给VSI打的标签  OutVcLabel   : 48125
                                       邻居给我分的，转发过程中报文封装的内层标签
  BroadTunnelID : --                   OutInterface : LDP LSP
  MainPwToken   : 0x0                  SlavePwToken : 0x0
  NKey          : 0x100007C            CKey         : 0x41
2）查看外层标签信息
 [NE1]display mpls lsp 
Flag after Out IF: (I) - RLFA Iterated LSP, (I*) - Normal and RLFA Iterated LSP
Flag after LDP FRR: (L) - Logic FRR LSP 
-------------------------------------------------------------------------------
                 LSP Information: LDP LSP
-------------------------------------------------------------------------------
FEC                In/Out Label    In/Out IF                      Vrf Name
10.10.1.1/32       3/NULL          -/-                            
10.10.2.2/32       NULL/3          -/Eth1/0/1                     
10.10.2.2/32       48126/3         -/Eth1/0/1                     
10.10.3.3/32       NULL/1027       -/Eth1/0/1                     
10.10.3.3/32       48127/1027      -/Eth1/0/1  
3）查看终端MAC地址表项
[NE1]display  mac-address
MAC address table of slot 0:
-------------------------------------------------------------------------------
MAC Address    VLAN/BD/    PEVLAN CEVLAN Port/Peerip     Type      LSP/LSR-ID
               VSI/SI/EVPN                                         MAC-Tunnel
-------------------------------------------------------------------------------
5489-98ff-088e a           -      -      Eth1/0/1        dynamic   0/128       
5489-9817-6e7a a           1000   -      Eth1/0/0.1      dynamic   0/0         
-------------------------------------------------------------------------------
Total matching items on slot 0 displayed = 2
4）抓包
```
![](/images/MEVPNVPLSVPN抓包.png)
### 实现过程
- 数据包发送：当PC1访问PC2，要获取对端主机的MAC，会发送arp广播帧，到达NE1接口1/0/0子接口，进入VSI实例，封装数据帧，加上内层标签，和外层标签，通过PW虚链路，向具有相同的vsi id值泛红流量，封装格式为`S;ip 1.1 Dip 1.2 S:MAC pc6 D:MAC ff 内层标签48125 外层标签1027 外层MAC`，经过公网设备AR2进行外层标签转发，剥离外层标签，到达NE3根据内层标签进入到对应的VSI接收广播帧进行泛洪，流量到达PC7，单播回复ARP请求
- 数据包回包：数据帧到达NE3，进入VSI，在对应的VSI实例下查找MAC地址表，根据MAC地址表转发，同时封装内层标签，外层标签，到达NE1后，根据内层标签关联到对应的VSI，查找MAC地址表，根据MAC地址表转发给PC6，PC6获取目标主机MAC地址后，进行ICMP封装，实现二层被访问
#### 内层标签
- 谁分配的：LDP通过非直连的邻居分配的
- 作用：用于指导数据帧转发时查看哪个VSI的MAC地址表转发
- 为谁分配的：为VSI实例分配标签
  - mpls L3 vpn是由mp-bgp为路由分配死亡标签，用于转发层面
  - vpls中是通过LDP为非直连的邻居分配标签，用于转发层面
#### 外层标签
- 公网中转发数据
- 理解：不像三层VPN由控制平面和转发平面，VPLS中获取对方MAC地址是通过转发平面自身ARP报文泛红学习的
## 不足
### 无法实现负载分担
在CE双归属场景中，如果双主会形成环路，因此需要设置主备模式，只有主转发流量，被链路不转发流量，只作为冗余备份，导致流量无法负载分担
![](/images/EVPNVPLSVPN不足1.png)
如何实现冗余的：创建E-trunk，将eth-trunk接口加入到E-trunk口下，在两台E-trunk之间选举一台主，另一台为备，只对主对应的端口转发流量，备不转发
![](/images/EVPNVPLSVPN不足2.png)
### 故障收敛速度慢
![](/images/EVPNVPLSVPN不足3.png)
# EVPN
## 协议概述
- EVPN存在三个表
  - ESI成员信息表
  - BUM帧流量信息表
  - MAC地址转发表
### 术语
- ES：（Ethernet Segment）代表用户站点或设备连接到PE的链路，类似于VPLS中的AC设备
- EVI：（EVPN instance）代表一个EVPN实例，相当于三层VPN中的instance
- ESI：ESI Lable是一个数值，EVPN Type 1路由所携带的团体属性，在多归属场景下，用于实现快速收敛和水平分割，同侧配置相同的，不同侧PE设置不同的，同一台设备接口不能冲突
- DF：（Designated Forwarder）用于在CE多归属场景下之转发一份BUM流量至CE
- BUM帧：（Broadcast（广播）、Unknown unicast（未知单播）、Multicast（组播））Label是由Type3路由携带，用于转发BUM流量
## 工作原理
### 控制平面
#### 问题及解决方法
##### 无流量触发时
- BUM帧处理方式
  ![](/images/EVPNtype3BUM帧标签.png)
  - 解决办法：`Type3`为`BUM帧分配内层标签`，构建BUM流量转发表（类似于MPLS VPN的私网标签）后期用于转发流量，这也是形成的第一个表项
- CE双归属场景，对端重复的PE流量转发给CE
  - 解决办法`Type4`携带ESI和source IP address在PE间选举DF，只有DF路由器转发广播流量到CE，这样既能防止环路，也能实现VPLS实现不了的负载分担问题（有一点点小问题：这个东西只应用于双归属场景，单播是不起作用的，在四种类型中只有type4是无标签）
  - 原理：手动在PE与CE互联的物理接口下配置ESI值
    - 同侧PE设备设置一样的ESI值进行DF的选举
    - 不同侧PE设备设置不一样的ESI，不进行DF选举，而且ESI不一样都不会接收此TYPE类型
  - 选举原则：基于Type4路由中携带的source IP address，地址小的为DF
- CE双归属场景，站点内流量又返回本站点
  - 解决办法：`Type1`为`ESI分发标签`，当本端PE收到对端PE发来的标签，转发时进行内层标签封装，当对端PE发现该标签是我分配的，就不会将流量转发给CE1（其实就是当一个PE发送流量时会进行泛洪所有的PE都会接收到，因为Type1为ESI分发了标签，所以当流量泛洪时，会进行Type1的检查，这时发现是自己分配的就不会接收了）
    ![](/images/EVPNtype1标签.png)
##### 有流量触发时
- 单薄MAC地址通告
  - 解决办法：`Type2`为单播MAC地址通告标签（MAC-VRF）
    ![](/images/MEVPNtype2标签.png)
    这张图片为流量为空（EVPN启动阶段生成的，因为初始所以为空）示意图，也代表着前几个标签转发过后最终形成的几张表
#### 过程描述
Type3（构建BUM转发表）——>Type4（DF选举）——>Type1（通告ESI标签，水平分割）——>Type2（EVPN流量转发）
##### 第一步
刚运行EVPN时，还没形成MAC地址表，没有已知MAC，不能为单播MAC分发标签，因为没有流量出发；此时PE会为BUM帧分配标签，使用`type3`
![](/images/EVPNtype3BUM帧标签.png)
##### 第二步
进行选举DF，手动配置ESI值，通过`type4`路由中携带ESI值和Source ip，ESI值相同的PE路由器接收type4路由后会进行DF选举，Source ip小的会成为DF，只有DF会向CE转发BUM帧，非DF不转发，防止重复流量发给CE
##### 第三步
PE设备通过`type1`路由为ESI分发标签，接收到type1标签的本端PE设备，在数据转发时会将该标签封装为内层标签，当数据包到达本端的另一台PE设备时，发现这个内层标签是自己分配出去的，则不会将流量再转发给CE1
![](/images/EVPNtype1标签.png)
##### 第四步
PE1会生成单播MAC地址，通过`type2`路由可以携带本端PE上EVPN实例的RD值、ESI值以及EVPN实例对应的死亡标签，并通告给PE2和远端PE3、PE4，远端PE3、PE4设备生成MAC地址表
![](/images/EVPN第四步1.png)
PE2收到type2后，感知到自己与CE1是直连，刷新MAC地址表，并以type2路由形式继续通告该MAC
![](/images/EVPN第四步2.png)
PE3和PE4会收到两条MAC地址，分别是PE1和PE2发来的，转发时会通过负载分担转发流量
![](/images/EVPN第四步3.png)
`当CE2侧有流量触发时，过程与CE1侧出发流量完全相同`
![](/images/EVPN第四步4.png)
![](/images/EVPN第四步5.png)
![](/images/EVPN第四步6.png)
此时PE1与PE2也会收到两条MAC地址，分别是PE3和PE4发来的，用于`负载分担`转发流量，控制平面构建完成MAC地址后，进入转发平面
`其中type2表项的形式其实也是转发平面流量触发的，只有type3、type4、type1是真正意义上的控制平面形成的表项`
### 转发平面
#### 去程
由于该ARP报文目的MAC是广播报文，因此通过匹配BUM帧转发表，向BUM帧转发表中的目的PE泛洪，同时封装内层标签`（Type3分配的）`和外层标签`（LDP分配的）`
- 转发给对端PE
  - BUM帧封装两层标签，内层BUM帧标签，外层LDP
  - 只有DF会继续通告该广播流量给CE（Type4选举的）
  - 非DF收到广播流量，不通告给CE
- 转发给本端PE
  - BUM帧封装`三层`标签，内层标签装`（Type分配）`的用于水平分割、防止环路，当本端CE收到Type1的标签发现是自己分配的，则不会接收；中间标签是`（Type3分配）`用于决定流量通过后在哪个VRF下转发广播流量；外层标签`（LDP分的）`用于在公网中转发（迭代查询）
#### 回程
对端CE收到ARP请求后，单播回复ARP请求
- 负载分担到一台PE上，因为通过E-trunk做的跨设备链路聚合多活模式，链路可以实现负载分担
- 假设到达PE3后，形成MAC地址表了，开始干控制平面的活了，把路由通告给邻居之后通过单播MAC表转发，并且负载分担到其中一条MAC地址表封装内层标签（Type2分配的）和外层标签转发（LDP分配的）
- 对端PE1或PE2收到后，根据MAC地址表转发给CE1
## 配置
### 二层VPN
#### 拓扑
![](/images/EVPN二层VPN拓扑.png)
#### 需求
10.1.1.1能访问10.1.1.2
#### 步骤
- 骨干网配置
  - 配置OSPF
  - 配置LDP
  - 建立EVPN邻居
- 绑定实例
  - 创建EVPN实例
  - BD和EVPN绑定，配置源地址
  - 物理口配置ESI
  - 子接口下绑定BD（桥接）
#### 命令
```
NE1配置：
ospf 1 router-id 1.1.1.1
 area 0.0.0.0
  network 10.1.12.1 0.0.0.0
  network 10.10.1.1 0.0.0.0
#
mpls lsr-id 10.10.1.1
#
mpls
#
mpls ldp
#
bgp 100
 peer 10.10.3.3 as-number 100
 peer 10.10.3.3 connect-interface LoopBack0
 l2vpn-family evpn
  undo policy vpn-target
  peer 10.10.3.3 enable
#
evpn vpn-instance 1 bd-mode
 route-distinguisher 1:1
 vpn-target 2:2 export-extcommunity
 vpn-target 2:2 import-extcommunity
#
bridge-domain 10
 evpn binding vpn-instance 1
#
evpn source-address 10.10.1.1
#
interface Ethernet1/0/1
 esi 0000.0000.0000.0000.1111
#
interface Ethernet1/0/1.1 mode l2
 encapsulation dot1q vid 10
 rewrite pop single                 /对接收的报文进行剥除VLAN Tag操作
 bridge-domain 10
 --------------------------------------------------------------------
NE2配置：
ospf 1 router-id 2.2.2.2
 area 0.0.0.0
  network 10.1.12.2 0.0.0.0
  network 10.1.23.2 0.0.0.0
  network 10.10.2.2 0.0.0.0
#
mpls lsr-id 10.10.2.2
#
mpls
#
mpls ldp
#
interface Ethernet1/0/0
 mpls
 mpls ldp
#
interface Ethernet1/0/1
 mpls
 mpls ldp
--------------------------------------------------------------------
NE3配置：
ospf 1 router-id 3.3.3.3
 area 0.0.0.0
  network 10.1.23.3 0.0.0.0
  network 10.10.3.3 0.0.0.0
#
mpls lsr-id 10.10.3.3
#
mpls
#
mpls ldp
#
bgp 100
 peer 10.10.1.1 as-number 100
 peer 10.10.1.1 connect-interface LoopBack0
 l2vpn-family evpn
  undo policy vpn-target
  peer 10.10.1.1 enable
#
evpn vpn-instance 1 bd-mode
 route-distinguisher 2:1
 vpn-target 2:2 export-extcommunity
 vpn-target 2:2 import-extcommunity
#
bridge-domain 10
 evpn binding vpn-instance 1
#
evpn source-address 10.10.3.3
#
interface Ethernet1/0/1
 esi 0000.0000.0000.0000.3333
#
interface Ethernet1/0/1.1 mode l2
 encapsulation dot1q vid 10
 rewrite pop single
 bridge-domain 10
=======================================================================
1.查看EVPN邻居
<ne1>display  bgp evpn  peer 
2.查看Type类型路由
display  bgp evpn  all routing-table 
参数：inclusive-route   type 3
     es-route          type 4
     ad-route          type 1
     mac-route         type 2
3.查看DF选举结果
 <ne1>display evpn vpn-instance name 1 df result  
ESI Count: 1
ESI: 0000.0000.0000.0000.1111
 IFName Ethernet1/0/1.1:
  DF Result    : Primary
 
扩展实验： 
```
![](/images/EVPN二层拓展实验.png)
这玩应只能用华为内部ensp来做实验
```
1.查看转发表display  evpn mac routing-table  
all-evpn-instance 
EVPN name: a
MACs: 4         IndirectID Entries: 4          

MAC-Address           VLAN/BD          PeerIP                                  Type      Interface           
36e5-8242-9966          --/10          10.10.3.3                               Dynamic   LDP LSP 
70cb-752c-48a5          --/10          --                                      Dynamic   Ethernet3/0/1.2 
dcdb-0552-8731          --/10          10.10.3.3                               Dynamic   LDP LSP 
de82-7bed-3205          --/10          --                                      Dynamic   Ethernet3/0/1.1 
------------------------------------------------------------------------------
EVPN name: 1
MACs: 2         IndirectID Entries: 2          

MAC-Address           VLAN/BD          PeerIP                                  Type      Interface           
faa9-7d7b-0026          --/20          --                                      Dynamic   Ethernet3/0/2.1 
faa9-7d7b-0036          --/20          10.10.3.3                               Dynamic   LDP LSP 
2.查看MAC地址表
[NE1]display  mac-address
MAC address table of slot 3:
----------------------------------------------------------------------------------------------------------------------
MAC Address    VLAN/BD/                        PEVLAN CEVLAN Port/Peerip                        Type      LSP/LSR-ID
               VSI/SI/EVPN                                                                                MAC-Tunnel
----------------------------------------------------------------------------------------------------------------------
faa9-7d7b-0026 BD 20                           200    -      Eth3/0/2.1                         dynamic       3/-         
faa9-7d7b-0036 BD 20                           -      -      10.10.3.3                          dynamic       3/-         
dcdb-0552-8731 BD 10                           -      -      10.10.3.3                          dynamic       3/-         
de82-7bed-3205 BD 10                           10     -      Eth3/0/1.1                         dynamic       3/-         
36e5-8242-9966 BD 10                           -      -      10.10.3.3                          dynamic       3/-         
70cb-752c-48a5 BD 10                           100    -      Eth3/0/1.2                         dynamic       3/-                       dynamic       3/-  

                          Dynamic   LDP LSP   
```
#### 解析
##### 控制平面
没有流量触发之前，运行EVPN以后，hi通过MP-BGP的updata报文的type3类型为BUM帧分配标签
![](/images/EVPN二层VPN控制平面1.png)
如果连接站点的物理接口配置了ESI，则会通告Type4用于DF选举
![](/images/EVPN二层VPN控制平面2.png)
和Type1路由一样，都是用于防环
![](/images/EVPN二层VPN控制平面3.png)
如果此时SW1发起访问，会通告Type2路由，为MAC地址分配标签，此时如果匹配了RT值就会接收以上三种不同类型的EVPN路由
![](/images/EVPN二层VPN控制平面4.png)
##### 转发平面
`注：ENSP上没有MAC表项`
![](/images/EVPN二层VPN转发平面1.png)
- 去程：
  - S1发送ARP广播报文，数据包到达NE1后封装Type3分配的内层标签，再封装LDP分配的外层标签，通过外层标签转发
  - 到达NE2通过外层标签转发，出方向玻璃标签，数据包到达NE3后，根据内层标签决定在那个VRF下泛洪ARP广播流量，此时S2会接收到该ARP请求，立即恢复请求
  - 去程抓包
    ![](/images/EVPN二层VPN转发平面去程抓包.png)
- 回程
  - 数据包到达NE3，根据MAC地址表转发，去往S1，出接口为10.10.1.1，封装内网标签（type2通告的），外层通过公网标签（ldp通告的）转发给NE1
    ![](/images/EVPN二层VPN转发平面回程1.png)
    NE1通过查找MAC地址表转发给S1，出接口为子接口
    ![](/images/EVPN二层VPN转发平面回程2.png)
  - 回程抓包
    ![](/images/EVPN二层VPN转发平面回程抓包.png)
### 三层VPN
#### 拓扑
![](/images/EVPN三层VPN拓扑.png)
#### 需求
PC1与PC2属于不同网段，跨运营商实现互访
#### 步骤
- 骨干网配置
  - 配置OSPF
  - 配置LDP
  - 建立EVPN邻居
- 对接站点
  - 创建三层实例
  - 在BGP实例下对接站点，建立EBGP邻居
  - 接口绑定实例
#### 命令
```
NE1配置：
ospf 1 router-id 1.1.1.1
 area 0.0.0.0
  network 10.1.12.1 0.0.0.0
  network 10.10.1.1 0.0.0.0
#
mpls lsr-id 10.10.1.1
#
mpls
#
mpls ldp
#
ip vpn-instance a
 ipv4-family
  route-distinguisher 100:1
  vpn-target 2:2 export-extcommunity evpn  
  vpn-target 2:2 import-extcommunity evpn
  evpn mpls routing-enable  //使能EVPN生成和发布IP前缀路由和IRB路由的功能，
  只有控制平面用EVPN，转发平面是MPLS或SR-MPLS时才需要此命令，控制平面是vpnv4时
  是自动的将实例的路由转换为vpnv4路由
#
bgp 100
 peer 10.10.3.3 as-number 100
 peer 10.10.3.3 connect-interface LoopBack0
#
 ipv4-family unicast
  undo synchronization
  peer 10.10.3.3 enable
#
 ipv4-family vpn-instance a
  advertise l2vpn evpn       //默认是给二层实例传MAC的，
  现在要用EVPN传路由了需要将本端收集到的实例路由，
  通过EVPN邻居type 5的形式发送到对端，需要告知设备
    注：*两条蓝色命令必须同时开启才会生成type 5并通告evpn路由,
    vpnv4是默认通告实例路由的*
  peer 10.1.11.2 as-number 200
#
 l2vpn-family evpn
  policy vpn-target
  peer 10.10.3.3 enable
#
interface Ethernet1/0/1
 ip binding vpn-instance a
 ip address 10.1.11.1 255.255.255.0
 ----------------------------------------------------------------
NE2配置：
ospf 1 router-id 2.2.2.2
 area 0.0.0.0
  network 10.1.12.2 0.0.0.0
  network 10.1.23.2 0.0.0.0
  network 10.10.2.2 0.0.0.0
#
mpls lsr-id 10.10.2.2
#
mpls
#
mpls ldp
#
interface Ethernet1/0/0
 mpls
 mpls ldp
#
interface Ethernet1/0/1
 mpls
 mpls ldp
----------------------------------------------------------------
NE3配置：
ospf 1 router-id 3.3.3.3
 area 0.0.0.0
  network 10.1.23.3 0.0.0.0
  network 10.10.3.3 0.0.0.0
#
mpls lsr-id 10.10.3.3
#
mpls
#
mpls ldp
#
ip vpn-instance a
 ipv4-family
  route-distinguisher 100:2
  vpn-target 2:2 export-extcommunity evpn
  vpn-target 2:2 import-extcommunity evpn
  evpn mpls routing-enable   //用于本端设备向EVPN对等
  体发布IP前缀路由和IRB路由
#
bgp 100
 peer 10.10.1.1 as-number 100
 peer 10.10.1.1 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  peer 10.10.1.1 enable
 #
 ipv4-family vpn-instance a
  advertise l2vpn evpn       //配置发布IP前缀类型的路由
  peer 10.1.34.4 as-number 300
 #
 l2vpn-family evpn           //进入BGP-EVPN地址族
  policy vpn-target
  peer 10.10.1.1 enable
 #
interface Ethernet1/0/1
 ip binding vpn-instance a
 ip address 10.1.34.3 255.255.255.0
===================================================================
1.查看EVPN邻居
<ne1>display  bgp evpn  peer 
2.查看Type类型路由
display  bgp evpn  all routing-table prefix-route                                   
```
#### 解析
##### 控制平面
Type5传递路由和标签
![](/images/EVPN三层VPN控制平面Type5传递路由和标签.png)
##### 转发平面
与MPLS L3层VPN转发过程非常类似