---
author: zty
title: Qos
date: 2024-04-12
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
### 基于类的MQC（Modular Qos Command-Line Interface）
#### 原理
##### 复杂流分类
根据IP优先级或者DSCP优先级、输入接口、IP报文的五元组等规则来对报文进行分类，然后让不同类别的报文进入**不同的队列**(CBQ提供)

对于不匹配任何类别的报文，送入系统定义的缺省类
##### CBQ队列
概念Class-based Queueing， 基于流策略队列，提供了三类队列调度机制
- EF
  - 概念：加速转发，使用PQ（严格按照优先级来进行流量转发）算法，优先调度，当EF或LLQ队列中的报文调度完毕后，才会调度其他队列中的报文，其他队列空闲会抢占
  - 分类
    - 普通EF队列，VIP配置最小带宽，当网络发生拥塞时，确保业务转发
    - LLQ低延时队列，VIP种P，需要配置最大带宽，防止一个业务将所有带宽全部占满，导致其他流量无法转发
- AF
  - 概念：确保转发，使用WRR（加权轮询）算法
  - 特点：需要配置最小带宽，当网络发生拥塞时，确保业务转发
- BE
  - 概念：尽力而为，使用WFQ（按权重分配带宽）算法
  - 特点未进入AF队列和EF队列的剩余报文进入BE队列有剩余带宽就转发，没有就丢弃
#### 配置
##### 拓扑
![](/images/Qos基于类的MQC拓扑.png)
##### 需求
保证研发流量优先转发，最大带宽50M；办公流量确保转发，保证带宽20M
##### 步骤
AR2使用MQC配置，指定研发队列EF（LLQ），指定办公队列AF（确保转发）
- 匹配研发流量和办公流量
- 定义流分类
- 定义流行为
  - 指定EF队列
  - 指定AF队列
- 定义流策略
- **在出接口下调用流策略**
##### 命令
```
AR2上MQC配置：
acl number 3000  
 rule 10 permit icmp source 192.168.1.1 0 destination 192.168.2.1 0 
acl number 3001  
 rule 5 permit icmp source 192.168.3.1 0 destination 192.168.2.1 0 
#
traffic classifier 2 
 if-match acl 3001
traffic classifier 1 
 if-match acl 3000
#
traffic behavior 2
 queue af bandwidth 20000
traffic behavior 1
 queue llq bandwidth 50000 cbs 1250000//cbs代表突发流量，默认最大带宽的25倍
#
traffic policy a
 classifier 1 behavior 1
 classifier 2 behavior 2
#
interface GigabitEthernet0/0/1
 traffic-policy a outbound
```
### 区别
- 基于队列的需要配置信任dscp，只有信任dscp才能进行优先级映射，在出接口调用qos queue-profile 1模板
- MQC不需要配置接口信任dscp，匹配流量后直接手动强制进队列，在出接口调用策略，常用
## 拥塞避免
### 概念
通过队列调度后依然处于拥塞，无法正常发送数据流量，需要通过选择性丢包来进行拥塞避免，两种方式常结合使用。不论是哪种方式，拥塞时都会有丢包，最好的方式就是升级硬件和带宽，但是成本太高
#### 类比
##### 拥塞管理
交警指挥下班晚高峰车辆，发现救护车，救护车走专业车道，优先走；公交车，走公交车道，先走；私家车，先堵着，等救护车走完了再走，这就是用队列调度实现的拥塞管理
##### 拥塞避免
知道晚高峰一定会堵车，重要流量可以通过，就好比救援车辆，公交车可以通过，看快要堵车了，交警会通知其他车辆不允许进入市区，不通过也就是流量不通了，从而实现有选择性地丢包，可以选择先丢弃谁，后丢弃谁
### 原理
#### 缓存队列满了再丢
缓存队列满了再丢，也就是尾丢弃技术，该技术有以下缺点：
- TCP全局同步过程
  - 因为TCP机制需要对方确认，当网络发生拥塞时，TCP接收不到确认报文，认为网络发生拥塞，TCP根据滑动窗口机制，减小发送的数据报文，环节王洛勇赛，建立TCP连接，当流量畅通时，TCP又调大自己的滑动窗口，造成网络中流量忽高忽低，形成震荡
    ![](/images/Qos拥塞避免滑动窗口机制.png)
- 无差别丢弃
  - 可能会导致某些重要流量被丢弃，无法保证数据的完整性
- TCP饿死
  - 以UDP传送机制的决定，报文转发快，不需要重传和确认，拥塞的链路充斥着大量的UDP流量，导致TCP饿死
#### 提前丢包
- RED（早期随即检测）
  ![](/images/QosRED.png)
  问题：丢弃不分优先级：可能导致重要的流量丢失，无法保证完整性
- WRED（加权早期随即检测）
  - WRED技术可以通过对不同优先级数据包或队列设置相应的丢弃策略，以实现对不同流量进行区分丢弃。（常用）
    ![](/images/QosWRED.png)
### 配置
#### 基于队列
##### 拓扑
![](/images/Qos拥塞避免拓扑.png)
##### 需求
1. 已经配置了基于队列的拥塞管理，研发流量标记优先级AF31、视频流量标记优先级AF13
2. 增加配置基于队列的拥塞避免
##### 步骤
1. 配置丢弃模板
2. 在队列模板中调用丢弃模板
3. 在接口出方向调用队列模板
##### 命令
```
AR2配置基于队列的拥塞避免
第1步：配置丢弃模板
drop-profile 1
 wred dscp
  dscp af13 low-limit 30 high-limit 80 discard-percentage 80
drop-profile 2
 wred dscp
  dscp af31 low-limit 50 high-limit 80 discard-percentage 30
#
第2步：队列模板调用丢弃模板
qos queue-profile 1
  schedule wfq 0 to 4 pq 5 to 7  /必须是wfq队列，才能配置WRED
  queue 1 drop-profile 1
  queue 3 drop-profile 2
第3步：出接口应用
interface GigabitEthernet0/0/1
 ip address 10.1.23.2 255.255.255.0 
  qos queue-profile 1
```
#### 基于类的MQC
##### 拓扑
![](/images/Qos拥塞避免拓扑.png)
##### 需求
配置基于MQC的拥塞避免
##### 步骤
- 配置丢弃模板
- 配置研发流量和视频流量
- 定义流分类
- 定义流行为
  - 调用丢弃模板
- 定义流策略
- 出口下调用流策略
##### 命令
```
AR2配置基于MQC的拥塞避免
第1步：创建丢弃模板
drop-profile 1
wred dscp
  dscp af13 low-limit 30 high-limit 80 discard-percentage 80
第2步：MQC配置
traffic classifier 1 operator or
 if-match dscp af13     //匹配已经标记的外部优先级 
#
traffic behavior 1
queue af bandwidth 50000 //必须要有基于类的拥塞管理配置，
否则无法配置拥塞避免
 drop-profile 1
#
traffic policy 1
 classifier 1 behavior 1
#
interface GigabitEthernet0/0/1
 traffic-policy 1 outbound
```
## 流量监管与整形
### 概念
流量监管和流量整形都属于限速技术
- 流量监管TP：
  - 动作：超过的流量，直接丢弃
  - 方向：出、入方向都可以，一般作用于入方向
  - 特点：丢包率高
  - 原理：令牌桶
- 流量整形TS
  - 动作：超过的流量，缓存，空闲再发
  - 方向：只能作用于出方向
  - 特点：时延高
  - 原理：令牌桶
### 原理
#### 单速单桶
##### 参数
- CIR：承诺信息速率，表示系统向C同中投放令牌的速率，即C桶允许传输或转发报文的平均速率
- CBS：承诺突发尺寸，表示C桶的容量，即C同瞬间能够通过的承诺突发流量
##### 原理
- 令牌桶中会存放一定的令牌，根据一定的速率周期性的放入令牌，当令牌桶满时不再放入，当有突发流量时根据数据大小与令牌桶容量的比较，如果令牌桶容量大于发来的数据包长度，则将数据报文标记为绿色，如果数据流量大于令牌桶数量，流量将被抛弃
- 为什么设计令牌桶，在使用令牌桶对流量进行评估时，是以令牌桶中的令牌数量是否足够满足报文的转发为依据的。如果桶中存在足够的令牌可以用来转发报文，称流量遵守或符合约定值，允许通过，否则称为不符合或超标，将报文丢弃
![](/images/Qos单速单桶.png)
##### 颜色
- 绿色：当报文长度小于C桶容量，减少C桶令牌，转发
- 红色：当报文长度大于C桶容量，丢弃
#### 单速双桶
##### 参数
EBS：超额突发尺寸，表示E桶的容量，即E桶瞬间能够通过的超出突发流量
##### 原理
- 系统通过承诺信息CIR，向令牌桶中投放令牌，当CBS桶投满后，再向EBS桶投放，EBS投满后不再增加令牌桶的数量
- 收到突发需要监管的流量时，先用CBS桶的令牌转发，如果够用直接转发，如果不够，再用EBS桶中的令牌，够用转发，不够丢弃
![](/images/Qos单速双桶.png)
##### 颜色
- 绿色：当报文长度小于C桶容量，减少C桶令牌，转发
- 黄色：对突发流量的管理，默认转发；
当报文大于C同小于E同，只减少E桶令牌
- 红色：如果报文长度大于E同中的令牌，报文被标记为红色，并且丢弃
#### 双速双桶
##### 参数
- PIR：峰值信息速率，表示向P桶中投放令牌的速率，即P桶允许传输或转发报文的峰值速率，PIR大于CIR；
- CIR：承诺信息速率，表示向C桶中投放令牌的速率，即C桶允许传输或转发报文的平均速率
- PBS：峰值突发尺寸，表示P桶的容量，即P桶瞬间能通过的峰值突发流量，PBS大于CBS
- CBS：承诺突发尺寸，表示C桶的容量，即C桶瞬间能够通过的承诺突发流量
##### 原理
令牌投放规则
- 按照CIR速率向C桶中投放令牌，当投放令牌数量小于CBS（存储容量）时，C桶中令牌数增加，否则不增加
- 按照PIR速率向P桶中投放令牌，当投放令牌数量小于PBS（峰值存储容量）时，P桶中令牌数增加，否则不增加
- PIR大于CIR；PBS大于CBS；PBS还是针对处理突发流量
![](/images/Qos双速双桶.png)
##### 颜色
- 绿色：当报文小于CBS桶的容量则两个桶的令牌都减
- 黄色：当报文大于CBS桶的容量，小于PBS容量，只减少PBS的令牌
- 红色：当报文大于PBS桶的容量，直接丢弃
##### 区别
- 单桶与双桶的区别：允许短暂的突发流量
- 双速双桶与单速双桶的区别：允许长期的突发流量
### 配置
#### 流量监管
##### 拓扑
![](/images/Qos拥塞避免拓扑.png)
##### 基于接口
```
AR2上入接口配置：
interface GigabitEthernet0/0/0
 qos car inbound cir 8 Kbps 

回显信息：
interface GigabitEthernet0/0/0
 qos car inbound cir 8 cbs 1504 pbs 2504 green pass yellow pass red disc
 如果只配置了cir,默认cbs是cir的188倍;pbs是cir的313倍，默认单速双桶，可配置双速双桶
查看命中信息：
[ar2]display qos car statistics interface GigabitEthernet 0/0/0 inbound
 Interface : GigabitEthernet0/0/0
 Direction : Inbound
---------------------------------------------------------------------------
Type               Packets/Bytes                    Rate(pps/bps)         
---------------------------------------------------------------------------
Drop                    0/0                             0/0               
Pass                   18/1846                          0/0               
Green                  18/1846                          0/0               
Yellow                  0/0                             0/0               
Red                     0/0                             0/0           
```
##### 基于类的MQC
```
AR2上入接口配置：
实验一：限速8K
#
traffic classifier 1 operator or
 if-match dscp af31 
#
traffic behavior 1
 car cir 8                      //默认动作一样car cir 8 cbs 1504 pbs 2504 
#
traffic policy 1
 classifier 1 behavior 1
#
interface GigabitEthernet0/0/0
 traffic-policy 1 inbound
 实验二：企业案例-限速50M
#
acl number 3100  
 description XianSu-50M
 rule 5 permit ip source 192.168.1.1 0 destination 192.168.2.1 0
#
traffic classifier XianSu-50M  
 if-match acl 3100
#
traffic behavior XianSu-50M
 car cir 51200 pir 80000 cbs 3200000 pbs 3200000
#
traffic policy XianSu-50M 
 classifier XianSu-50M behavior XianSu-50M
#
interface GigabitEthernet1/0/0
 traffic-policy XianSu-50M inbound
-----------------------------------------------------------------------
单位换算：
KB/s的含义是兆字节每秒，Kbit/s的含义是兆比特每秒
1Byte = 8bit => 1MB/s = 8Mbit/s
形容网速的单位：cir:单位为Kbit/s
1Kb=1024b
1Mb=1024Kb
1Gb=1024Mb
形容存储容量的单位：
1B=8bit
1KB=1024B
1MB=1024KB
1GB=1024MB
```
#### 流量整形
##### 基于接口
```
AR2上出接口配置
interface GigabitEthernet0/0/1
 qos gts cir 8
回显信息：
interface GigabitEthernet0/0/1
 qos gts cir 8 cbs 1500   //只能配置单速单桶，缺省情况下，
 cbs值为cir值的25倍，当cir值的25倍小于1500时，取1500

```
##### 基于类的MQC
```
traffic classifier 1 operator or
 if-match dscp af31 
#
traffic behavior 1
 gts cir 8 cbs 1500 queue-length 1  
#
traffic policy 1
 classifier 1 behavior 1
#
interface GigabitEthernet0/0/1
 traffic-policy 1 outbound
解析： queue-length 1  指定队列最多可存储的报文数
```
### 易混淆概念
CIR、CAR、带宽、交换机接口速率概念区分
```
网络中带宽的单位通常是比特每秒（bps，bits per second）。
这个单位描述的是网络传输速度的标准，即每秒钟可以传输多少比特的数据。
除了基本单位bps外，还有根据SI（国际单位制）和IEC（国际电工委员会）
标准定义的多种更高阶的单位，用于表示更大的数据传输速率：
形容网速/带宽的单位：
Kbps（千比特每秒）：1 Kbps = 1,000 bps
Mbps（兆比特每秒）：1 Mbps = 1,000 Kbps 
Gbps（吉比特每秒）：1 Gbps = 1,000 Mbps 
Tbps（太比特每秒）：1 Tbps = 1,000 Gbps 
形容存储容量的单位：
1B=8bit
1KB=1024B
1MB=1024KB
1GB=1024MB
```
CIR、CAR、带宽以及交换机接口速率这几个概念在网络技术中都和数据传输能力相关，但它们指的是不同的事情：

1. **CIR（Committed Information Rate，承诺信息速率）**：CIR是在帧中继（Frame Relay）和其他虚拟专用网络（VPN）技术中使用的概念，它表示网络服务提供商（ISP）保证为客户提供的最小数据传输速率。CIR是服务质量（QoS）协议的一部分，确保数据传输具有一定的性能保障。如果数据传输速率低于CIR，客户可以向ISP索赔。
2. **CAR（Committed Access Rate，承诺接入速率）**：来对流量进行控制，利用令牌桶来衡量每个数据报文是超过还是遵守所规定的报文速率；

      CAR是一种网络流量管理技术，用于控制进入或离开网络接口的数据流量速率。通过对流量进行限制（限速）和/或保证最小带宽，CAR确保网络资源的合理分配。CAR可以帮助防止网络拥塞，确保关键应用获得必要的带宽。

1. **带宽**：带宽通常指网络连接的最大传输能力，即网络每秒能传输的最大数据量。它是衡量网络传输能力的指标，单位通常是比特每秒（bps）及其倍数，如Mbps、Gbps等。
2. **交换机接口速率**：交换机接口速率指的是网络交换机（或路由器）上各个物理端口支持的最大数据传输速率。这个速率取决于端口的硬件设计，如10/100/1000 Mbps以太网端口或更高速率的10 Gbps、40 Gbps等接口。交换机接口速率定义了通过该端口可能实现的最高速度，实际速度可能受到连接的设备、网络条件和配置的影响。

总结来说，CIR和CAR都是与网络服务质量和流量管理相关的概念，它们定义了网络传输的最小保证速率和对流量的管理策略。而带宽是一个更广泛的概念，描述的是网络通信线路的最大数据传输能力。交换机接口速率则是指特定网络硬件接口支持的最大传输速率。这些指标各自反映了网络性能和能力的不同方面。

这些概念的单位都是以数据传输速率来衡量的，具体如下：

1. **CIR（承诺信息速率）和**CAR（承诺接入速率）：单位通常是比特每秒（bps）及其倍数，如Kbps（千比特每秒）、Mbps（兆比特每秒）、Gbps（吉比特每秒）等。这些单位用来衡量网络服务提供商保证的最小数据传输速率（CIR）或网络流量管理策略下控制的数据传输速率（CAR）。
2. **带宽**：单位也是比特每秒（bps）及其倍数。它描述的是网络连接的最大传输能力，即在理想条件下，网络每秒能传输的最大数据量。
3. **交换机接口速率**：同样使用比特每秒（bps）及其倍数作为单位。这个速率指的是网络交换机或路由器上各个物理端口支持的最大数据传输速率。例如，常见的以太网端口有10/100/1000 Mbps（即1 Gbps）或更高速率的10 Gbps、40 Gbps等

想象一下，我们的网络连接就像是一条高速公路，而数据包就像是在这条高速公路上行驶的车辆。

**带宽**就像是这条高速公路的宽度。高速公路越宽，同时能容纳的车辆就越多，车辆就能更快速、更顺畅地通过。如果带宽很大，意味着网络能够同时处理大量的数据传输，就像一条有很多车道的宽广高速公路可以同时容纳很多车辆一样。但这并不意味着每辆车（或数据包）的速度就一定会很快，它只是说明了通路可以容纳的流量大小。

**交换机接口速率**则可以看作是连接到这条高速公路上的入口或出口的最高速度限制。比如，如果一个交换机接口的速率是1 Gbps，那就好比这个入口或出口的速度限制是1 Gbps。无论高速公路（带宽）本身有多宽，车辆（数据包）通过这个入口或出口的速度是有上限的，这个上限就是由交换机接口速率决定的。

综合来看，即使你的高速公路（带宽）非常宽阔，能够容纳大量的车辆流量，如果高速公路的入口或出口（交换机接口速率）限制了速度，那么车辆（数据）进入或离开高速公路的速度就会受到这个限制。反之，如果入口或出口非常宽敞（高接口速率），但高速公路本身不够宽（带宽较小），那么车辆可能在高速公路上遇到拥堵，导致实际行驶速度降低。

所以，理想的网络环境应该是拥有足够宽的带宽来处理高流量的数据传输，同时具备高速的交换机接口来确保数据快速流入流出。

网络术语中，**接口速率**和**接口带宽**通常被用来指代相似的概念，尤其是在讨论交换机接口的上下文中。两者都指一个网络接口能够达到的最大数据传输速率。比如，一个1Gbps的接口速率或带宽意味着理论上这个接口每秒可以传输1亿比特（bit）的数据。

但是，根据不同的上下文，这两个术语间的区别可能会有细微的不同：

- **接口速率**通常指的是物理层面上接口能支持的最大传输速率，它是由接口的物理硬件特性决定的。这个速率是一个理论上的最大值，不考虑网络的实际负载或任何可能的网络拥堵。
- **接口带宽**在某些情况下可以用来表示实际的、可用的数据传输能力，这可能会受到网络策略、配置或实际网络流量的影响。 