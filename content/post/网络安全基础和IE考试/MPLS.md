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
  undo synchronization      //用来关闭BGP与IGP的同步功能
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
### OSPF VPN环路
#### 场景
CE双归属场景
#### 类型
- 三类防环
  - 拓扑
    ![](/images/MPLSVPNOSPF35类防环.png)
  - 问题
    - 次优路径
      - PE1收到CE1的路由，通过VPNv4邻居传递给反射器，反射器会反射给PE2和PE3，PE2引入BGP路由到OSPF实例中，此时PE3会从PE2收到该条路由，PE3从OSPF学来的路由优先级为150，从BGP学来的路由优先级为255，因此会选择PE2通过的路由，此时PE3访问192.168.1.0，则形成次优路径
    - 环路
      - 在PE2和PE3都进行了BGP和OSPF的双向引入，当PE2在OSPF中引入BGP路由，会生成3类LSA通告给PE3，PE3优选从PE2通告的3类LSA计算路由，放入IP路由表中，此时PE3会将OSPF再引入BGP中。
      - 此时PE3会自己通过引入生成192.168.1.0的路由，也会从反射器收到192.168.1.0的路由，因为11条选录原则中，从本地生成的路由优于从邻居学来的，所以PE3会通告自己通过引入生成的路由给PE2
      - 此时PE2会分别从PE3和PE1收到192.168.1.0的路由，如果根据选路原则优选了PE3发来的，则形成环路，如果优选PE1发来的，则无环路
  - 解决办法
    - DN位置位
      - 因为OSPF100的进程绑定了VPN，此时OSPF通告这个3类LSA时自动就会将该3类LSA中的DN位置位，CE2收到这条3类LSA，不会检查DN位，会接收和计算路由，因为CE2的OSPF进程没有绑定实例，PE3收到后只接收，不计算路由，因为PE3也绑定了实例，所以就会检查DN位
      - 正常情况下不用设置任何东西，CE就会对PE传过来的3类LSA进行DN位职位的操作
    - 这里重点解释一下什么是DN位
      - 它用来表示一个方向（down），当DN位被置位时说明这条LSA由PE发给CE。值得注意的是DN位只能用于3类LSA
      - OSPF规定，PE在接收到DN位已经置位了的LSA之后，PE路由器在OSPF路由的SPF计算期间忽略该路由，并不把这条LSA重发布到BGP中去
    - 还有一种情况如果我不需要这种DN位置位的情况（就比如我的两个PE都和CE做了实例，来进行连接，这样的情况下我们就不能让DN位置位了，因为这样，有可能导致其他路由不通），在这种情况下，我们就需要禁止DN位置位
      - 命令
        ```
        vpn-instance-capability simple
        这个命令也可以有效防止因为tag标签导致的5类LSA隔离导致路由不能计算
        ```
- 五类防环
  - 拓扑
    ![](/images/MPLSVPNOSPF35类防环.png)
  - 解决办法
    - tag：默认5类LSA中tag是1
      - OSPF route-tag只用于私网。在PE上当OSPF发现一条五类LSA的tag和自己的route-tag的一样，就会忽略这条路由不进行处理，用于防止CE双归属是，5类LSA发生环路
    - PE2在OSPF中引入BGP后使用BGP的AS号计算5类LSA的tag，通告5类LSA，CE不检查tag，PE3会用本地的tag检查接收到五类LSA中的tag，如果相同就直接收不计算
### 组网
#### Hub-Spoke
- PE-CE都运行BGP
  - 拓扑
    ![](/images/MPLSVPNHub-Spoke模型.png)
  - 需求
    - 分公司都能访问总公司
    - 分公司互访，流量需要经过总公司
  - 步骤
    1. 配置骨干网
        - 配置IGP，使用ISIS，建立L2的邻居关系
        - 创建 BGP VPNv4邻居
        - 配置MPLS
    2. 配置示例
        - 配置RD
        - 配置RT
    3. 对接站点
  - 配置
    ```
    PE2部分配置
    interface GigabitEthernet0/0/1.1
    dot1q termination vid 10
    ip binding vpn-instance vpn_in
    ip address 10.1.6.2 255.255.255.0 
    arp broadcast enable
    #
    interface GigabitEthernet0/0/1.2
    dot1q termination vid 20
    ip binding vpn-instance vpn_out
    ip address 10.1.7.2 255.255.255.0 
    arp broadcast enable
    #
    bgp 300
    peer 10.10.4.4 as-number 300 
    peer 10.10.4.4 connect-interface LoopBack0
    #
    ipv4-family unicast
      undo synchronization
      peer 10.10.4.4 enable
    # 
    ipv4-family vpnv4
      policy vpn-target
      peer 10.10.4.4 enable
    #
    ipv4-family vpn-instance vpn_in 
      peer 10.1.6.3 as-number 400 
    #当AS号相同时as-path会触发防环，丢弃接收来的路由，所以我们需要配置以下
    ipv4-family vpn-instance vpn_out 
      peer 10.1.7.3 as-number 400 
      peer 10.1.7.3 allow-as-loop 1  AS号允许重复1次，默认为1 /PE2传给PE3，
      PE3不需要配置允许AS号重复，因为是IBGP邻居,只在EBGP邻居检测
    ```
  - 特殊注解
    - Hub-Spoke只是一种特殊的组网方式，控制平面传递路由和数据转发平面转发数据，原理都是一样的，本质都是通过匹配RT值，决定了路由注入到那个实例中，从而影响转发路径
    - PE2不能用一个实例，因为一般情况下从一个接口发出去的路由，不能从接口再收到，防止环路，该场景中会收到传回来的路由，但不是最优的路由，也就不满足Hub-Spoke组网，即分公司之间的流量不会经过总部的CE设备转发
    - EBGP传递IBGP邻居是本应该不改变下一跳而在VPNv4中会改变
- PE-CE都运行OSPF
  - 拓扑
    ![](/images/MPLSVPNHub-Spoke模型.png)
  - 需求
    1. 分公司能够访问总公司
    2. 分公司互访，流量需要经过总公司
    3. CE-spoke与PE-spoke之间运行OSPF，CE-hub与PE-hub之间运行OSPF
  - 配置
    ```
    PE2配置：
    ospf 100 vpn-instance vpn_in
    import-route bgp
    area 0.0.0.0 
      network 10.1.6.1 0.0.0.0 
    #
    ospf 200 vpn-instance vpn_out
    vpn-instance-capability simple
    area 0.0.0.0 
      network 10.1.6.5 0.0.0.0 

    CE3配置：
    ospf 200 
    area 0.0.0.0 
      network 10.1.6.2 0.0.0.0 
      network 10.1.6.6 0.0.0.0 

    ```
  - 解析
    - 该场景中因为OSPF VPN在绑定多实例场景认为会有环路，因此在发送3、5、7类LSA时，默认会将LSA中的DN位置位，此时另一台PE收到DN位置位的LSA后就不会计算OSPF路由，防止环路，但是在该场景中，没有环路也会导致vpn out收到该LSA无法计算路由，导致业务不通，因此需要使用`vpn-instance-capability simple`命令，不检查DN Bit和Route-tag 而直接计算出所有OSPF路由 
# MPLS跨域VPN
## option A
### 案例
- 拓扑
  ![](/images/MPLS跨域VPN.png)
- 需求
  - PC1和PC3跨AS实现互访
- 步骤
  1. 分别配置单域AS100、AS200
      - 配置OSPF
      - 配置BGP，建立VPNv4邻居
      - 配置MPLS
  2. CE与PE对接
      - 建立BGP邻居
      - PE上创建实例，并在接口绑定
  3. PE与PE之间建立IPV4邻居，传递IPv4路由
- 解析
  - 控制平面
    - AS200
      - AR10将2.0的路由通过BGP邻居传递给AR8，AR8在BGP的VPN时立下学习到该路由，转换成为VPNv4路由添加RD、RT值、内层标签，通过VPNv4邻居传递给反射器，反射器将路由反射给AR6客户机。
      - AR6通过匹配RT值，匹配成功接收VPNv4路由，只去掉RD，保留内层标签，转换为实例路由，在通过ipv4邻居传递给AR4
    - AS100
      - AR4从邻居收到ipv4路由，路由绑定为实例1，AR4将该实例路由转换为VPNv4路由，加上RT、RD、内层标签，通过反射器反射给AR2
      - AR2收到该VPNv4路由后去掉RD值，根据RT值，将路由注入到对应的实例，保留内层标签，并将该路由通过BGP邻居传递给AR1
  - 转发平面
    - AS100
      - 在AR1查看路由表去往2.0的BGP路由，下一跳是AR2后基于目标地址192.168.2.0查询实例下的路由，发现去往2.0的地址是BGP邻居10.10.4.4，并压入内层标签1028
        ![](/images/MPLS跨域VPNoptionA转发平面1.png)
      通过迭代查询目标地址10.10.4.4的FIB表，tunnel口不为0
        ![](/images/MPLS跨域VPNoptionA转发平面2.png)
      查询标签转发表压入外层标签1025
        ![](/images/MPLS跨域VPNoptionA转发平面3.png)
      经过传输设备，将出标签替换为3.弹出外层标签，转发给AR4，根据内层标签进入实例1
        ![](/images/MPLS跨域VPNoptionA转发平面4.png)
      进入实例1下基于2.0查找路由表，下一跳是EBGP邻居的接口地址
        ![](/images/MPLS跨域VPNoptionA转发平面5.png)
    - AS200
      - AR6收到后在实例下继续查找路由表，封装外层标签1028，下一跳为10.10.8.8，根据MPLS外层标签转发到AR8，AR8根据内层标签找到对应的实例，在实例下查找路由表，转发给ipv4邻居的下一跳（过程与AS100内转发相同）
- 命令
```
接口配置及AS内的IGP配置为预配;MPLS作为预配
AR4的G0/0/1与AR6的G0/0/2接口不运行MPLS

AR2配置：
mpls lsr-id 10.10.2.2
mpls
#
mpls ldp
#
ip vpn-instance 1
 ipv4-family
  route-distinguisher 1:1
  vpn-target 2:2 export-extcommunity
  vpn-target 2:2 import-extcommunity
 #
 bgp 100
 peer 10.10.5.5 as-number 100 
 peer 10.10.5.5 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  peer 10.10.5.5 enable
 # 
 ipv4-family vpnv4
  policy vpn-target
  peer 10.10.5.5 enable
 #
 ipv4-family vpn-instance 1 
  peer 10.1.12.1 as-number 400
 #
 interface GigabitEthernet0/0/0
 ip binding vpn-instance 1
 ip address 10.1.12.2 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 10.1.23.2 255.255.255.0 
 mpls
 mpls ldp
AR5配置：
bgp 100
 peer 10.10.2.2 as-number 100 
 peer 10.10.2.2 connect-interface LoopBack0
 peer 10.10.4.4 as-number 100 
 peer 10.10.4.4 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  peer 10.10.2.2 enable
  peer 10.10.4.4 enable
 # 
 ipv4-family vpnv4
  undo policy vpn-target
  peer 10.10.2.2 enable
  peer 10.10.2.2 reflect-client
  peer 10.10.4.4 enable
  peer 10.10.4.4 reflect-client
AR4配置：
#
ip vpn-instance 1
 ipv4-family
  route-distinguisher 1:1
  vpn-target 2:2 export-extcommunity
  vpn-target 2:2 import-extcommunity
#
mpls lsr-id 10.10.4.4
mpls
#
mpls ldp
#
bgp 100
 peer 10.10.5.5 as-number 100 
 peer 10.10.5.5 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  peer 10.10.5.5 enable
 # 
 ipv4-family vpnv4
  undo policy vpn-target
  peer 10.10.5.5 enable
 #
 ipv4-family vpn-instance 1 
  peer 10.1.46.6 as-number 200 
 #
 interface GigabitEthernet0/0/1
 ip binding vpn-instance 1
 ip address 10.1.46.4 255.255.255.0
 
 AS200内路由器与AS100相同，配置略
```
### 不足
- ASBR实例多，对设备性能影响大
  - ASBR既要传递又要转发数据，路由多，流量大，一对公司，四个实例（4*n），n表示几对公司
- 为什么划分多实例
  - 区分不同公司的路由，把不同公司的路由，传递到对应的公司去

## option B
### 案例
#### 拓扑
![](/images/MPLS跨域VPN.png)
#### 需求
PC1和PC2跨AS实现互访
#### 步骤
1. 分别配置单域AS100、AS200
    - 配置OSPF
    - 配置BGP、建立VPNv4邻居
    - 配置MPLS
2. CE与PE对接
    - 建立BGP邻居
    - PE上创建按实例，并在接口绑定
3. PE与PE之间建立VPNv4邻居，传递VPNv4路由
#### 解析
##### 控制平面（与optionA的区别）
- AS100
  - ASBR没有实力，关闭RT检查，与RR建立VPNv4邻居关系，RR关闭RT检查接收VPNv4路由
    ![](/images/MPLSVPNoptionBAS100.png)
  ASBR之间建立VPNv4 EBGP邻居关系，通过VPNv4邻居传递VPNv4路由给AS200
- AS200
  - AR6接收该VPNv4路由
  ![](/images/MPLSVPNoptionBAS100.png)
  AR6将VPNv4路由传递给反射器AR9，传递VPNv4路由时下一跳会自动改变
  ![](/images/MPLSVPNoptionBAS200下一跳自动改变.png)
  `注：（在传递ipv4路由时下一跳不改变）`

  到达AR8匹配RT值，接收该VPNv4路由，去掉RD，根据入向RT导入对应实例
##### 转发平面
PC1访问PC2，AR2查找实例下路由192.168.2.0，去往该路由的下一跳是10.10.4.4，封装内层标签，查询目标地址10.10.4.4数据包迭代进行mpls隧道，封装外层标签，通过标签转发，AR3弹出外层标签，到达AR4露出内层标签，通过查找2.0的VPNv4路由的详细信息，可知该内层标签是AR6发给自己的，重新封装内层标签，下一跳是10.1.46.6，通过标签交换转发给AR6
  ![](/images/MPLSVPNoptionB转发平面1.png)
  AR6收到VPNv4路由后，重新封装内层标签，查找VPNv4路由表迭代进隧道
  ![](/images/MMPLSVPNoptionB转发平面2.png)
  ![](/images/MPLSVPNoptionB转发平面3.png)
  封装外层标签
  ![](/images/MPLSVPNoptionB转发平面4.png)
  通过外层标签转发AR7，AR7剥离外层标签给AR8，AR8通过内层标签进入对应的实例，在实例下查找路由表转发给AR10
  ##### VPNv4路由标签为什么变化
  - ASBR1与ASBR2建立VPNv4邻居，从EBGP邻居收到的VPNv4路由传递给IBGP邻居时下一跳会发生变化，内层标签也会发生变化，因为标签只有本地有效
  - 如果不变会导致的问题：
    - 如果标签相同，将无法通过标签区分是去往哪个site
  ![](/images/MMPLSVPNoptionBVPNv4路由为什么变化.png)
#### 命令
```
AR2配置：
ip vpn-instance 1
 ipv4-family
  route-distinguisher 1:1
  vpn-target 2:2 export-extcommunity
  vpn-target 2:2 import-extcommunity
#
mpls lsr-id 10.10.2.2
mpls
#
mpls ldp
#
interface GigabitEthernet0/0/0
 ip binding vpn-instance 1
 ip address 10.1.12.2 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 10.1.23.2 255.255.255.0 
 mpls
 mpls ldp
#
interface LoopBack0
 ip address 10.10.2.2 255.255.255.255 
#
bgp 100
 peer 10.10.5.5 as-number 100 
 peer 10.10.5.5 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  peer 10.10.5.5 enable
 # 
 ipv4-family vpnv4
  policy vpn-target
  peer 10.10.5.5 enable
 #
 ipv4-family vpn-instance 1 
  peer 10.1.12.1 as-number 400 
#
ospf 1 router-id 2.2.2.2 
 area 0.0.0.0 
  network 10.1.23.2 0.0.0.0 
  network 10.10.2.2 0.0.0.0 
AR5配置：
#
mpls lsr-id 10.10.5.5
mpls
#
mpls ldp
#
interface GigabitEthernet0/0/0
 ip address 10.1.35.5 255.255.255.0 
 mpls
 mpls ldp
#
interface LoopBack0
 ip address 10.10.5.5 255.255.255.255 
#
bgp 100
 peer 10.10.2.2 as-number 100 
 peer 10.10.2.2 connect-interface LoopBack0
 peer 10.10.4.4 as-number 100 
 peer 10.10.4.4 connect-interface LoopBack0
 # 
 ipv4-family vpnv4
  undo policy vpn-target
  peer 10.10.2.2 enable
  peer 10.10.2.2 reflect-client
  peer 10.10.4.4 enable
  peer 10.10.4.4 reflect-client
#
ospf 1 router-id 5.5.5.5 
 area 0.0.0.0 
  network 10.1.35.5 0.0.0.0 
  network 10.10.5.5 0.0.0.0
AR4配置：
mpls lsr-id 10.10.4.4
mpls
#
mpls ldp
#
interface GigabitEthernet0/0/0
 ip address 10.1.34.4 255.255.255.0 
 mpls
 mpls ldp
#
interface GigabitEthernet0/0/1
 ip address 10.1.46.4 255.255.255.0 
 mpls
#
interface LoopBack0
 ip address 10.10.4.4 255.255.255.255 
#
bgp 100
 peer 10.1.46.6 as-number 200 
 peer 10.10.5.5 as-number 100 
 peer 10.10.5.5 connect-interface LoopBack0
#
 ipv4-family vpnv4
  undo policy vpn-target
  peer 10.1.46.6 enable
  peer 10.10.5.5 enable
#
ospf 1 router-id 4.4.4.4 
 area 0.0.0.0 
  network 10.1.34.4 0.0.0.0 
  network 10.10.4.4 0.0.0.0
AR6配置
#
mpls lsr-id 10.10.6.6
mpls
#
mpls ldp
#
interface GigabitEthernet0/0/1
 ip address 10.1.67.6 255.255.255.0 
 mpls
 mpls ldp
#
interface GigabitEthernet0/0/2
 ip address 10.1.46.6 255.255.255.0 
 mpls
#
interface LoopBack0
 ip address 10.10.6.6 255.255.255.255 
#
bgp 200
 peer 10.1.46.4 as-number 100 
 peer 10.10.9.9 as-number 200 
 peer 10.10.9.9 connect-interface LoopBack0
 #
 ipv4-family vpnv4
  undo policy vpn-target
  peer 10.1.46.4 enable
  peer 10.10.9.9 enable
#
ospf 1 router-id 6.6.6.6 
 area 0.0.0.0 
  network 10.1.67.6 0.0.0.0 
  network 10.10.6.6 0.0.0.0 
AR8配置：
#
ip vpn-instance 1
 ipv4-family
  route-distinguisher 1:2
  vpn-target 2:2 export-extcommunity
  vpn-target 2:2 import-extcommunity
#
mpls lsr-id 10.10.8.8
mpls
#
mpls ldp
#
interface GigabitEthernet0/0/1
 ip binding vpn-instance 1
 ip address 10.1.18.8 255.255.255.0 
#
interface GigabitEthernet0/0/2
 ip address 10.1.78.8 255.255.255.0 
 mpls
 mpls ldp
#
interface LoopBack0
 ip address 10.10.8.8 255.255.255.255 
#
bgp 200
 peer 10.10.9.9 as-number 200 
 peer 10.10.9.9 connect-interface LoopBack0
# 
 ipv4-family vpnv4
  policy vpn-target
  peer 10.10.9.9 enable
#
 ipv4-family vpn-instance 1 
  peer 10.1.18.10 as-number 500 
#
ospf 1 router-id 8.8.8.8 
 area 0.0.0.0 
  network 10.1.78.8 0.0.0.0 
  network 10.10.8.8 0.0.0.0 

```
### 特点与不足
- 特点
  - 配置简单，ASBR不需要创建实例
- 不足
  - ASBR即传递路由，有转发数据，依旧对ASBR性能占用过大
## option C
### option C方案1
#### 拓扑
![](/images/MPLSVPNoptionC拓扑.png)
#### 需求
PC1和PC2跨AS实现互访
#### 步骤
1. 配置单个AS内IGP，配置MPLS
2. 建立BGP的ipv4邻居，传递ipv4路由，给监理邻居的VPNv4的环回口地址分配标签，使用路由策略让MP-BGP为BGP路由分配标签
#### 解析
- 分析
  - optionC狮子啊PE设备之间建立VPNv4邻居，需要解决的问题就是如何让建立邻居的环回口地址互通。需要将PE的ipv4路由和标签传递到对端PE，路由传递容易关键是解决标签如何传递，为什么要传递标签呢？`因为在数据转发过程中，可能会出现路由黑洞，所以要通过标签转发，标签必须是连续的。`ASBR之间是不同AS，也没有IGP，LDP分配标签要保证IGP可达，LDP不能为该路由分配标签，因此可以通过MP-BGP为ipv4路由分配标签
- 控制平面
  - 传递路由
    - 在两个ASBR之间建立BGP的ipv4邻居，并分别与自己AS内的PE设备建立ipv4的邻居关系，并在ASBR上宣告建立vpnv4邻居的环回口地址
    - ASBR与PE之间跨设备建立ipv4的BGP邻居，传递BGP路由
  - 传递标签
    - 此时虽然学习到路由，但是没有标签，无法互通，因此需要在AR4通告给AR6邻居时打上BGP标签，在AR6上匹配到AR4分配的标签后，再打上BGP标签通告给AR8，同理在AR6去往AR4的邻居也需要打标签，ASBR需要支持标签通告能力，此时互相学习到了对方的路由，地址可以通告标签实现互通
    - AR2与AR8之间建立VPNv4的邻居，传递VPNv4路由
- 转发平面
  - 在AR2上查找VPNv4路由，下一跳是10.10.8.8，封装内层标签1026
    ![](/images/MPLSVPNoptionC1转发平面1.png)
  - 去往10.10.8.8下一跳是10.10.4.4，封装中层标签1027
    ![](/images/MPLSVPNoptionC1转发平面2.png)
  - 去往10.10.4.4封装外层标签1025，下一跳是AR3，发给AR3，查找标签转发表转发，也同时剥离外层标签
    ![](/images/MMPLSVPNoptionC1转发平面3.png)
  - 到达ASBR后查找BGP分配的标签，发给AR6，AR6查找LDP分配的标签去往10.10.8.8，到AR7，AR7剥离外层标签，剩下内层标签
    ![](/images/MPLSVPNoptionC1转发平面4.png)
  - AR8收到该数据包根据内层标签转发到相应的实例中，根据查找实例下路由表，转发给AR10，实现通讯

  `注：BGP默认可以给VPNv4分标签，BGP默认不会给IPv4路由分标签`
  - ASBR在从EBGP邻居学来的路由通告给IBGP时，下一跳不变，但在这个场景下，当打上标签后，下一跳会改变
#### 命令
```
AR2配置：
bgp 100
 peer 10.10.4.4 as-number 100 
 peer 10.10.4.4 connect-interface LoopBack0
 peer 10.10.8.8 as-number 200 
 peer 10.10.8.8 ebgp-max-hop 5 //解析如下
 peer 10.10.8.8 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  peer 10.10.4.4 enable
  peer 10.10.4.4 label-route-capability
  peer 10.10.8.8 enable
 # 
 ipv4-family vpnv4
  policy vpn-target
  peer 10.10.8.8 enable
 #
 ipv4-family vpn-instance a 
  peer 10.1.12.1 as-number 400 
AR4配置：
bgp 100
 peer 10.1.46.6 as-number 200 
 peer 10.10.2.2 as-number 100 
 peer 10.10.2.2 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  network 10.10.2.2 255.255.255.255 
  peer 10.1.46.6 enable
  peer 10.1.46.6 route-policy b export
  peer 10.1.46.6 label-route-capability
  peer 10.10.2.2 enable
  peer 10.10.2.2 route-policy a export
  peer 10.10.2.2 next-hop-local 
  peer 10.10.2.2 label-route-capability
  #
route-policy a permit node 10 
 if-match mpls-label 
 apply mpls-label
#
route-policy b permit node 10 
 apply mpls-label
AR6配置：
bgp 200
 peer 10.1.46.4 as-number 100 
 peer 10.10.8.8 as-number 200 
 peer 10.10.8.8 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  network 10.10.8.8 255.255.255.255 
  peer 10.1.46.4 enable
  peer 10.1.46.4 route-policy a export
  peer 10.1.46.4 label-route-capability
  peer 10.10.8.8 enable
  peer 10.10.8.8 route-policy b export
  peer 10.10.8.8 next-hop-local 
  peer 10.10.8.8 label-route-capability
#
route-policy a permit node 10 
 apply mpls-label
#
route-policy b permit node 10 
 if-match mpls-label 
 apply mpls-label
AR8配置
bgp 200
 peer 10.10.2.2 as-number 100 
 peer 10.10.2.2 ebgp-max-hop 5 
 peer 10.10.2.2 connect-interface LoopBack0
 peer 10.10.6.6 as-number 200 
 peer 10.10.6.6 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  peer 10.10.2.2 enable
  peer 10.10.6.6 enable
  peer 10.10.6.6 label-route-capability
 # 
 ipv4-family vpnv4
  policy vpn-target
  peer 10.10.2.2 enable
 #
 ipv4-family vpn-instance a 
  peer 10.1.18.10 as-number 500 
#
```
### option C方案2
#### 拓扑
![](/images/MPLSVPNoptionC拓扑.png)
#### 需求
PC1和PC2跨AS实现互访
#### 步骤
1. 配置单个AS内IGP，配置MPLS
2. 在ASBR之间建立BGP的ipv4邻居，传递ipv4路由，将BGP路由引入IGP，让LDP为引入的BGP路由分配标签
#### 解析
方案2：较方案1的区别在与PE建立VPNv4邻居的环回口地址是如何传递的，方案2中是通过在ASBR上引入BGP实现的IGP-BGP-IGP，BGP间的标签仍然通过MP-BGP分配，IGP分配的标签通过LDP为BGP路由分配
- 控制平面
  1. 传递路由：AR6宣告10.10.8.8，通过BGP邻居传递给AR4，AR4上BGP引入OSPF，AR2通过IGP学习到AR8的环回口路由
  2. 传递标签：AR4上手动让LDP给引入的BGP路由分配标签，默认不分配
- 转发平面
  - 基于192.168.2.0查找实例路由表，下一跳为10.10.8.8
    ![](/images/MPLSVPNoptionC2转发平面1.png)
  - 去往下一跳为标签转发，封装的内层标签为AR8分配，封装外层标签AR3通过LDP分配的，通过外层标签转发给AR3
    ![](/images/MPLSVPNoptionC2转发平面2.png)
  - AR3经过外层标签转发给AR4，该标签是LDP分配的，到达ASBR依旧依据标签转发，该标签是BGP分配的
    ![](/images/MPLSVPNoptionC2转发平面3.png)
  - 到达AR6查找标签转发，该标签是LDP分配的，一次转发给AR8，AR8通过内层标签注入到对应的实例下，实现实例路由表的转发
#### 重点配置
```
AR8配置：
bgp 200
 peer 10.10.2.2 as-number 100 
 peer 10.10.2.2 ebgp-max-hop 5 
 peer 10.10.2.2 connect-interface LoopBack0
 peer 10.10.6.6 as-number 200 
 peer 10.10.6.6 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  peer 10.10.2.2 enable
  peer 10.10.6.6 enable
  peer 10.10.6.6 label-route-capability
 # 
 ipv4-family vpnv4
  policy vpn-target
  peer 10.10.2.2 enable
 #
 ipv4-family vpn-instance a 
  peer 10.1.18.10 as-number 500 
AR6配置：
route-policy a permit node 10 
 apply mpls-label
#
bgp 200
 peer 10.1.46.4 as-number 100 
#
 ipv4-family unicast
  undo synchronization
  network 10.10.8.8 255.255.255.255 
  peer 10.1.46.4 enable
  peer 10.1.46.4 route-policy a export
  peer 10.1.46.4 label-route-capability
#
ospf 1 router-id 6.6.6.6 
 import-route bgp
 area 0.0.0.0 
  network 10.1.67.6 0.0.0.0 
  network 10.10.6.6 0.0.0.0 
#
mpls lsr-id 10.10.6.6
mpls
 lsp-trigger bgp-label-route
AR4配置：
route-policy b permit node 10 
 apply mpls-label
#
bgp 100
 peer 10.1.46.6 as-number 200 
#
 ipv4-family unicast
  undo synchronization
  network 10.10.2.2 255.255.255.255 
  peer 10.1.46.6 enable
  peer 10.1.46.6 route-policy b export
  peer 10.1.46.6 label-route-capability
#
ospf 1 router-id 4.4.4.4 
 import-route bgp
 area 0.0.0.0 
  network 10.1.34.4 0.0.0.0 
  network 10.10.4.4 0.0.0.0 
#
mpls lsr-id 10.10.4.4
mpls
 lsp-trigger bgp-label-route
#
mpls ldp
AR2配置：
bgp 100
 peer 10.10.4.4 as-number 100 
 peer 10.10.4.4 connect-interface LoopBack0
 peer 10.10.8.8 as-number 200 
 peer 10.10.8.8 ebgp-max-hop 5 
 peer 10.10.8.8 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  peer 10.10.4.4 enable
  peer 10.10.4.4 label-route-capability
  peer 10.10.8.8 enable
 # 
 ipv4-family vpnv4
  policy vpn-target
  peer 10.10.8.8 enable
 #
 ipv4-family vpn-instance a 
  peer 10.1.12.1 as-number 400 
```
#### 不足
如果AS内设备过多，每一他爱设备都要分配标签，作用的范围比较大，占用设备性能