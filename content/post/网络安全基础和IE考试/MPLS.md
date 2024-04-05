---
author: Hugo Authors
title: MPLS及MPLS VPN
date: 2024-03-22
description:  网络安全基础和IE考试
series: 
  - 网络安全基础和IE考试

---
多协议标签交换MPLS（Multiprotocol Label Switching）是一种骨干网技术。早期是为了解决IP地址转发效率低的问题，转发过程中不在查找IP路由表，而是查二层的标签转发，实现快速转发，但是随着转发性能的提高，基于IP地址转发一统天下，MPLS淡出了人们的视线，但随着VPN的兴起。MPLS可以很好的与VPN结合，从而又以MPLS VPN的形式回到大众视线
<!--more-->
# MPLS
## 协议概述
### 术语
#### 按照位置划分
![](/images/MPLS位置解析图.png)
- MPLS域
  - 数据在传输过程中，会经过不同的环境，有二层以太网环境，三层IP环境，又有单播域、组播域、MPLS域即数据转发过程中经过运行了MPLS协议的网络设备
- LER（Lable Edge Router）
  - 位于MPLS域边缘、连接其他网络的成为边缘路由器LER
- LSR（Label Switching Router）
  - 位于MPLS内部的路由器称为LSR，标签交换路由器
- LSR-ID
  - 标识设备，手动配置，最好是设备的loopback接口
#### 按照数据处理方式
![](/images/MPLS按照数据处理方式转发图.png)
- 入节点
  - 定义：
    - LSP的起始节点，一条LSP只能有一个Ingress
  - 作用：
    - Ingress的主要功能是给报文压入`（PUSH）`一个新的标签，封装成MPLS报文进行转发
- 中间节点
  - 定义:
    - LSP的中间节点，一条LSP可能有多个Transit。
  - 作用：
    - Transit的主要功能是查找标签转发信息表，通过标签交换（SWAP）完成MPLS报文的转发
- 出节点（Egress）
  - 定义：
    - LSP的末节点，一条LSP只能有一个Egress
  - 作用：
    - Egress的主要功能是弹出标签（POP），恢复成原来的IP报文进行相应的转发
- FEC（转发等价类）
  - MPLS这里可以理解为路由
- Label
  - 用于唯一标识一个FEC
- LSP（Label Switched Path）
  - 标签交换路径。可以通过静态生成，一般不用，多用于动态表签分发协议自动生成（查FIB ）
### 报文结构
#### MPLS标签结构
![](/images/MPLS标签结构.png)
注：路由器如何判断报文是否使用标签转发，通过数据帧的type字段，arp：0806，ip：0800，门票六十：8847
![](/images/MPLS嵌套标签结构.png)
#### 字段解析
- Label：20bit，标签值域
  - 0~15：特殊标签
    - 0：显示空标签：Egress节点向倒数第二跳分配显示空标签，数据转发时不弹出标签，，在Egress不查标签转发表，直接查路由表 
    - 3：隐式空标签（默认分配）：Egress节点向倒数第二跳分配隐式空标签，数据转发时直接弹出标签
  - 16~1023：静态LSP和静态CR-LSP共享的标签空间（最少使用的）
  - 1024以上：LDP、MP-BGP、RSVP-TE动态信令协议都可以为MPLS分配标签，所以使用的标签控制范围（我们大部分使用的就是这种标签）
- Exp：3bit，用于扩展
  - 常用作CoS（Class of Service），其作用于Ethernet802.1p的作用类似
- S：1bit，栈底标识
  - MPLS支持多层标签，即标签嵌套。S值为1时表明为最底层标签，S值为0时表示还有其他标签
- TTL：8bit
  - 和IP分组中的TTL（Time To Live）意义相同。防止出现环路时报文无限制转发
## 工作原理
![](/images/MPLS工作原理.png)
### 控制平面
#### RIB
- 概念
  - （Routing Information Base）路由表，通过IGP协议生成IP路由表
- 作用
  - 存储路由信息，生成FIB表`（注：LDP为路由分配标签的前提是要有去往该目标地址的路由）`
![](/images/MPLS路由表.png)
#### LIB（标签数据库，标签信息表）
我们会想到为什么RIB表中都有路由通过了，我们何必再去找标签转发表呢，但在以前的技术不成熟的时候来说，依靠路由表转发的情况是很慢的，所以会采用路由表和标签转发表结合来使用
##### 概念
LDP给路由分配的标签（LSP）存放位置
![](/images/MPLS标签信息表.png)
##### 作用
存储标签信息，将最优的LSP信息加载到LFIB表中
##### LDP（Lable Distribution Protocol）
- 概念
  - 负责为FEC（路由）分配标签及建立LSP标签转发路径
- 原理
  - 报文（四种消息）
    - 发现（Discovery）消息：用于发现邻居，使用hello包携带`mpls lsr id`，该地址用于建立TCP链接（传递过程类似于OSPF），使用组播地址来发现邻居（224.0.0.2）
      - 既然使用TCP建立会话了，是不是就可以跨设备建立邻居了
    - 会话（Session）消息：只有当TCP建立连接后才能建立会话消息，TCP建立成功后，用于协商参数，建立LDP（标签分发协议）会话，使用消息类型为Initialization消息、Keepalive消息（类似于BGP的维持关系）
    - 通告（Advertisement）信息：给路由分配标签，通过Label Mapping字段携带（FEC与标签的映射）中心思想：下游给上有分配标签
    - 通知（Notification）消息：差错通知，跟BGP中的类似
  - 标签管理
    - 分发方式
      - `自主：不需要上游发起请求，下游就会给上有分配标签并传递（华为默认）`
      - 按需:必须上游发起请求，下游才能给上游分配标签
    - 控制方式
      - `有序：必须收到下游的标签才会给上游分配标签（华为默认）`
      - 无序：不需要收到下游的标签，也会给上游分发标签
    - 保留方式
      - `自由：会把最优的下一跳发来的标签放入LFIB（标签转发表）表中进行标签转发。次优邻居发来的标签放入LIB（标签数据库、标签转发信息表）中用于备份，当最优路径bown掉时，使用备份路径转发（华为默认）`
      - 保守：只保留最优下一跳发来的路由，没有备份
#### 控制平面解析
AR4上产生主机路由4.4.4.4/32，AR4默认为这条主机路由分配标签，`标签分发方向为从下游向上游逐跳分配，标签分发方向与数据转发方向是相反的，通过查看LSP表，根据标签分配原则（`入方向标签代表我给邻居分配的标签，出方向标签代表邻居给我分的入标签`）`
![](/images/MPLSAR4.png)
在AR3上查看路由表
![](/images/MPLSAR3.png)
在AR2上查看路由表
![](/images/MPLSAR2.png)
在AR1上查看路由表
![](/images/MPLSAR1.png)
### 转发平面
#### FIB
![](/images/MPLSFIB.png)
查`tunnel ID`值不为0则查找标签转发表`0x7`，如果为0则查找IP转发表`0x0`，准确来说是查找token值。
#### LFIB
![](/images/MPLSLFIB.png)
#### 转发平面解析
- Ingress的处理
  1. 查看FIB表，根据目的IP地址4.4.4.4找到对应的Tunnel ID
  2. 根据FIB表的Tunnel ID找到对应的NHLFE表项，将FIB表项和NHLFE表象关联起来
    ![](/images/MPLSFIB单独IP.png)
  3. 查看NHLFE表项，可以得到出接口、下一跳、出标签和标签操作类型、标签操作类型为push
    ```
    <ar1>display mpls lsp verbose
    ```
    ![](/images/MPLSNHLFE.png)
  4. 压入标签，并根据QoS策略处理EXP，同时处理TTL，然后封装好的MPLS报文发送给下一跳
- Transit的处理
  1. 根据MPLS的标签值查看对应的ILM（入接口标签映射）表的Token找到对应的NHLFE表项
    ![](/images/MPLSILM.png)
  2. 查看NHILFE表项，可以得到出接口、下一跳、出标签和标签操作类型swap，转发给下一跳
    ```
    <ar1>display mpls lsp verbose
    ```
    ![](/images/MPLSNHLFE查看Transit.png)
- Egress的处理
  - Egress节点收到MPLS报文后，查看ILM表获得标签操作类型，同时处理EXP和TTL
    - 如果标签中的S=1，表明该标签是栈底标签，直接进行IP转发
    - 如果标签中的S=0，表明还有下一层标签，继续进行下一层标签转发
  - 注：该场景描述的是次末跳标签封装的是非“0”或“3”

# MPLS VPN
## 公司与公司之间的互连方案
### 专线
- 裸光纤
- 运营商专线（SDH、PTN、波分）
- 优点：快、带宽大。缺点：昂贵
### VPN
#### 二层
- vpls
- evpn
- l2tp
#### 三层
- IPsec vpn：最常用，常用于企业对企业
- dsvpn：华为私有vpn
  - 深信服私有vpn：sanfor VPN
- GRE：不能加密
- mpls VPN
#### 应用层
- SSL
  - 用于个人到企业
  - 场景
    - 出差时用移动网络或者酒店网络连接公司内部网络
## 概述
- MPLS VPN使用BGP在运营商骨干网上发布VPN路由，使用MPLS在运营商骨干网上转发VPN报文，是一种常见的L3VPN技术
- 又分为单域和跨域就是跨运营商传路由并转发报文
- 通过划分实例来区分不同的网络（类似于防火墙的虚拟系统）将不同实例的路由表隔离开来
## 配置
### 拓扑
![](/images/MPLSVPN实验拓扑.png)
### 需求
- A总公司能访问A分公司，B总公司能访问B分公司，不同公司之间路由隔离
### 步骤
- 配置骨干网
  - 配置IGP
  - 配置BGP
  - 配置MPLS
- 创建实例
 - 配置RD
 - 配置RT
- CE与PE设备对接
  - 配置CE设备
  - 接口绑定OSPF实例
### 解析
#### 控制平面-传递路由
- 问题和解决方法
  1. 问题：PE设备收到不同站点发来的重叠的路由（不同站点地址重叠）
  - 解决方案：在PE设备上建立实例将不同的站点隔离，通过VRF（虚拟路由转发）进行路由表隔离
    - 分为三部解决：
      1. 创建一个实例
      2. 接口绑定到实例中
      3. 在实例下去建立BGP邻居关系
    - 做完这些后不同实例学到的路由就会自动进行隔离
  2. 问题：对端PE设备如何区分不同站点的路由（远端路有冲突）
  - 通过在IPV4上加路由区分符（RD）来区分不同站点的路由
    - RD：8字节
    - IPV4：四字节
  3. 问题：对端实例如何只接收总部传来的路由（路由导入到哪个VRF）
  - 通过匹配RT值解决，在两端PE设备上设置相同的RT值，只有当出口方向的RT值等于对端PE设备入方向的RT值，这样才能在特定的实例下接收路由。
- 路由传递过程
  - PE设备通过接口绑定学习到对应实例的IGP路由，并将该路由注入到对应的BGP实例中去，此时会转换为VPNv4路由，并携带RD值、RT值，以及MP-BGP打的内层标签传递给VPNv4邻居
  - 邻居设备会进行RT值检查，匹配RT后，放入VPNv4路由表，并转换为实例路由同时剥离RD值，保留内层私网标签
  - 字段所在位置
    - RT报文位置
      ![](/images/MPLSRT值位置.png)
    - 内层标签、RD值
      ![](/images/MPLS内层标签RD值位置.png)
#### 转发平面
通过MPLS来防止路由黑洞，这时候就要用到MPLS来弥补VPN的漏洞，当路由转发时运营商网络中不可能就几台机器可能中间会存在无数机器，可是要防止路由黑洞就要做全互联，这是不现实的，也不可能被实现，所以在没有全互联的网络中就不可避免地产生路由黑洞，此时我们采用标签进行转发就不会出现这种情况
- 问题及解决
  1. 问题解决数据包到达对端PE设备查那个虚拟路由表（VRF）转发
  - 解决方案：利用内层标签（MP-BGP分的）查找对应的VRF转发
  2. 问题：在公网中遇到路由黑洞怎么解决
  - 解决方案：利用外层标签（LDP`标签转发协议`分的）转发在公网（MPLS VPN网络）中进行转发
- 转发过程
  1. CE通过OSPF路由将数据包发给PE，PE在实例下以2.0为目标地址查找BGP路由表，下一跳为10.10.3.3
    ![](/images/MPLSRT值位置.png)
  2. 在根据10.10.3.3查找FIB表（此过程为迭代查询），发现tunnel值不为0，进行标签转发
    ![](/images/MPLSRT值位置.png)
  3. 根据标签转发表转发完成
    ![](/images/MPLSRT值位置.png)
  目标地址为10.10.3.3，压入外层标签，到达P节点进行标签交换，出方向是3号标签，直接弹出，露出私网标签，到达AR3后，根据私网标签找到对应的实例，在实例下查找OSPF路由实现转发
### 在这里提一个做项目时能用到的小技巧
- 当我们在做割接方案时总会在割接前后检查设备的连通性和运行的业务状态等信息，从这个地方我们可以用一条命令来检查其中的关键信息
  ```
    display bgp peer
  ```
- 其中显示最后一列显示，常常会被换行到下一行的一个东西`frcv`，他能表示我从邻居那里学来了多少路由，能直观的感受到割接前后设备的运行状态
### 配置命令
```
AR1配置
#
ip vpn-instance a
 ipv4-family
  route-distinguisher 1:1
  vpn-target 2:2 export-extcommunity
  vpn-target 2:2 import-extcommunity
#
ip vpn-instance b
 ipv4-family
  route-distinguisher 1:2
  vpn-target 3:3 export-extcommunity
  vpn-target 3:3 import-extcommunity
#
mpls lsr-id 10.10.1.1
mpls
#
mpls ldp
#
interface GigabitEthernet0/0/0
 ip binding vpn-instance a
 ip address 10.1.14.1 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 10.1.12.1 255.255.255.0 
 mpls
 mpls ldp
#
interface GigabitEthernet0/0/2
 ip binding vpn-instance b
 ip address 10.1.17.1 255.255.255.0 
#
interface LoopBack0
 ip address 10.10.1.1 255.255.255.255 
#
bgp 200
 peer 10.10.3.3 as-number 200 
 peer 10.10.3.3 connect-interface LoopBack0
 # 
 ipv4-family vpnv4
  policy vpn-target
  peer 10.10.3.3 enable
 #
 ipv4-family vpn-instance a 
  import-route ospf 100
 #
 ipv4-family vpn-instance b 
  import-route ospf 200
#
ospf 1 router-id 1.1.1.1 
 area 0.0.0.0 
  network 10.1.12.0 0.0.0.255 
  network 10.10.1.1 0.0.0.0 
#
ospf 100 router-id 1.1.1.1 vpn-instance a
 import-route bgp
 area 0.0.0.0 
  network 10.1.14.1 0.0.0.0 
#
ospf 200 router-id 1.1.1.1 vpn-instance b
 import-route bgp
 area 0.0.0.0 
  network 10.1.17.1 0.0.0.0

AR2配置
#
mpls lsr-id 10.10.2.2
mpls
#
mpls ldp
#
interface GigabitEthernet0/0/0
 ip address 10.1.12.2 255.255.255.0 
 mpls
 mpls ldp
#
interface GigabitEthernet0/0/1
 ip address 10.1.23.2 255.255.255.0 
 mpls
 mpls ldp
#
interface LoopBack0
 ip address 10.10.2.2 255.255.255.255 
#
ospf 1 router-id 2.2.2.2 
 area 0.0.0.0 
  network 10.1.12.0 0.0.0.255 
  network 10.1.23.0 0.0.0.255 
  network 10.10.2.2 0.0.0.0 
 
AR3配置
#
ip vpn-instance a
 ipv4-family
  route-distinguisher 1:1
  vpn-target 2:2 export-extcommunity
  vpn-target 2:2 import-extcommunity
#
ip vpn-instance b
 ipv4-family
  route-distinguisher 1:2
  vpn-target 3:3 export-extcommunity
  vpn-target 3:3 import-extcommunity
#
mpls lsr-id 10.10.3.3
mpls
#
mpls ldp
#
interface GigabitEthernet0/0/0
 ip address 10.1.23.3 255.255.255.0 
 mpls
 mpls ldp
#
interface GigabitEthernet0/0/1
 ip binding vpn-instance a
 ip address 10.1.35.3 255.255.255.0 
#
interface GigabitEthernet0/0/2
 ip binding vpn-instance b
 ip address 10.1.38.3 255.255.255.0 
#
interface LoopBack0
 ip address 10.10.3.3 255.255.255.255 
#
bgp 200
 peer 10.10.1.1 as-number 200 
 peer 10.10.1.1 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  peer 10.10.1.1 enable
 # 
 ipv4-family vpnv4
  policy vpn-target
  peer 10.10.1.1 enable
 #
 ipv4-family vpn-instance a 
  import-route ospf 100
 #
 ipv4-family vpn-instance b 
  import-route ospf 200
#
ospf 1 router-id 3.3.3.3 
 area 0.0.0.0 
  network 10.1.23.0 0.0.0.255 
  network 10.10.3.3 0.0.0.0 
#
ospf 100 router-id 3.3.3.3 vpn-instance a
 import-route bgp
 area 0.0.0.0 
  network 10.1.35.3 0.0.0.0 
#
ospf 200 router-id 3.3.3.3 vpn-instance b
 import-route bgp
 area 0.0.0.0 
  network 10.1.38.3 0.0.0.0 
```
## MPLS VPN扩展
### OSPF VPN扩展
#### 拓扑
  ![](/images/MPLSVPN实验拓扑.png)
#### 场景
PE和CE使用OSPF，宣告进区域0，中间的MPLS区域，相当于超级骨干区域，默认会以3类LSA形式通告到对端站点，虽然PE设备在BGP下引入的OSPF，但并不会以5类LSA的形式通告路由
#### 原理
通过update消息将描述这条LSA的详细信息在引入BGP是，依然携带OSPF路由的属性
##### 携带的OSPF的LSA信息
- LSA类型
- 区域号
- router ID
- RT
- Domian ID
  ![](/images/MPLSVPNOSPFdid.png)
##### 携带OSPF的LSA信息的字段
![](/images/MPLSVPNOSPF携带的LSA字段.png)
##### 命令解析
- domain ID 相同
  - 生成1、2类LSA，此处为1
  ![](/images/MPLSVPNOSPF12类LSA.png)
  - 生成的3类LSA，此处为3
  ![](/images/OSPFVPNOSPF3类LSA.png)
  - 5/7类LSA
    - 生成的是5类LSA引入时，type类型为1时，此处为5：0
    - 生成的是5类LSA引入时，type类型为2时，此处为5：1
    ![](/images/MPLSVPNOSPF5leiLSA.png)
- domain ID不相同时，生成的所有LSA传递过去都属于5类LSA
  