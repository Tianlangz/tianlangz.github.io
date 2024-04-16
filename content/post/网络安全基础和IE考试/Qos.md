---
author: zty
title: Qos
date: 2024-04-12
description:  网络安全基础和IE考试
series: 
  - 网络安全基础和IE考试
tags : [
    网络基础,HCIE考试
]
categories : [
    ENSP,
    华为设备配置
]
series : [HCIE考试]
aliases : [数通基础]
---
（Quality of Service）服务质量，用户对不同的流量有不同的需求，如果需要保证高带宽的、不能丢包的，或延时和抖动要求较高，这时可以使用Qos，针对不同的网络业务需求提供不同的服务质量，牺牲一些东西来保证剩下的流量安全通过。
<!--more-->
# 概述
## 影响因素
### 带宽
由于网络的带宽是传输链路上最小带宽决定的，当有大量数据流同时涌入低俗的带宽将会造成网络拥塞
### 延时
延时是指一个保温从一个网络的一端传送到另一端所需要的时间。
### 抖动
抖动是由于每个报文的端到端时延不相等造成的，比如你发的123，到对面因为时延的不同可能对面收到的是213.
### 丢包
丢包率
## 定义
服务质量，对不同的用户提供不同的服务，实现服务的区分，在现网中我们看到Qos的场景一般都是设备在出口时设置的，而现网中一般都使用CBQ基于类来区分流量通过的优先顺序
## 服务模型
### 尽力而为（est-Effort Service）
#### 特点
是一个单一的服务模型，也是最简单的服务模型。在尽力而为的服务模型的网络上可通过增大网络带宽、升级网络设备等方式来提升网络通信质量
#### 优点
可以改善高带宽、低抖动和延时、丢包率等
#### 缺点
成本高，升级更换设备存在网络中断风险，不能对不同的业务提供不同的服务
### 综合模型（Integrated Service）
#### 特点
该服务模型在发送报文前，需要向网络申请特定的服务，这个请求是通过信令（signal）来完成的，应用程序首先通知网络它自己的流量参数和所需的特定服务质量的请求，包括带宽、时延等。应用程序一般在收到网络确认信息后，即认为网络已经为这个应用程序的报文发送预留了资源，然后立即发送报文

理解记忆：信令就像警察，流量到来之前了解沿途的信息状态，提供专用的通道，了解协商完路径后，在进行发送。预留公交车道
![](/images/Qos综合模型示意图.png)
#### 优点
可以在网络拥塞的情况下为某些特定业务提供带宽、延时保证。
#### 缺点
运算复杂，在网络节点耗费大量CPU和内存资源；实现较复杂；当无流量发送时，仍然占用带宽，使用率较低该方案要求端到端所有节点设备都支持并运行信令RSVP协议
### 差分服务模型
#### 特点
首先将网络中的流量分成多个类，然后为每个类打上标记（通过外部映射对应内部映射，内部映射到队列），将数据报文通过流量监管或整形，或拥塞管理或拥塞避免机制进行转发或丢弃
![](/images/Qos差分服务模型示意图.png)
#### 优点
不需要跟踪每个数据流的状态，资源占用少，扩展性较强；且能实现对不同业务提供不同的服务质量
#### 缺点
需要在端到端每个节点都进行手工部署，对人员能力要求较高
# 工作原理
## 流量分类与标记
### 为什么需要分类
如果不分类，不知道那些流量需要特殊服务
### 常见的流量类型
ICMP、http、协议、www、tcp、udp
### 分类方式与标记
#### 复杂流分类
- 匹配规则复杂，匹配到流量后，对流量进行外部优先级的标记
  - 匹配五元组（源目IP、端口，协议）
  - 匹配源、目标MAC
  - 匹配VLAN
- 作用位置：边界DS设备上，调用在入接口
- 配置方式：使用MQC进行分类、打标
![](/images/Qos分类与标记示意图.png)
#### 简单流分类
- 匹配规则简单
  - 匹配IP报文的DSCP值
  - 匹配MPLS报文的EXP阈值
  - 匹配VLAN报文的802.1q值

  匹配后通过映射关系可以实现修稿报文发送出去时所携带的优先级
- 作用位置：内部DS设备
  ![](/images/Qos分类与标记示意图.png)
  - 优先级映射原理
    - 可以根据配置修改报文发送出去时所携带的优先级，以便其他设备根据报文的优先级提供相应的Qos服务
    - 还可以对进入设备的报文，设备将报文携带的外部优先级映射为内部优先级，然后根据内部优先级与队列之间的映射关系确定报文进入的队列，从而针对队列进行流量整形、拥塞避免、拥塞管理等处理
#### 标记位置
- 二层网络

  在VLAN802.1q标记中的PRI字段打标记，3个bit范围0-7，数据越大，等级越高，优先转发，8种服务类型，也叫COS
  ![](/images/Qos二层网络标记位置.png)
- MPLS
  ![](/images/QosMPLS标记位置.png)
  EXP，3个bit，8种服务类别0-7
- IP网络
  ![](/images/QosIP网络标记位置.png)
  - TOS字段中只有3个bit，定义优先级的是IPP字段，8种服务类型
  - TOS字段，后来变为DSCP域，6bit，64种服务类型，扩充了服务的功能。0-63
### 配置
#### 拓扑
  ![](/images/Qos网络配置拓扑.png)
#### 需求
1. AR1做复杂流分类（MQC方式）将ip报文中的dscp优先级修改为cs6（48）
2. AR2做简单流分类（基于优先级映射方式）在AR2入方向将cs6改为ef（48-46）
#### 步骤
1. 复杂流分类
    - 匹配UDP目标端口号8000的流量
    - 定义流分类
    - 定义流行为
    - 定义流策略
    - 入接口下调用流策略
2. 简单流策略
    - 入方向配置新人接口
    - 配置优先级映射
    - 抓包查看优先级
#### 命令
```
1）复杂流策略：
acl number 3000  
 rule 5 permit udp destination-port eq 8000 
#
traffic classifier 1 operator or
 if-match acl 3000
#
traffic behavior 1
 remark dscp cs6
#
traffic policy 1
 classifier 1 behavior 1
 #
interface GigabitEthernet0/0/0
 ip address 192.168.1.254 255.255.255.0 
 traffic-policy 1 inbound
2）简单流策略
interface GigabitEthernet0/0/0
 ip address 10.1.12.2 255.255.255.0 
 trust dscp override
#
qos map-table dscp-dscp
  input 48 output 46
```
## 拥塞管理
### 基于队列
#### 原理
##### 首先进行优先级映射
通过复杂流策略标记外部优先级---（自动映射）--->内部优先级---（自动映射）--->内部队列，通过队列调度实现差异化服务，队列数值越大，越优先转发
- 802.1q/Exp
  |外部（标记）优先级|内部优先级|队列|
  |-|-|-|
  |0|0|0|
  |1|1|1|
  |2|2|2|
  |3|3|3|
  |4|4|4|
  |5|5|5|
  |6|6|6|
  |7|7|7|
- DSCP
  |外部（标记）优先级|内部优先级|队列|
  |-|-|-|
  |0-7|0|0|
  |8-15|1|1|
  |16-23|2|2|
  |24-31|3|3|
  |32-39|4|4|
  |40-47|5|5|
  |48-55|6|6|
  |56-63|7|7|
- 标记外部优先级解析
  ![](/images/Qos标记外部优先级解析.png)
  - 只有1-4执行丢弃优先级，先匹配队列优先级，对应好后，去寻找丢弃优先级，丢弃的话会从内部优先级小的然后丢弃优先级大的先进行丢弃
  - 外部优先级字母与数字对应关系
    ![](/images/Qos外部优先级字母与数字的对应关系.png)
  - 解析：对于重要的业务，比如开发流量，低延迟的语音业务，非常重要，我们可以选择高优先级来进行标记，比如CS5、CS6、EF，网络拥塞时不丢弃，优先转发
  - 对于不重要的流量可以选择AF，当网络发生拥塞时，可以根据丢弃优先级进行有选择的丢弃
##### 其次对应进入队列
默认每个接口对应8个队列，编号0-7，数值越大，优先级越高，通过内部优先级自动进入队列
###### 查看外部优先级到内部优先级的映射表
```
 <Huawei>display diffserv domain all 
diffserv domain name:default
 8021p-inbound 0 phb be green
 8021p-inbound 1 phb af1 green
 8021p-inbound 2 phb af2 green
 8021p-inbound 3 phb af3 green
 8021p-inbound 4 phb af4 green
 8021p-inbound 5 phb ef green
 8021p-inbound 6 phb cs6 green
 8021p-inbound 7 phb cs7 green
 ip-dscp-inbound 0 phb be green
 ip-dscp-inbound 1 phb be green
 ip-dscp-inbound 2 phb be green
 ip-dscp-inbound 3 phb be green
 ip-dscp-inbound 4 phb be green
 ip-dscp-inbound 5 phb be green
 ip-dscp-inbound 6 phb be green
 ip-dscp-inbound 7 phb be green

```
###### 查看内部优先级到队列的映射表
```
<Huawei>display  qos  local-precedence-queue-map
 Current configurations of mapping between local-precedence and que
  local-precedence value: be    queue index: 0 //默认流量都在0号队列转发
  local-precedence value: af1   queue index: 1   
  local-precedence value: af2   queue index: 2
  local-precedence value: af3   queue index: 3
  local-precedence value: af4   queue index: 4
  local-precedence value: ef    queue index: 5
  local-precedence value: cs6   queue index: 6
  local-precedence value: cs7   queue index: 7
```
我们通过标记外部优先级，通过内部优先级对应到队列，接下来要为队列配置不同的队列调度机制，实现差异化的服务，高优先级走快速转发，不丢弃，低优先级的可以丢弃
##### 最后进行队列调度
###### 队列调度算法
- 先进先出FIFO（first in first out）
  - 概念：先进先出，机制简单，无法对不同的优先级进行区分
  - 优点：实现机制简单且处理速度快
  - 缺点：不能有差别的对待优先级不同的报文
- 严格优先级SP（Strict Priority）
  - 概念：严格按照队列优先级的高低顺序进行调度
  - 优点：对高优先级的报文提供了优先转发
  - 缺点：低优先级队列可能出现“饿死”现象
- 加权公平队列WFQ（Weighted Fair Queuing）
  - 概念：按队列权重来分配每个流占有出口的带宽
  - 优点：不同的队列获得公平的调度机会，从总体上均衡两个流的延迟；权重大，分配的带宽多，权重小，分配的带宽少
  - 缺点：低时延业务仍得不到即时调度；
###### 队列调度方式
- PQ
  - 概念：PQ（Peiority Queuing）调度，PQ队列之间采用SP调度算法，严格按照优先级调度，高优先级调度完，才调度低优先级
  - 优点：确保高优先级业务优先发送
  - 缺点：低优先级业务被饿死
- RR
  - 概念：轮巡（Round Robin）
  - 优点：每个队列公平调度
  - 缺点：无法实现区分服务，重要流量无法保证优先转发
- WRR
  - 概念：加权轮询WRR（Weighted Round Robin），采用加权轮询WRR，解决了RR不能设置权重的问题，WRR加入权重值，**`权重值大的调度次数多`**，如图，有34个队列，高优先级队列设置权重值为50%，低优先级权重值设置25%，此时第一轮调度方式为各拿一个包，第二轮只有高优先级能得到调度，低优先级不能调度；第三轮再重新开始
    ![](/images/QosWRR加权轮询示意图.png)
  - 优点：避免了采用SP调度时低优先级队列中的报文可能长时间得不到服务的站点；避免了轮询无法区分重要流量的问题
  - 缺点：低延时需求业务（如语音）得不到及时调度
- WFQ
  - 概念：WFQ（Weighted Fair Queuing）**`基于权重值分配带宽`**
    - 加入有单个低优先级队列，3号队列分配权重值50%，2号和1号队列分配权重值分别为25%，得到的带宽计算公式为分配的带宽=权重值/总权重值*接口带宽（1000M）；根据公式计算3号队列分到的带宽为500M，2号和1号得到的带宽为250M
  - 优点不同的队列获得公平的调度机会，从总体上均衡各个流的延迟；权重大，分配的带宽就多，权重小，分配的带宽就少
  - 缺点：低延时业务仍得不到即使调度
- LPQ
  - 与PQ的区别：当接口同时配置了WFQ和PQ队列时，网络拥塞时PQ会抢占WFQ队列贷款，**`LPQ不会抢占WFQ带宽`**
  - 在PQ和WFQ队列调度结束后，如果还有剩余带宽则分配给LPQ，用于转发不重要的流量
- **`综合应用`**
  - 场景
    - 5-7号队列用于研发业务最重要保证优先转发，使用PQ（先严格按照优先级来进行处理）队列
    - 3-4号队列用于办公流量，使用WFQ（基于权重值分配带宽）队列
    - 0-3号队列视频流量不重要，使用LPQ（在不影响WFQ的情况下，占用剩下带宽转发流量）调度
      ![](/images/Qos综合应用示意图.png)
    - 调度方式：若PQ队列中有报文，则总是优先发送PQ队列中的报文，直到PQ队列中没有报文时，在调度WFQ队列，等到WFQ中的报文被发送完毕，最后才调度LPQ队列中的报文
#### 配置
##### 拓扑
![](/images/Qos混合模型配置拓扑.png)
##### 需求
1. 研发流量最重要，当网络发生拥塞时，优先保证研发流量转发
2. 视频流量不重要，当网络发生拥塞时，可以根据丢弃概率进行丢弃
##### 步骤
1. AR1上进行复杂流分类，研发标记为CS7，视频标记为AF13
    - 匹配研发流量和视频流量
    - 定义流分类
    - 定义流行为
    - 定义流策略
    - 分别在入接口下调用流策略
2. 配置低优先队列0-4使用WFQ调度，配置高优先级队列5-7使用PQ调度
##### 命令
```
AR1复杂流策略配置：
acl number 3000      
 rule 10 permit icmp source 192.168.1.1 0 destination 192.168.2.1 0 
acl number 3001     
 rule 5 permit icmp source 192.168.3.1 0 destination 192.168.2.1 0 
#
traffic classifier 2 operator or
 if-match acl 3001
traffic classifier 1 operator or
 if-match acl 3000
#
traffic behavior 2
 remark dscp af13
traffic behavior 1
 remark dscp cs7
#
traffic policy 2
 classifier 2 behavior 2
traffic policy 1
 classifier 1 behavior 1
#
interface GigabitEthernet0/0/0
 traffic-policy 1 inbound
#
interface GigabitEthernet0/0/2
 traffic-policy 2 inbound
AR2配置基于队列的拥塞管理：
interface GigabitEthernet0/0/0
 trust dscp override
#
qos queue-profile 1
  schedule wfq 0 to 4 pq 5 to 7
#
interface GigabitEthernet0/0/1
 qos queue-profile 1
 ================================================================
 查看1：查看流量是否进入队列
 [ar2]display  qos queue statistics interface GigabitEthernet 0/0/1
-----------------------------------------------------------------------------
Queue            Passed          Pass Rate        Dropped         Drop Rate
            (Packets/Bytes)      (pps/bps)    (Packets/Bytes)     (pps/bps)
-----------------------------------------------------------------------------
Protocol             485/              0/              0/              0/
                   51662               0               0               0 
0                      5/              0/              0/              0/
                     490               0               0               0 
1                      3/              0/              0/              0/
                     294               0               0               0 
2                      0/              0/              0/              0/
                       0               0               0               0 
3                      0/              0/              0/              0/
                       0               0               0               0 
4                      0/              0/              0/              0/
                       0               0               0               0 
5                      0/              0/              0/              0/
                       0               0               0               0 
6                      0/              0/              0/              0/
                       0               0               0               0 
7                      1/              0/              0/              0/
                      98               0               0       
 
 查看2：队列调度方式
 [ar2]display qos queue-profile 1
Queue-profile: 1
Queue  Schedule  Weight  Length(Bytes/Packets) GTS(CIR/CBS)
-----------------------------------------------------------------
0      WFQ       10                 -/-              -/-     
1      WFQ       10                 -/-              -/-     
2      WFQ       10                 -/-              -/-     
3      WFQ       10                 -/-              -/-     
4      WFQ       10                 -/-              -/-     
5      PQ        -                  -/-              -/-     
6      PQ        -                  -/-              -/-     
7      PQ        -                  -/-              -/-   
```