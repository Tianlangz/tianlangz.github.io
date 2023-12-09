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
    - 64512-65534为私有AS
  - 4字节AS号范围是0-4294967295
    - 4200000000-4292967294为私有AS
    - 2字节的私有AS在4字节中依然是私有AS
# 概述
  - 作用：实现在AS与AS之间动态交换路由信息
  - 稳定：BGP是基于TCP协议建立的，使用端口号TCP179，非常稳定
  - 特点：
    - 稳定性高
    - 传递大量路由
    - 丰富的路由控制策略
  - 特征：
    - 路由器之间的BGP绘画基于TCP连接而建立
    - 运行BGP的路由器被称为BGP路由器
    - 两个建立BGP会话的路由器互为对等体（也就是邻居），BGP对等体之间交换BGP路由表
    - BGP路由器只发送增量的BGP路由更新，或进行触发式更新（不会周期性更新）
    - BGP能够传递大批量的路由，可在大规模网络中应用
# EBGP基本配置
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
## 检查解析：
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
  - origin：起源代码
    - i-BGP：表示的这条路由是通过network方式注入的
    - ? - incomplete：表示这条路由是通过import-router方式注入的

### BGP通告原则：传递路由的方法
- BGP路由器只会把自己最优的路由传递给自己的邻居状态代码为`>`的路由传递给自己的邻居

### EBGP邻居关系
- EBGP邻居在传递路由的时候，会自动的修改下一跳地址
- 会将下一跳地址修改为和对端设备建立邻居的那个接口IP地址