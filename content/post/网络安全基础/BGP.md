---
author: Hugo Authors
title: BGP
date: 2023-12-09
description:  网络基础知识
series: 
  - 网络安全基础

---

BGP详解

<!--more-->
# AS：自治系统
  - 每个自治系统都有唯一的一个编号，即AS号
  - 2字节AS号范围是0-65535
    - 0被保留使用
    - 64512-65534为私有AS
  - 4字节AS号范围是0-4294967295
    - 0被保留使用
    - 4200000000-4292967294为私有AS
    - 2字节的私有AS在4字节中依然是私有AS
# 概述
  - 边界网关协议
  - 作用：实现在AS与AS之间动态交换路由信息
  - 稳定：BGP是基于TCP协议建立的，使用端口号TCP179，非常稳定
  - 特点：
    - 稳定性高
    - 可以传递大量路由，支持大规模网络
    - 丰富的路由控制策略，可以实现灵活的选路
  - 特征：
    - 路由器之间的BGP绘画基于TCP连接而建立
    - 运行BGP的路由器被称为BGP路由器
    - 两个建立BGP会话的路由器互为对等体（也就是邻居），BGP对等体之间交换BGP路由表
    - BGP路由器只发送增量的BGP路由更新，或进行触发式更新（不会周期性更新）
    - BGP能够传递大批量的路由，可在大规模网络中应用
  - 易错：
    - IGP：内部网关路由协议（OSPF，ISIS都属于IGP协议）
    - EGP：外部网关路由协议（最开始用的）
    - BGP：边界网关协议
    - IBGP：AS号相同的BGP邻居
    - EBGP：AS号不相同的BGP邻居

### BGP通告原则：传递路由的方法
- BGP路由器只会把自己最优的路由传递给自己的邻居状态代码为`>`的路由传递给自己的邻居

## BGP五种报文
- open报文：用于建立BGP邻居
  - 两方发送OPEN报文，就会实现邻居的建立，当邻居建立成功，open报文就不会再次发送了
  - 类似于hello报文

- Keepalive报文：用于维护邻居关系，保持BGP链接 
  - 每隔一分钟给对方发送一份报文，以保持连接的有效性，维护邻居关系，保持连接
  - 三倍时间我还没收到报文，那么就会断开邻居关系

- `Update报文：用于在邻居之间传递路由信息`
  - 类似于LSU报文
  - 一条update消息可以发布多条属性相同的可达路由信息，也可以撤销多条不可达路由信息
- Notification报文：
  - 通知报文，用于通知BGP邻居之间的报错信息
  - 用于中断BGP链接
- Router-refresh报文：
  - 策略更改后，让邻居重新发送路由信息

## BGP工作过程
  - 建立路由表
  - 同步路由表
## BGP状态机

- Idle：初始化状态
  - tcp未连接状态，邻居的状态为idle，当BGP设备和邻居进行tcp连接时，会从当前的状态转为connect状态
- connext：tcp连接状态 
  - 如果TCP建立连接成功，bgp设备会向邻居发送open报文，状态也会转为opensent状态
  - 如果TCP链接建立失败则进入Active状态，反复尝试连接
- active：活跃状态
  - 在当前状态下，bgp设备依然没有放弃希望，依然试图继续建立tcp连接，如果成功状态依然可以转为opensent状态
  - 如果依然失败，bgp则会停留在active状态
  - 备注：如果重传计时器超市，依然没有收到邻居回应，bgp设备会心灰意冷的返回connext状态
- opensent：open报文已发送状态，已向邻居发送ospf报文
  - 如果bgp设备给邻居发送open报文后，如果收到来自邻居表的回复，并且回复的报文是正确的，BGP设备会继续发送keepalive报文，状态转为open comfirm状态
  - 如果bgp设备给邻居发送open报文后，如果收到来自邻居的回复，但是回复的报文有错误，bgp设备会向邻居发送notification报文（用于断开bgp连接状态），段考后会跳转到最初的idle状态
- open comfirm：open报文确认状态
  - 如果收到邻居发来的keep alive报文，状态会转为established建立状态
  - 如果收到邻居发来的notification报文，状态会跳转为idle状态
- established：连接已建立状态
  - 如果收到邻居发来的正确的update报文或者keepalive报文，bgp就会认为邻居是正常的
  - 如果收到邻居发来的错误的udate报文或者keepalive报文，bgp设备会发送notification报文，通知邻居，我们分手吧，不做邻居了，会回到idle初始化状态

## BGP路由默认优先级为255
## BGP不能在同一机器中运行多个AS号
# IBGP
IBGP对等体之间不一定需要物理上的链接，只要TCP连接可以建立即可
# EBGP
外部邻居：用于AS之间
## EBGP基本配置
```
[R1]bgp 100————开启BGP功能，配置as100
[R1-bgp]router-id 1.1.1.1————配置router-id
[R1-bgp]peer 192.168.12.2 as 200————指定邻居设备的IP地址和邻居设备所在的AS
[R1-bgp]network 192.168.1.0 24————注入路由，把已经存在的路由条目放进BGP协议

[R2]bgp 200
[R2-bgp]router-id 2.2.2.2
[R2-bgp]peer 192.168.12.1 as 100
[R2-bgp]network 192.168.2.0 24
```
## BGP邻居表检查解析：
```
[R1]dis bgp peer

 BGP local router ID : 1.1.1.1
 Local AS number : 100
 Total number of peers : 1		  Peers in established state : 1

  Peer            V          AS  MsgRcvd  MsgSent  OutQ  Up/Down       State       PrefRcv

  192.168.12.2    4         200        2        4     0 00:00:19     Established      0
```
- BGP local router ID : 1.1.1.1   ：表示的是BGP协议给当前这台设备分配的名字是1.1.1.1
- Local AS number : 100           ：表示的是当前这台设备所属的AS号
- otal number of peers : 1        ：表示的是当前设备全部的邻居数量
- Peers in established state : 1  ：
  - 表示的是当前设备有效的邻居数量（状态为建立的邻居数量）
  - established是BGP的一种状态，最完美的民居状态，类似于ospf中的full
- peer          ：表示的是对端邻居的IP地址
- v             ：表示的是BGP版本bgp4+
- AS            ：表示的是邻居设备所在的AS
- msgrcvd       ：表示的是从邻居那里收到的消息数目
- msgsent       ：表示的是向邻居设备发送的消息数目
- outQ          ：表示的是出向队列
  - 等待发往指定邻居的消息————出向数据————排队等待出去的数据
  - bgp要求稳定，如果邻居设备无法向对端发送一个稳定的报文，那么这个报文就会被要求重新传输
  - 那些需要重传的报文，就会放到这个出项队列中，所以正常情况下，这个参数的值是0
  - 如果这个参数不是0，说明和邻居之间的链路非常不稳定，容易出现网络拥塞和丢包。
- up/down       ：
  - 如果状态是Established那么这个时间表示的是邻居建立了多长时间，就是up的时间
  - 如果状态不是Established那么这个时间表示的是这个邻居断开了多长时间，就是down的时间
- state         ：表示的是邻居的状态，最完美的状态是Established
- prefrcv       ：
  - 表示的是邻居传递给我的路由条目数量
  - 如果这个数值为0，就表示邻居没有给我传递路由
  - 如果这个数值为1，就表示邻居给我传递了一个路由

## 显示BGP协议路由表
```
[R1]display bgp routing-table 

 BGP Local router ID is 1.1.1.1 
 Status codes: * - valid, > - best, d - damped,
               h - history,  i - internal, s - suppressed, S - Stale
               Origin : i - IGP, e - EGP, ? - incomplete


 Total Number of Routes: 2
      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn

 *>   192.168.1.0        0.0.0.0         0                     0      i
 *>   192.168.2.0        192.168.12.2    0                     0      200i
```
解析：
  - Status codes:   状态代码
  - \* - valid  ：  有效的路由，
  - \> - best   ：  最好的路由，表示这条路由是最优的线路，最好的
  - i  -internal：  内部的路由
  - origin：起源代码
    - i-BGP：表示的这条路由是通过network方式注入的
    - ? - incomplete：表示这条路由是通过import-router方式注入的

### EBGP邻居关系
- EBGP邻居在传递路由的时候，会自动的修改下一跳地址
- 会将下一跳地址修改为和对端设备建立邻居的那个接口IP地址


## BGP通告原则
- BGP路由器仅仅将自己最优的路由发布给自己最好的邻居（即下一跳可达）
- 从EBGP邻居获取的路由，会发布给所有邻居
- 从IBGP邻居获取的路由，不会再发送给其他IBGP邻居（内内不相传）
- IGP和BGP同步：该条原则被称为BGP同步原则
- 解决方案：
  - 引入到OSPF网络中
  - 使用静态路由互联·
  - IBGP全互联：修改下一跳地址
    - 优点：简单
    - 缺点：后期维护难，每个设备都需要建立大量的TCP连接，消耗大量系统资源
  - RR，路由反射器
    - 角色：反射器和反射器客户端
    - 优点：简单
    - 反射规则：非非不反射，

## 路由反射器反射规则（RR：反射器）
- RR从非客户机收到的路由会反射给客户机，会发布给EBGP邻居
- RR从非客户机收到的路由不会反射给非客户机（非非不反射）
- RR从客户机收到的路由会反射给客户机和非客户机
- RR从EBGP邻居收到的路由，会反射给所有的客户机、非客户机

## 利用loopback接口做BGP
- lookback：回环口
  - 稳定的不会物理上down的接口
  - 此接口配置成功后，和物理接口一样，在设备上也会产生一条直连路由
  - lookback口的IP地址配置好了之后，则代表这个设备上，只有这一个地址。接口的广播域上没有其他地址
```

```
## BGP的宣告方式：network、import
- BGP中network宣告的路由中，在存在与IGP的路由表当中，如果IGP路由表不存在这条路有，那么BGP也无法进行宣告。如果IGP路由表（display IP routing-table）存在这条路有，那么BGP就可以进行宣告
## 路经属性分类
- 公认必遵

  必须包含在每个Update消息里
  - Origin
  - AS_Path
  - Nrxt_hop
- 公认任意

  可以包含在某些Update消息里
  - Local_Preference
  - Atomic_aggregate
- 可选过渡

  BGP设备不识别此类属性依然会接受该类属性并通告给其他对等体
  - Aggregator
  - Community
- 可选非过渡

  BGP设备不设别此类属性会忽略该属性，且不会通告给其他对等体
  - MED
  - Cluster-List
  - Originator-ID