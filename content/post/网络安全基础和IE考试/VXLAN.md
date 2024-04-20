---
author: zty
title: CSS/WLAN/Eth
date: 2024-04-16
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
# 设备虚拟化
## 堆叠概述
### 概述
- 将多台交换机设备通过线缆连接组合在一起，虚拟成一台设备 ，是一种虚拟化技术，通过交换机进行堆叠，可以实现网络可靠性和网络大数据量转发，同时简化网络管理
- 堆叠ID
  - 堆叠ID，即堆叠交换机的编号，用来标识堆叠交换机
  - 交换机堆叠ID默认为0（越大越优先）
- 堆叠优先级
  - 用于主备选举，默认100
- 冗余方案
  - VRRP+MSTP
  - 堆叠+链路聚合
### 堆叠的分类
- 盒式交换机iStack：多台设备堆叠（华为盒式交换机-中低端）
- 框式集群交换机系统CSS：两台设备集群（华为框式设备-高端）
### 堆叠方式
- 通过堆叠卡和堆叠线
  - 盒式堆叠卡
    ![](/images/CSS盒式堆叠卡.png)
  - 框式堆叠卡堆叠
    - S系列框式交换机：需要单独购买堆叠板卡，和堆叠线缆，且不同类型的堆叠设备对应的设备堆叠板卡是不同的，不匹配型号的堆叠板卡，也能插上，线也能连接，但是重启会报错，堆叠组建不起来
    ![](/images/CSS框式堆叠卡.png)
    - 特点：不需要单独的占用业务口作为堆叠端口。相当于内部总线连接，配置简单，可靠性高；CE交换机通过业务端口改造成堆叠端口，不需要特殊的堆叠板卡和堆叠线
- 业务口堆叠
  - 盒式交换机
    ![](/images/CSS盒式业务口堆叠.png)
  - 框式交换机
    ![](/images/CSS框式业务口堆叠.png)
### 管控平面
  ![](/images/CSS管控主备.png)
- 管理平面、控制平面、转发平面合一
  - 管控平面：主备工作模式，管控平面合一，不需要配置；
- 主备原则：主备板卡异框，切换顺序
  - 转发平面：负载分担，都转发流量，将业务端口加入到堆叠成员端口，将两个转发平面合一，用于同步MAC，ARP及路由表；
  - 带外管理平面，走单独的带外管理网络，因为管控平面合一，对外呈现一个管理IP地址；
### 堆叠的角色
- 主交换机（Master）：负责管理整个堆叠，堆叠中只有一台主交换机
- 备交换机（Standby）：
  - 主交换机的备份交换机，当主交换机故障时，备交换机会接替原主交换机的所有业务
  - 堆叠中只有一台备交换机
- 从交换机（Slave）
  - 主要用于业务转发，从交换机数量越多，堆叠系统的转发能力越强
  - 除主交换机和备交换机之外，堆叠中其他所有的成员都是从交换机
### 主备竞选原则
1. 运行状态比较，已经运行的交换机比处于启动状态的交换机优先竞争为主交换机
2. 优先级高的为主
3. MAC地址小的为主
### 堆叠成员加入
- 新加入的堆叠交换机，会进行角色选举，新加入的交换机会选举为从交换机，堆叠系统中原有主备从角色不变
- 角色选举结束后，主交换机更新堆叠拓扑信息
- 新加入的交换机更新堆叠ID，并同步主交换机的配置文件，稳定运行
### 多主检测（发生脑裂）
- 为什么多主检测
  - 主控板发生故障时（就是其实没坏但是备认为主坏了，连接不上，就自己成为主了），两台设备会有相同的IP和路由，同时对外承载业务，此时可以依靠堆叠的双主检测来避免堆叠分裂后出现双主。配置了MAD（多主检测）后主交换机和备交换机之间定时发送心跳报文来维护堆叠系统的状态
  - 检测方式：
    - 业务口直连
    - 管理口
  - DAD检测原理：周期发送DAD探测报文，当管理板卡故障时，会通知对方设备，不要变成主设备，我是主，你把端口error_down就可以了，这样做流量就会只从主走了
  - 故障恢复：loser状态的设备，将重新启动，重新加入堆叠域，同时将error-down的业务端口恢复正常，整个堆叠系统恢复正常
## 堆叠配置
```
1、集群卡方式(S12706)：
配置SwitchA的集群连接方式为集群卡集群，集群ID为1，集群优先级为100。
<HUAWEI> system-view
[HUAWEI] sysname SwitchA
[SwitchA] set css mode css-card
[SwitchA] set css id 1                 G1  /1/1/0
[SwitchA] set css priority 100
配置SwitchB的集群连接方式为集群卡集群，集群ID为2，集群优先级为10。
<HUAWEI> system-view
[HUAWEI] sysname SwitchB
[SwitchB] set css mode css-card
[SwitchB] set css id 2
[SwitchB] set css priority 10
------------------------------------------------------------------------------
2、业务口方式(S5700)
配置堆叠ID和堆叠优先级
# 配置SwitchA的堆叠优先级为200
[SwitchA]
         stack slot 0 priority 200       
[SwitchA] interface stack-port 0/1
[SwitchA-stack-port0/1] port interface gigabitethernet 0/0/27 enable
                        port interface gigabitethernet 0/0/28 enable

# 配置SwitchB的堆叠ID为1
[SwitchB] stack slot 0 renumber 1   
          
[SwitchB] interface stack-port 0/2 
[SwitchB-stack-port0/2] port interface gigabitethernet 0/0/27 enable
                        port interface gigabitethernet 0/0/28 enable
```
**配置堆叠注意事项：**
  - **先配置主，后配置备**
  - **保存，下电**
  - **连接堆叠线**
  - **先开启主，后开启备**
  
  **注意：堆叠端口不能相同，否则堆叠无法建立**
# 链路聚合
## Eth-trunk（链路聚合技术）
eth-trunk可以把多个独立的物理接口绑定在一起，作为一个大带宽的逻辑接口使用
## eth-trunk优势
- 增加设备间的互联带宽
- 提高设备间的可靠性
- 对流量负载均衡，提高链路利用率
## eth-trunk链路聚合模式
- 手工负载分担模式（S系列）
- 静态LACP模式（S系列）
- 动态LACP模式（CE系列）
## eth-trunk接口配置流程
- 创建eth-trunk
- 选择链路聚合模式
- 在eth-trunk中加入成员接口
## 手工负载分担模式
- 手工模式下3条活动链路都参与数据转发并分担流量
- 当一条链路故障时，在剩余的两条活动链路中分担流量
![](/images/eth手工模式.png)
### 配置手工模式Eth-trunk
#### 拓扑
![](/images/eth手工模式拓扑.png)
#### 需求
- 对交换机之间的链路进行链路捆绑，增加互联带宽
- 确保同VLAN的PC之间互通
#### 配置步骤
1. PC配置IP地址
2. 所有交换机创建vlan10和vlan20
3. 交换机和PC互联接口设置为access，并加入指定vlan
4. 创建eth-trunk
5. 配置eth-trunk的工作模式为手工模式
6. eth-trunk中加入成员接口
#### 配置命令
```
SW1配置：
[SW1]vlan batch 10 20   
[SW1]interface G0/0/3 
[SW1-G0/0/3]port link-type access    
[SW1-G0/0/3]port default vlan 10 
 
[SW1-G0/0/3]interface G0/0/4
[SW1-G0/0/4]port link-type access    
[SW1-G0/0/4]port default vlan 20  
[SW1-G0/0/4]quit

[SW1]interface eth-trunk 1   //创建并进入 eth-trunk 1
[SW1-Eth-Trunk1]mode manual load-balance   //配置手工模式
[SW1-Eth-Trunk1]trunkport g0/0/1   //加入成员端口 
[SW1-Eth-Trunk1]trunkport g0/0/2   //加入成员端口 
[SW1-Eth-Trunk1]port link-type trunk      //配置 eth-trunk 类型为 trunk
[SW1-Eth-Trunk1]port trunk allow-pass vlan 10 20   //允许所有的 vlan

SW2配置：
[SW2]vlan batch 10 20   
[SW2]interface G0/0/3 
[SW2-G0/0/3]port link-type access    
[SW2-G0/0/3]port default vlan 10  
[SW2-G0/0/3]interface G0/0/4
[SW2-G0/0/4]port link-type access    
[SW2-G0/0/4]port default vlan 20  

[SW2]interface eth-trunk 1
[SW2-Eth-Trunk1]mode manual load-balance
[SW2-Eth-Trunk1]trunkport g0/0/1
[SW2-Eth-Trunk1]trunkport g0/0/2
[SW2-Eth-Trunk1]port link-type trunk     
[SW2-Eth-Trunk1]port trunk allow-pass vlan 10 20  

验证与测试：
pc1 ping pc3  通
pc2 ping pc4  通
```
## LACP模式
- LACP模式也称M:N模式（M条活动链路，N条备份链路）
- 当活动链路故障时，备份链路才进行转发
![](/images/ethLACP模式拓扑.png)
### LACP模式工作原理
- 确定主动端
- 确定活动链路
- LACP抢占功能
### 确定LACP主动端
- 通过比较两端交换机的系统优先级来确定LACP主动端
- 系统优先级数值越小越优先，默认值时32768
- 如果系统优先级相同，则比较两端设备的MAC地址，越小越优先
### 确定LACP活动链路
- 通过系统优先级选举LACP主动端后，以主动端的接口优先级来选择活动接口
- 接口优先级数值越小越优先，默认值：32768
- 如果主动端设备的接口优先级相同，则根据接口号的大小来选举活动端口。（接口号越小越优先）
#### 主断了重连默认不抢占（VRRP默认抢占）
### 配置LACP模式Eth-trunk
#### 拓扑
![](/images/ethLACP模式配置拓扑.png)
#### 需求
- PC1和PC3属于vlan10，PC2和PC4属于vlan20
- 设备之间配置lacp模式的链路聚合，并确保同vlan之间的主机可以互通
#### 配置步骤
- PC配置IP地址
- 所有交换机都创建vlan10 20
- 交换机和PC互联的接口做成access，并且加入指定的vlan
- 设置交换机的lacp优先级，确定主动端设备
- 配置链路聚合
  - 创建链路聚合组，组号为1
  - 配置链路聚合的工作模式lacp
  - 在链路聚合中添加成员接口
  - 设置接口trunk模式
  - 设置活动端口（活动链路）的上限阈值为2
  - 开启lacp抢占，缺省情况下，优先级抢占处于禁止状态
#### 配置命令
```
SW1配置：
[SW1]vlan batch 10 20
[SW1]int g0/0/4
[SW1-GigabitEthernet0/0/4]port link-type access
[SW1-GigabitEthernet0/0/4]port default vlan 10
[SW1-GigabitEthernet0/0/4]int g0/0/5
[SW1-GigabitEthernet0/0/5]port link-type access
[SW1-GigabitEthernet0/0/5]port default vlan 20
[SW1-GigabitEthernet0/0/5]quit
[SW1]lacp priority 100    //配置lacp的系统优先级（越小越优先）
[SW1]interface eth-trunk 1   //创建链路聚合组1
[SW1-Eth-Trunk1]mode lacp-static   //链路聚合的工作模式是lacp
[SW1-Eth-Trunk1]trunkport  g 0/0/1  to  0/0/3   //在链路聚合组中添加成员接口
[SW1-Eth-Trunk1]port link-type trunk     //设置trunk模式
[SW1-Eth-Trunk1]port trunk allow-pass vlan 10 20    //允许vlan10 20 流量通过
[SW1-Eth-Trunk1]max active-linknumber 2   //设置活动端口的上限阈值为2  
[SW1-Eth-Trunk1]lacp preempt enable   //开启抢占功能

SW2配置：
[SW2]vlan batch  10  20
[SW2]int g0/0/4
[SW2-GigabitEthernet0/0/4]port link-type access
[SW2-GigabitEthernet0/0/4]port default vlan 10
[SW2-GigabitEthernet0/0/4]int g0/0/5
[SW2-GigabitEthernet0/0/5]port link-type access
[SW2-GigabitEthernet0/0/5]port default vlan 20
[SW2-GigabitEthernet0/0/5]quit
[SW2]int eth-trunk 1
[SW2-Eth-Trunk1]mode lacp-static 
[SW2-Eth-Trunk1]trunkport g 0/0/1 to 0/0/3
[SW2-Eth-Trunk1]max active-linknumber 2
[SW2-Eth-Trunk1]port link-type trunk
[SW2-Eth-Trunk1]port trunk allow-pass vlan 10 20
[SW2-Eth-Trunk1]lacp preempt enable 


测试与验证：
pc1 ping  pc3  通
pc1 ping  pc4  通

[SW1]display eth-trunk 1    //显示链路聚合信息                     
Preempt Delay Time: 30     //抢占延迟30秒        
System Priority: 100     //系统优先级：100                  
Least Active-linknumber: 1  Max Active-linknumber: 2 （最大活跃链路：2）                         
Operate status: up  （状态：up)                  
-------------------------------------------------------------------------
ActorPortName          Status   PortType PortPri  PortNo
GigabitEthernet0/0/1   Selected 1GE      32768    2  （lacp给本段接口分配的序号） 
GigabitEthernet0/0/2   Selected 1GE      32768    3     
GigabitEthernet0/0/3   Unselect 1GE      32768    4

备注：  Selected ：被选择的接口      
       Unselect ：未被选择的接口
       PortPri : 端口lacp优先级
       PortNo  ： lacp 协议给成员口分配的编号

Partner: (本段接口所连接的对端设备接口信息）
--------------------------------------------------------------------
ActorPortName          SysPri   SystemID        PortPri  PortNo
GigabitEthernet0/0/1   32768    4c1f-ccef-5a42  32768    4（lacp给对端接口分配的序号）
GigabitEthernet0/0/2   32768    4c1f-ccef-5a42  32768    5 
GigabitEthernet0/0/3   32768    4c1f-ccef-5a42  32768    6

显示eth-trunk 1 详细信息
[SW1]display eth-trunk 1    //显示链路聚合信息                     
Preempt Delay Time: 30     //抢占延迟30秒        
System Priority: 100     //系统优先级：100                  
Least Active-linknumber: 1  Max Active-linknumber: 2 （最大活跃链路：2）                         
Operate status: up  （状态：up)                  
------------------------------------------------------------------------
ActorPortName          Status   PortType PortPri  PortNo
GigabitEthernet0/0/1   Selected 1GE      32768    2  
GigabitEthernet0/0/2   Selected 1GE      32768    3     
GigabitEthernet0/0/3   Unselect 1GE      32768    4

备注：  Selected ：被选择的接口      
       Unselect ：未被选择的接口
       PortPri : 端口lacp优先级
       PortNo  ： lacp 协议给成员口分配的编号

Partner: (本端接口所连接的对端设备接口信息）
-------------------------------------------------------------------------
ActorPortName          SysPri   SystemID        PortPri  PortNo
GigabitEthernet0/0/1   32768    4c1f-ccef-5a42  32768    4 
GigabitEthernet0/0/2   32768    4c1f-ccef-5a42  32768    5 
GigabitEthernet0/0/3   32768    4c1f-ccef-5a42  32768    6
```
#### 手工模式与LACP模式的区别
- LACP可以设置活动链路和非活动链路
- 检测动态链路故障（比如一条聚合链路的两条线接到了不同的设备上）在手工模式中会显示正常，而在LACP模式中会显示unselect
## CE交换机中动态LACP模式
### 拓扑
![](/images/eth动态LACP模式配置拓扑.png)
### 命令
```
CE1配置：
interface Eth-Trunk1
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
 mode lacp-dynamic
#
interface GE1/0/0
 undo shutdown
 eth-trunk 1
#
interface GE1/0/1
 undo shutdown
 eth-trunk 1
 #
interface GE1/0/2
 undo shutdown
 port default vlan 10
CE2配置
#
interface Eth-Trunk1
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
#
interface GE1/0/0
 undo shutdown
 eth-trunk 1
#
interface GE1/0/1
 undo shutdown
 eth-trunk 1
#
interface GE1/0/2
 undo shutdown
 port default vlan 10
情况（一）分析：如果CE1配置了动态LACP,CE2配置了手动负载分担，ping测试可以与PC2互通这是与静态LACP区别的地方，如果静态LACP两端模式协商不一致时（一端静态LACP,一端手动），聚合口down的情况下，物理口不会转发二层流量；
如果是动态lacp（一端动态lacp，一端手动）在聚合口down的情况下，物理口依然可以转发流量
```
![](/images/eth动态LACP查验截图1.png)
![](/images/eth动态LACP查验截图2.png)
![](/images/eth动态LACP查验截图3.png)
```
情况（二）分析：如果CE1配置了动态LACP,CE2配置了静态LACP,两端依然可以互相访问，此时聚合口是up的
从状态上来看，相当于都是静态LACP
CE1配置：
#
interface Eth-Trunk1
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
 mode lacp-dynamic
CE2配置：
#
interface Eth-Trunk1
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
 mode lacp-static
```
![](/images/eth动态LACP查验截图4.png)
![](/images/eth动态LACP查验截图5.png)
场景：动态LACP只用于与服务器互联的场景，其他场景都用静态LACP，如果在其他场景使用动态LACP，在聚合口没有协商成功时，物理口仍然能转发二层流量，会产生环路风险

**为什么与服务器互联时需要使用动态LACP**

**可以理解为一种优化：如果是静态LACP当服务器重启时，静态LACP协商失败，流量会中断，如果是动态LACP即使LACP没有协商成功，只要物理口up，流量就会正常转发流量，减少业务中断时间**
# WLAN
## 概念
WLAN无线局域网的简称，通过无线电波进行传输数据，使用802.11协议。
- 目前现网中用得比较多的协议标准是：
  - 802.11n（2.4G和5G）
  - 802.11ac协议，称之为wifi5（1236供电、4578数据）
  - 802.11ax协议，称之为wifi6（万兆，1236数据、4578供电）
## 场景和特点
### 场景
![](/images/WLAN场景.png)
### 特点
![](/images/WLAN特点.png)
## 术语
- AP设备
  - 接入点，就是无线交换机，用来连接手机和电脑的交换机（类似于家里的无线路由器）
  - 分类：室内AP（面板式和吸顶式）室外AP
  - 胖AP：一般用于家庭（不需要AC也能配置很多东西）
  - 瘦AP（由POE交换机统一供电，通过网线）用于组网，所有配置靠AC推送
  - 云AP
- POE交换机：可以给无线AP通过网线提供电力支持，不需要给AP单独配电源
- AC控制器：对无线AP进行统一管理的，下发配置（无线路由器+无线防火墙+上网行为管理）
- 工作站STA：支持无线联网的终端设备（笔记本，手机，连接无线网卡的台式电脑）
- 射频信号：可以远距离传输的无线电波，目前主用的是2.4G或5G频段的电磁波
- SSID：服务及标识符（无线网的名字）
## 组网方式
- 隧道转发：AP的管理流量和业务流量都经过AC控制器，AC容易成为瓶颈，不利于网络的拓展（但是考试考这个）
  ![](/images/WLAN组网方式1.png)
- 直接转发：AP和管理流量经过AC，终端的业务流量不经过AC，常用该种网络架构，便于扩容，适用于较大网络规模，减少AC性能的占用
  ![](/images/WLAN组网方式2.png)
## VLAN Pool
### 现网面临的挑战
- 在无线网络环境下，使用手机或者平板上网的人很多，无线网络下，用户移动性增强，可能导致某一区域IP地址请求过多的情况
- 通过情况下，一个SSID只能对应一个业务VLAN，通常一个VLAN对应一个C类地址池，只有253个可以分发的IP地址，如果在同一个VLAN内，扩大地址池范围，会导致广播报文增多
- VLAN Pool：就是VLAN池，就是把多个VLAN放在一个VLAN池子中
### VLAN pool的优势
- VLAN pool配置为无线用户的业务VLAN，实现一个SSID能够同时支持多个业务VLAN。
- 新接入的用户被动态的分配到VLAN Pool中的各个VLAN中，减少了单个VLAN下的用户数，缩小了广播域；同时每个VLAN尽量均匀的分配IP地址，减少了IP地址的浪费
  ![](/images/WLANVLANPool.png)
## 配置
### 拓扑
  ![](/images/WLAN配置拓扑.png)
### 需求
- 有线主机PC1能访问AR1
- 无线主机STA2、STA3访问AR1，分别使用隧道转发模式和直接转发模式实现
### 配置步骤
- AC配置
  - AP上线
    - AP获取地址（AC作为AP的DHCP server）
  - 下发业务模板（下发配置）
    - SSID模板（无线信号名字）
    - 安全模板（密码）
    - vap模板
### 命令
```
隧道转发模式：
SW1配置
vlan batch  100
#
interface GigabitEthernet0/0/1
 port link-type trunk
 port trunk pvid vlan 100
 port trunk allow-pass vlan 100
#
interface GigabitEthernet0/0/2
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
----------------------------------------------------------------------------
SW3配置
vlan batch 10 20 30 100 255
#
dhcp enable
#
ip pool 1
 gateway-list 192.168.10.254
 network 192.168.10.0 mask 255.255.255.0
 dns-list 8.8.8.8
#
ip pool 2
 gateway-list 192.168.20.254
 network 192.168.20.0 mask 255.255.255.0
 dns-list 8.8.8.8
#
interface GigabitEthernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
#
interface GigabitEthernet0/0/2
 port link-type trunk
 port trunk allow-pass vlan 30
#
interface GigabitEthernet0/0/3
 port link-type trunk
 port trunk allow-pass vlan 10 20 100
#
interface Vlanif10
 ip address 192.168.10.254 255.255.255.0
 dhcp select global
#
interface Vlanif20
 ip address 192.168.20.254 255.255.255.0
 dhcp select global
#
interface Vlanif30
 ip address 192.168.30.254 255.255.255.0
#
interface Vlanif255
 ip address 192.168.255.1 255.255.255.0
------------------------------------------------------------------------
AC配置
#
vlan batch 10 20 100
#
vlan pool a
 vlan 10 20
#
dhcp enable
#
interface Vlanif100
 ip address 192.168.100.254 255.255.255.0
 dhcp select interface
#
interface GigabitEthernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 100
wlan
  ssid-profile name tedu
   ssid tedu
  security-profile name tedu
   security wpa-wpa2 psk pass-phrase Huawei2#$ aes
  vap-profile name tedu
   forward-mode tunnel    //默认直接转发，修改转发模式为隧道转发
   service-vlan vlan-pool a
   ssid-profile tedu
   security-profile tedu
  ap-group name guest
  ap-id 0 type-id 69 ap-mac 00e0-fcb7-2890 ap-sn 2102354483109C565D27
   ap-name guest
   ap-group guest
   radio 0
    vap-profile tedu wlan 1
   radio 1
    vap-profile tedu wlan 1
   radio 2
    vap-profile tedu wlan 1

```
### 隧道传递中的封装过程
- 从终端向AR1转发（从内到外标签封装顺序）
  1. icmp
  2. 源IP（终端IP）目的IP（AR1IP）
  3. 业务vlan
  4. 源MAC（终端AC）目的AC（网关MAC）
  5. capwap
  6. UDP
  7. 源IP（APIP）目的IP（ACIP）
  8. 管理vlan
  9. 源MAC（APMAC）目的MAC（ACMAC）
- 从AR1向终端转发（从内到外标签封装顺序）
  1. icmp
  2. 源IP（AR1IP）目的IP（终端IP）
  3. 业务vlan
  4. 源MAC（网关MAC）目的AC（终端AC）
  5. capwap
  6. UDP
  7. 源IP（ACIP）目的IP（APIP）
  8. 管理vlan
  9. 源MAC（ACMAC）目的MAC（APMAC）
### 几种可能不上线的情况（现网）
- capwap源地址没指定
- 没配置mac认证
- ap模式不一致
- ap，ac的版本不一致