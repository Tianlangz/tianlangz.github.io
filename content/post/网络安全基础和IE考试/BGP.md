---
author: Hugo Authors
title: BGP
date: 2023-12-09
description:  网络安全基础和IE考试
series: 
  - 网络安全基础和IE考试

---

BGP详解

<!--more-->
# 协议概述
## 路由分类
### 算法
#### IGP
- OSPF
  - 链路状态协议（SPF最短路径优先协议）
- ISIS
  - 链路状态协议（SPF最短路径优先协议）
#### BGP
距离适量协议（路径矢量协议）
### 位置
#### 企业内部
- IGP
  - OSPF
  - ISIS（数据中心使用的IGP）
#### 企业之间
- EGP
  - BGP
  - EGP
### 协议优先级
#### 华为
- 直连
  - 0
  - 开销默认为0
- OSPF
  - 内部10
  - 外部引入150
  - 开销默认为1
- ISIS
  - 15
  - 开销默认10
- 静态
  - 60
  - 开销值为0
- BGP
  - 255
  - MED为0
#### 思科
- 直连
  - 0
- OSPF
  - 110
- ISIS
  - 115
- 静态
  - 1
- BGP
  - 20
## BGP不能在同一机器中运行多个AS号
## 概念
- 边界网关协议(距离/路径矢量协议)DV
- 作用：实现在AS与AS之间动态交换路由信息
- 稳定：BGP是基于TCP协议建立的，使用端口号TCP179，非常稳定
- 特点：
  - 稳定性高
  - 可以传递大量路由，支持大规模网络
  - 丰富的路由控制策略，可以实现灵活的选路
- 特征：
  - 路由器之间的BGP会话基于TCP连接而建立
  - 运行BGP的路由器被称为BGP路由器
  - 两个建立BGP会话的路由器互为对等体（也就是邻居），BGP对等体之间交换BGP路由表
  - BGP路由器只发送增量的BGP路由更新，或进行触发式更新（不会周期性更新）
  - BGP能够传递大批量的路由，可在大规模网络中应用
  - 只要TCP连接可以建立即可建立邻居，因为TCP可以跨设备，跨网段去建立连接
## 场景
- 省级网到数据中心互联
- 数据中心内部，数据中心之间
- 运营商
## AS：自治系统
- 每个自治系统都有唯一的一个编号，即AS号
- 2字节AS号范围是0-65535
  - 0被保留使用
  - 64512-65534为私有AS
- 4字节AS号范围是0-4294967295
  - 0被保留使用
  - 4200000000-4292967294为私有AS
  - 2字节的私有AS在4字节中依然是私有AS
## 邻居关系
### IBGP邻居
- 公司内部，推荐使用环回口建立IBGP的邻居关系
- 环回口稳定
### EBGP邻居
- 公司之间，推荐使用物理口建立EBGP的邻居关系
- 默认情况下EBGP不支持用环回口建立邻居
  - 因为有TTL检测机制，TTL=1的检测，邻居无法建立，TTL值不为1
  ```peer x.x.x.x ebgp-max-hop 2```
## 路由注入
### network
BGP中network宣告的路由中，在存在与IGP的路由表当中，如果IGP路由表不存在这条路有，那么BGP也无法进行宣告。如果IGP路由表（display IP routing-table）存在这条路有，那么BGP就可以进行宣告
- 精细化路由注入
### import
- 粗略的注入路由
- 不可能一条条搬运所以在搬运OSPF形成的路由表的时候，就要使用`import`了
## 更新方式
### 触发式更新
- BGP
  - 只有路由发生了变化，才发送更新报文，而且只发送增量路由
### 周期性更新
- OSPF
  - 更新时间1800s
  - 老化时间3600s
- ISIS
  - 更新时间900s
  - 老化时间1200s
## 符号
- \* 有效路由
- \> 最优路由
## 通告原则
- 只向邻居通告最优路由
  - BGP路由器只会把自己最优的路由传递给自己的邻居状态代码为`>`的路由传递给自己的邻居
- 从EBGP邻居学习到的路由，会通告给所有邻居（IBGP，EBGP）
- 从IBGP邻居学习到的路由，不会通告给他的IBGP邻居
  - 水平分割原则
    - 为什么有
      - 防止AS内的路由环路
    - 产生问题是什么
      - 路由黑洞（有些路由器找不到除直连以外的邻居）
    - 解决方案
      - 全互联
      - RR（反射器）
      - 联盟
      - GRE
      - MPLS
      - BGP同步
- BGP同步原则
  - 从IBGP邻居学习到的路由，必须同步给IGP中，才能传递给EBGP邻居，默认是关闭的，实际上也是打不开的，他也不支持打开
- RR
  - 反射器
  - 场景
    - 用于IGP邻居之间
  - 角色
    - 反射器（RR）
    - 客户端（client）
    - 非客户端
  - 反射原则 
    1. 从EBGP邻居学来的路由，会传递给他的IBGP和EBGP 
    2. RR从客户端学来的路由会反射给客户端/非客户端
    3. RR从非客户端学来的路由会反射给客户端
    4. 从非客户端学来的路由不会反射给非客户端
  - 防环
    - 簇内
      - 起源者ID
    - 簇间
      - 簇列表
# 工作原理
## 报文
### open报文：用于建立BGP邻居
  - 两方发送OPEN报文，就会实现邻居的建立，当邻居建立成功，open报文就不会再次发送了
  - 类似于hello报文

### Keepalive报文：用于维护邻居关系，保持BGP链接 
  - 每隔60s给对方发送一份报文，以保持连接的有效性，维护邻居关系，保持连接
  - 三倍时间我还没收到报文，那么就会断开邻居关系

### `Update报文：用于在邻居之间传递路由信息`
  - 类似于LSU报文
  - 一条update消息可以发布多条属性相同的可达路由信息，也可以撤销多条不可达路由信息
  - 作用：
    - 防环
    - 选路
### Notification报文：
  - 通知报文，用于通知BGP邻居之间的报错信息
  - 用于中断BGP链接
  - 差错报文
    - 错误代码
    - 错误子代码
### Router-refresh路由刷新报文：
  - 因为BGP是属于触发更新机制，当路由策略发生变化时，需要更新策略
  - 自动刷新
  - 手动刷新
    - refresh bgp
      - all（针对所有邻居）
      - ibgp（针对IBGP邻居）
      - ebgp（针对EBGP邻居）
      - peer x.x.x.x（针对某个邻居）
    - 后面还可以接export/import
      - export
        - 比如如果前面的参数是all，那就立即发送一个update报文把自己所有的路由都发过去
      - import
        - 会发送一个Router-refresh路由刷新报文，然会对端会立刻返回一个update报文把自己的路由都发送过去

## BGP状态机
### Idle：初始化状态
- 等32s，用于配置邻居设备
- tcp未连接状态，邻居的状态为idle，当BGP设备和邻居进行tcp连接时，会从当前的状态转为connect状态
- 停留在此状态
  - 没有IGP路由的情况下
  - 任何状态周到报错信息，都会退回到idle
### connext：tcp连接状态 
- 发送tcp报文，建立tcp连接，发送open报文
- 如果TCP建立连接成功，bgp设备会向邻居发送open报文，状态也会转为opensent状态
- 如果TCP链接建立失败则进入Active状态，反复尝试连接
- 卡在此状态的原因
  - 没有配置BGP进程
  - 认证类型不一致
  - 建立TCP邻居的目标端口179被过滤了
### active：活跃状态
- 在当前状态下，bgp设备依然没有放弃希望，状态坚持32s计时器，依然试图继续建立tcp连接，如果成功状态依然可以转为opensent状态
- 如果依然失败，bgp则会停留在active状态
- 备注：如果重传计时器超时，依然没有收到邻居回应，bgp设备会心灰意冷的返回connext状态
- 卡在此状态的原因
  - 邻居没有指定peer
  - 建立邻居的两台设备没有指定更新源，更新源检测机制（只要有一侧有更新源检测就行，就能建立起TCP连接）
### opensent：open报文已发送状态，已向邻居发送ospf报文
- 如果bgp设备给邻居发送open报文后，如果收到来自邻居表的回复，并且回复的报文是正确的，BGP设备会继续发送keepalive报文，状态转为open comfirm状态
- 如果bgp设备给邻居发送open报文后，如果收到来自邻居的回复，但是回复的报文有错误，bgp设备会向邻居发送notification报文（用于断开bgp连接状态），段考后会跳转到最初的idle状态
- 容易卡在此状态的原因：
  - router id不能相同
  - AS号配置错误（寻找对端时填写AS号填写错误）

### open comfirm：open报文确认状态
- 如果收到邻居发来的keepalive报文，状态会转为established建立状态
- 如果收到邻居发来的notification报文，状态会跳转为idle状态
- 容易卡在此状态的原因
  - EBGP建立邻居使用的环回口
### established：连接已建立状态
- 如果收到邻居发来的正确的update报文或者keepalive报文，bgp就会认为邻居是正常的
- 如果收到邻居发来的错误的udate报文或者keepalive报文，bgp设备会发送notification报文，通知邻居，我们分手吧，不做邻居了，会回到idle初始化状态
## 路经属性分类
### 公认必遵
所有设备必须识别该属性，必须包含在每个Update消息里
- Origin
  - i
    - network来的
  - e
    - EBGP引入来的
    - 华为设备没有e
  - ？
    - 通过引入进来的
- AS_Path
  - 防环
  - 选路
  - 分类
    - 有序
    - 无序
    - 联盟有序
    - 联盟无序
- Next_hop
  - 从EBGP邻居学来的路由，在传递给IBGP时，下一跳不变
    - 防止次优路径
### 公认任意（原本）
所有设备必须支持该属性，可以包含在某些Update消息里
- Local_Preference
- Atomic_aggregate
### 可选过渡（聚团）
BGP设备可以不识别此类属性依然会接受该类属性并通告给其他对等体
- Aggregator
- Community
### 可选非过渡
BGP设备不识别此类属性会忽略该属性，且不会通告给其他对等体
- MED
- Cluster-id
- Originator-ID
## 选路原则（11条）
### 协议首选值
### 本地优先级
### 本地生成的路由优于邻居学来的路由
- 手动聚合路由>自动聚合路由>network>引入的路由
### as-path路径短的
### origin
  - i>e>？
### med
  - 小的
### 从EBGP邻居学来的路由优于从IBGP邻居学来的路由
### 优选IGP下一跳小的
- 实现负载分担
### 簇列表短的
### originator-id小的
- 如果没有起源者ID，那就选router id小的
### 邻居的ip address小的
- 这里指的是用peer宣告出去的那个
## 路由策略工具
route policy
### 作用
- 过滤路由
- 修改属性
### 原理
#### 格式
``` 
route policy a 名字（premit/deny）动作 node x（这个动作是对acl的premit/deny做操作）

acl 2000 ： ip source premit/deny 192.168.1.0 0.0.0.255 匹配路由

route policy x ??动作 node 10

if-match acl
```  
#### 路由匹配工具
- acl
  - 主业
    - 做访问控制，分析数据包，默认动作允许，结合traffic-filter
  - 副业
    - 匹配路由 
      - 匹配路由通配符0代表匹配，1代表不匹配
      - 匹配路由不专业，不精准，粗略的匹配路由，不能区分掩码
- ip-prefix
  - 前缀列表
  - 专门用于匹配路由，可以通过掩码精确匹配路由
  - 格式
  ```
  ip ip-prefix a（名字） deny/pre（动作）掩码范围
  ```
- as-path-filter （只有BGP才能用）
  - 通过as号匹配路由
  - 匹配正则
  - 匹配普通AS号
- community filter 
  - 匹配团体字
  - 匹配正则
- tag
  - IGP
- rd-filter
  - vpn场景下匹配rd值
- extcommunity-filter
  - VPN场景下匹配rt值

# 高级特性
## 路由匹配工具
### as-path-filter （只有BGP才能用）
- 配置
  - route policy下调用
  - as-path-filter（直接在邻居下调用）
- 概念
  - 通过as号匹配路由
  - 匹配正则
    - 想匹配相邻AS传递过来的路由
    - 想匹配始发AS是100的路由
  - 匹配普通AS号
### community filter 
- 匹配团体字
- 匹配正则
![](/images/团体属性怎样用route policy引用.png)
## ORF
- 概念：出方向路由过滤
- 作用：减少链路带宽的占用
- 原理：通过ORF技术，告诉邻居自己需要哪条路由，邻居收到后，按照需要发布路由
## 对等体
作用：简化配置
## BGP安全
### MD5认证
设置后不能发生更改，更改的话业务需要中断
### keychain认证
配置和现实机制更复杂，但好在能动态修改密码，所以安全性更高
### GTSM
- 通用的TTL检测机制
- 作用：防止现网中的设备，建立非法邻居
- 公式：[255-3+1,255]
## 发射器








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
# BGP邻居表检查解析：
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

# 显示BGP协议路由表
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

# EBGP邻居关系
- EBGP邻居在传递路由的时候，会自动的修改下一跳地址
- 会将下一跳地址修改为和对端设备建立邻居的那个接口IP地址