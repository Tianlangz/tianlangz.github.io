---
author: zty
title: Security
date: 2024-03-21
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
- 以太网安全
- 防火前高级特性
<!--more-->
# 以太网安全
## 端口隔离
- 场景
  - 同一vlan中，相同网段主机不允许访问
- 原理
  - 不能互访的接口加入到同一个隔离组，同一个隔离组内的主机不能实现互访，不同隔离组的主机可以互访
### 隔离模式
#### 概念
- 2层隔离
  - 隔离组内主机二层不能访问，不同隔离组中能访问
  - 默认的端口隔离是二层的
- 2、3层隔离
  - 配置了二层隔离的主机，如果三层网关配置了vlan内代理，三层可以实现互访，因此可配置2、3层都隔离
### 隔离类型
#### 双向隔离
隔离组内的主机双向都不能互访
#### 单项隔离
主机A的流量能到达B，B的流量不能到达A
## MAC安全
### MAC地址表安全
- 场景

|场景|技术|
|-|-|
|1.防止MAC地址表篡改，保护重要资源不被攻击|静态MAC，不老化|
|2.防止非法主机接入|黑洞MAC，源目MAC是黑洞MAC会丢弃，不老化|
|3.防止MAC地址表暴增|缩短MAC地址表老化时间|
|4.限制接口学习非法主机MAC|禁止端口MAC地址学习|
|5.防止接入端口MAC地址泛洪攻击|限制端口MAC地址学习数量|
  1. 保护服务器资源，防止攻击者PC1篡改MAC地址表，将地址表中的服务器的MAC地址改成自己的MAC，从而获取网络中的合法流量
    - 解决方案：通过服务器的静态MAC和端口绑定，静态MAC优先动态MAC
    
    mac-address static MAC地址 端口号 vlan vlan号 
  2. PC2主机是非法主机，不允许其他主机访问
    - 解决方案：配置黑洞MAC，不管匹配到源/目的MAC。都会丢弃
    
    mac-address blackhole MAC地址 vlan vlan号
  3. 防止设备学习到非法的MAC地址，错误的修改MAC地址表中的原MAC地址表项
    - 解决方案：配置接口不学习MAC地址，默认不学习MAC，但是流量会转发

    命令
    interface 端口号
    mac-address learning disable
    可设置参数：不学习mac，不转发流量
    [sw1-GigabitEthernet0/0/3]mac-address learning disable action discard 
  4. 限制MAC地址学习数量

    interface 端口号
    mac-limit maximum 2
  5. 修改MAC地址表老化时间，默认300s（现网中不建议去调整）

    [sw1]mac-address aging-time 50
- MAC地址分类
  - 静态MAC地址
  - 动态MAC地址
  - 黑洞MAC地址
### 端口安全
#### 场景
- 限制MAC地址数量，防止非法主机加入

  （与MAC地址表中的区别，MAC地址表限制MAC，侧重点是防止MAC地址泛洪攻击，默认`没有MAC地址也转发流量`;端口安全，是限制MAC地址进入，侧重点是防止非法主机接入网络，默认动作是`丢弃MAC，不转发流量`）
- 固定主机和端口位置

  解释：就像是一个饭店只允许十个人进店消费，多余的人来了我们也不招待，那么这十个人就是安全MAC
#### 安全MAC类型
- 安全动态MAC
  - 作用：将动态MAC自动转换成安全动态MAC，防止非法MAC地址接入，会重启表项丢失，默认不老化
  - 适用范围：适用于接入用户不固定的场景
  - 问题：当交换机重启以后，表项就会丢失，此时如果非法主机抢先接入网络，（他先进入了饭店）抢到了前十的名额，合法主机就不能接入
  ```
  interface GigabitEthernet0/0/2
  port-security enable /当接口开启端口安全MAC，默认接口下只能学习到1条MAC
  #
  [Huawei]display mac-address
  MAC address table of slot 0:
  -------------------------------------------------------------------
  MAC Address    VLAN/       PEVLAN CEVLAN Port      Type   LSP/LSR-ID  
               VSI/SI                                   MAC-Tunnel  
  -------------------------------------------------------------------
  5489-981d-6861 1           -      -      GE0/0/2      security  -           
  -------------------------------------------------------------------
  Total matching items on slot 0 displayed = 1 

  #
  interface GigabitEthernet0/0/2
  port-security enable 
  port-security max-mac-num 2  /修改安全MAC的数量2

  [Huawei]display  mac-address 
  MAC address table of slot 0:
  ------------------------------------------------------------------
  MAC Address    VLAN/       PEVLAN CEVLAN Port    Type  LSP/LSR-ID  
                 VSI/SI                                   MAC-Tunnel  
  -----------------------------------------------------------------
  5489-986b-1912 1           -      -      GE0/0/2        security            
  5489-981d-6861 1           -      -      GE0/0/2        security           
  ------------------------------------------------------------------
  Total matching items on slot 0 displayed = 2 
  ```
  安全动态MAC在接口shutdown，或者设备重启后，安全MAC表项会消失，因此需要转换为黏性MAC
- Sticky MAC
  - 作用：作用都一样，特点是重启设备，接口down后不消失，不老化
  - 适用范围：适用于大量固定用户场景，自动地将动态安全MAC转换成Sticky MAC
  ```
  #
  interface GigabitEthernet0/0/2
  port-security enable
  port-security max-mac-num 2
  port-security mac-address sticky
  #
  [Huawei]display  mac-address 
  MAC address table of slot 0:
  ----------------------------------------------------------------------
  MAC Address    VLAN/       PEVLAN CEVLAN Port       Type    LSP/LSR-ID  
                VSI/SI                                       MAC-Tunnel  
  ---------------------------------------------------------------------
  5489-986b-1912 1           -      -      GE0/0/2   sticky    -           
  5489-981d-6861 1           -      -      GE0/0/2   sticky    -           
  ----------------------------------------------------------------------
  Total matching items on slot 0 displayed = 2 
  当我需要增加少量安全黏性MAC时，可选择手动增加
  #
  interface GigabitEthernet0/0/2
  port-security enable
  port-security max-mac-num 3    /在原来基础上，手动增加一条黏性MAC
  port-security mac-address sticky
  port-security mac-address sticky 5489-981D-6861 vlan 1

  [Huawei]display  mac-address 
  MAC address table of slot 0:
  ----------------------------------------------------------------
  MAC Address    VLAN/       PEVLAN CEVLAN Port   Type    LSP/LSR-ID  
                VSI/SI                                  MAC-Tunnel  
  ---------------------------------------------------------------
  5489-981a-40f9 1           -      -      GE0/0/2       sticky          
  5489-986b-1912 1           -      -      GE0/0/2       sticky           
  5489-981d-6861 1           -      -      GE0/0/2       sticky           
  ------------------------------------------------------------  
  ```
- 安全静态MAC
  - 作用：作用都一样，特点是重启设备不消失，不老化（不常用，选配）
  - 适用范围：适用于少量固定用户场景，手动的将动态MAC配置成安全静态MAC
  ```
  interface GigabitEthernet0/0/2
  port-security mac-address 5489-986b-1912 vlan 1    /ENSP不支持 
  ```
- 静态MAC、动态MAC与安全MAC的区别:
  1. 静态，动态MAC没有安全动态MAC学习的限制
  2. 当配置安全动态MAC后，触发访问时动态MAC可以转换为安全动态MAC
  3. 安全静态MAC，解决的是重启设备动态MAC不丢失而已
#### 安全保护动作
- Restrict：丢弃源MAC地址不存在的报文并上报告警，默认动作
- Protect：只丢弃源MAC地址不存在的报文，不上报告警
- Shutdown：接口状态被置为error-down，并上报告警
### MAC地址漂移
- 概念：同一个MAC地址在不同的接口上翻动
- 场景：环路或者网络攻击
- 解决方案
  - 环路
    - 环路确认方法：`<sw1>display mac-address flapping record`
    - 环回解决方案：确认成环的端口
      - 限制端口优先级
      - 不允许相同优先级接口MAC地址漂移
      - 拔掉 
  - 网络攻击
    ![](/images/MAC地址漂移网络攻击解决办法.png)
### MACsec
- 概念：以太网链路提供设备到设备的安全保护，类似于IPSEC
- 场景：应用在内网同一网段中逐跳为数据帧提供保护，IPSEC是用于站点和站点之间，保护三层数据报文安全
- 原理
  - 概念
    - MKA（MACsec Key Agreement）密钥协定，作用是定义了复杂的密钥生成体系，确保MACsec数据传输的安全性
    - CAK（secure Connectivity Association Key）安全连接关联密钥，使用户在设备上配置的密钥，不直接用于数据报文的加密
    - CKN（secure Connectivity Association Key Name）安全连接关联密钥名称：CAK的名称，必须在两端设备的接口上配置相同的CKN
    - SAK（Secure Association Key）安全关联密钥，对数据加密
    - ICV（Integrity Check Value）完整性校验值：发送端根据报文计算生成ICV放在报文尾部，报文接收端使用相同的算法计算得到ICV与报文携带的ICV进行比对。如果这两个ICV相同，说明报文没有被修改，校验通过；否则认为报文被修改，丢弃该报文
  - 原理
    - 两台设备配置相同的CKA、CKN，并通过优先级选举密钥服务器
    - 密钥服务器根据CAK生成用于加密数据报文的SAK，分发给对端设备
    - 对端设备按使用SAK解密
  ![](/images/MACsec密钥派生关系图.png)
```
配置：
CE1:
mac-security-profile name test1
 mka keyserver priority 1
 macsec mode normal
#
interface 100GE1/0/1
 mac-security-profile test1
 mka cak-mode static ckn ****** cka ********
CE2:
mac-security-profile name test2
 mka keyserver priority 2
 macsec mode normal
#
interface 100GE1/0/1
 mac-security-profile test2
 mka cak-mode static ckn ****** cka ********
```
## 交换机流量控制
- 场景：防止交换机收到BUM帧泛洪流量过大（Broadcast,Unknown unicast,Multicast广播、未知单播及组播帧），影响已知单播流量转发
- 防范原理：给BUM流量配置阈值，超过阈值直接丢弃
- 配置：
  ![](/images/交换机流量控制精准配置.png)
- 风暴控制
  - 其实还是BUM帧泛洪场景，只不过成环了，造成广播风暴，解决办法是对超出阈值的流量，配置一个接口动作，设置成关闭或阻塞接口，来防止广播风暴
## DHCP Snopping
- 概念：DHCP嗅探，用于在DHCP Client和DHCP Server 之间保护DHCP的防止各种攻击
- 作用
  - 信任DHCP服务器-->防范仿冒DHCP攻击
  - 监听DHCP报文，并生成DHCP绑定表（IP、MAC、vlan、端口、租期信息）-->防范DHCP饿死和中间人攻击
### 攻击类型及防范
1. 仿冒DHCP - DHCP信任功能（企业现网中存在多DHCP服务器场景，不想攻击的攻击）
  ![](/images/仿冒DHCP防范.png)
  - 命令
  ```
  sysname produce
  #
  vlan batch 10
  #
  dhcp enable
  #
  dhcp snooping enable
  #
  vlan 1
  description Manager
  vlan 10
  description caiwubu
  #
  interface GigabitEthernet0/0/1
  port link-type access
  port default vlan 10
  dhcp snooping enable
  #
  interface GigabitEthernet0/0/2
  port link-type access
  port default vlan 10
  dhcp snooping enable
  #
  interface GigabitEthernet0/0/24
  port link-type trunk
  port trunk allow-pass vlan 2 to 4094
  dhcp snooping enable
  dhcp snooping trusted

  ```
2. DHCP饿死攻击 - DHCP监听
- DHCP饿死攻击（小卡拉米）：攻击者变化MAC地址，获取大量IP地址，导致正常主机没有地址可用
![](/images/限制学习MAC地址数量.png)
解决方案：限制学习MAC地址数量
- DHCP饿死攻击-PLUS
  - 攻击者伪造DHCP Request报文中CHADDR字段，因为DHCP服务器是根据CHADDR字段分配的IP，该字段不断变化，获取大量IP，导致正常主机饿死
  - 解决方案：对报文的源MAC地址与CHADDR中的字段进行对比检查，一致通过
```
命令：
dhcp enable
#
dhcp snooping enable
#
dhcp snooping check dhcp-chaddr enable     //开启检查功能
```
3. 中间人攻击- DHCP监听

  攻击本质就是：攻击者用自己的MAC欺骗了主机和服务器，让主机和服务器在转发数据帧的时候通过MAC地址表，错误地将流量转发到攻击者主机
  ![](/images/中间人攻击后流量走向.png)
  
  解决方案：使用DHCP Snooping的绑定表工作模式，当接口接收到ARP报文，使用ARP报文中的“源IP+源MAC”匹配DHCP Snooping绑定表。如果匹配就进行转发，如果不匹配就丢弃
  ```
  命令：
  dhcp enable
  #
  dhcp snooping enable
  #
  arp dhcp-snooping-detect enable   //开启ARP与DHCP绑定表联动功能 
  ```
  ![](/images/使用DHCP snooping绑定表.png)
## IP Source Guard（保护）
### 场景
- 防止内部人员私自更改IP地址，防止外来人员接入内网，适用于小规模网络
### 原理
- 静态绑定表：主机手动静态绑定IP和MAC地址，只有匹配绑定的报文才能转发
- 动态绑定表：通过DHCP snooping自动生成
### 配置
- 终端静态地址场景：
  ```
  第一步：系统视图下
  user-bind static ip-address 192.168.2.50  mac-address 70f3-9515-9ca1
  user-bind static ip-address 192.168.2.51  mac-address  ecf4-bb13-b846 

  第二步：进入物理接口
  [sw]interface g0/0/2   进入物理接口下  
      ip source check user-bind enable    
  注意：使能该命令后,192.168.2.0/24的用户如果没有绑定第一步，都不能上网

  ```
- 终端DHCP场景
  ![](/images/终端DHCP场景拓扑.png)
  ```
  SW2配置
  作为网关设备，配置DHCP server服务器
  #
  ip pool 1
  gateway-list 192.168.1.254
  network 192.168.1.0 mask 255.255.255.0
  #
  interface Vlanif1
  ip address 192.168.1.254 255.255.255.0
  dhcp select global
  #
  SW3配置
  开启DHCP Snooping
  #
  dhcp enable
  #
  dhcp snooping enable
  #
  interface GigabitEthernet0/0/1
  dhcp snooping enable
  #
  interface GigabitEthernet0/0/2
  dhcp snooping enable
  #
  interface GigabitEthernet0/0/3
  dhcp snooping enable
  #
  interface GigabitEthernet0/0/4
  dhcp snooping enable
  #
  interface GigabitEthernet0/0/5
  dhcp snooping trusted
  ------------------------------------------------------------------
  当终端PC1获取地址时，SW3会自动生成DHCP Snooping绑定表
  <sw3>display  dhcp snooping user-bind all 
  DHCP Dynamic Bind-table:
  Flags:O - outer vlan ,I - inner vlan ,P - map vlan 
  IP Address       MAC Address     VSI/VLAN(O/I/P) Interface      Lease           

  --------------------------------------------------------------------------------
  192.168.1.248    5489-9855-6c73  1   /--  /--    GE0/0/1        2023.10.30-20:20
  --------------------------------------------------------------------------------
  print count:           1          total count:  
  此时如果PC3修改IP地址为192.168.1.248则无法访问网络，因为与dhcp snooping绑定表不匹配  
  ```
# 防火墙高级特性
## 基础
- 加zone
- 域间策略
  - 单向策略，回包匹配会话
  - 把策略全打开：`default action permit`
- 策略路由
- 首包检测机制
  - 来回路径不一致场景
- 华为FW处理流程
  - 顺序依次为
    - nat-server
    - 路由
    - 安全策略
    - 源nat
    - IPsec vpn
### 密码
- 防火墙：Admin@123
- 交换机路由器：admin@huawei.com
## 双机热备
### 场景
  - 用在高可靠场景，当一台FW故障，业务可以切换到另一台，不影响业务转发
### 原理
  - VRRP备份组之间单独管理，不能联动，使用VGMP协议，管理VRRP协议只要一个VRRP组故障，设备就能进行主备切换
  - 使用HRP协议，备份配置和表象，具体备份的配置。型号：USG6000E
  ![](/images/适用的机器，能找到的文档.png)
### 组网
  - 主备模式：主转发流量，备不转发（常用的模式）
  负载分担模式：主备都转发流量
### 配置
- 拓扑
  ![](/images/防火墙热备.png)
- 需求
  - PC2访问PC1，主备场景，当主链路故障，业务可以实现快速切换
- 配置
  ```
  FW1:
  第一步：配置接口（IP地址）
  第二步：接口加入zone
  第三步：放行安全策略（主）
  第四步：配置双机
  第五步：手动同步配置
  第六步：一致性检查
  第七部：主备倒换
  FW1配置：
  interface GigabitEthernet1/0/0
  undo shutdown
  ip address 192.168.1.253 255.255.255.0
  vrrp vrid 1 virtual-ip 192.168.1.254 active
  #
  interface GigabitEthernet1/0/1
  undo shutdown
  ip address 1.1.1.1 255.255.255.252
  service-manage http permit
  service-manage https permit
  service-manage ping permit
  service-manage ssh permit
  service-manage snmp permit
  service-manage telnet permit
  #
  interface GigabitEthernet1/0/2
  undo shutdown
  ip address 192.168.2.253 255.255.255.0
  vrrp vrid 2 virtual-ip 192.168.2.254 active
  #
  firewall zone trust
  set priority 85
  add interface GigabitEthernet0/0/0
  add interface GigabitEthernet1/0/0
  #
  firewall zone untrust
  set priority 5
  add interface GigabitEthernet1/0/2
  #
  firewall zone dmz
  set priority 50
  add interface GigabitEthernet1/0/1
  #
  security-policy
  default action permit
  rule name hrp
    source-zone dmz
    source-zone local
    destination-zone dmz
    destination-zone local
    action permit
  rule name t_u
    source-zone trust
    destination-zone untrust
    source-address 192.168.1.0 mask 255.255.255.0
    destination-address 192.168.2.0 mask 255.255.255.0
    action permit
  #
  hrp enable
  #
  hrp interface GigabitEthernet1/0/1 remote 1.1.1.2
  FW2配置：
  interface GigabitEthernet1/0/0
  undo shutdown
  ip address 192.168.1.252 255.255.255.0
  vrrp vrid 1 virtual-ip 192.168.1.254 standby
  #
  interface GigabitEthernet1/0/1
  undo shutdown
  ip address 1.1.1.2 255.255.255.252
  service-manage http permit
  service-manage https permit
  service-manage ping permit
  service-manage ssh permit
  service-manage snmp permit
  service-manage telnet permit
  #
  interface GigabitEthernet1/0/2
  undo shutdown
  ip address 192.168.2.252 255.255.255.0
  vrrp vrid 2 virtual-ip 192.168.2.254 standby
  #
  firewall zone trust
  set priority 85
  add interface GigabitEthernet0/0/0
  add interface GigabitEthernet1/0/0
  #
  firewall zone untrust
  set priority 5
  add interface GigabitEthernet1/0/2
  #
  firewall zone dmz
  set priority 50
  add interface GigabitEthernet1/0/1
  #
  security-policy
  default action permit
  rule name hrp
    source-zone dmz
    source-zone local
    destination-zone dmz
    destination-zone local
    action permit
  rule name t_u
    source-zone trust
    destination-zone untrust
    source-address 192.168.1.0 mask 255.255.255.0
    destination-address 192.168.2.0 mask 255.255.255.0
    action permit
  #
  hrp enable
  hrp interface GigabitEthernet1/0/1 remote 1.1.1.1
  =================================================================
  手工同步配置：HRP_M<FW1>hrp sync config
  一致性检查：HRP_M[FW1]hrp configuration check all
  HRP_M[FW1]display  hrp configuration check  all
  2023-08-12 09:13:34.970 
  Module           State  Start-time          End-time        Result
  all              finish 2023/08/12 09:13:19 2023/08/12 Same Configurat
  ion
  主备切换测试：HRP_M[FW1]undo hrp switch standby
  备份静态路由：默认不备份 hrp auto-sync config static-route  
  hrp目的端口：18514
  ```
## 虚拟系统
### 场景
- 用在多部门业务隔离场景，相当于VRF
### 原理
![](/images/防火墙虚拟系统原理图像.png)
将一台物理防火墙，分成多个虚拟防火墙，每一台虚拟防火墙业务隔离，虚拟防火墙与物理防火墙，以及虚拟防火墙与虚拟防火墙之间通过虚拟接口连接实现
### 配置
- 拓扑
  ![](/images/虚拟防火墙拓扑.png)
- 需求
  - 使用虚拟系统实现
    1. 部门A主机可以访问Internet
    2. 部门A可以和部门C互访
- 配置步骤
  - 创建虚拟系统
    - 分配接口
  - 进入虚拟系统a
    - 配置接口
    - 接口加zone
    - 配置到public的路由
    - 放行a-public安全策略
    - 放行a-b安全策略
  - 进入public系统
    - 配置接口，加zone
    - 配置虚拟系统间的路由
    - 配置访问互联网缺省路由
    - 配置返回虚拟系统a路由
    - 放行安全策略
    - 配置源地址转换
  - 进入虚拟系统b
    - 配置接口
    - 接口加zone
    - 放行a-b安全策略
```
FW配置：
1.创建虚拟系统：
vsys enable
#
vsys name vsysa 1
 assign interface GigabitEthernet1/0/0
#
vsys name vsysb 2
 assign interface GigabitEthernet1/0/1
2.进入虚拟系统vsysa
[FW]switch vsys vsysa
interface GigabitEthernet1/0/0
 ip vpninstance a  vsys
 ip address 10.1.1.254 255.255.255.0
#
firewall zone trust
 set priority 85
 add interface GigabitEthernet1/0/0
#
firewall zone untrust
 set priority 5
 add interface Virtual-if1
#
security-policy
 rule name vsysa_vsysb
  source-zone trust
  destination-zone untrust
  source-address 10.1.1.0 mask 255.255.255.0
  destination-address 10.1.2.0 mask 255.255.255.0
  service icmp
  action permit
 rule name vsysa_public
  source-zone trust
  destination-zone untrust
  source-address 10.1.1.0 mask 255.255.255.0
  action permit
#
ip route-static 0.0.0.0 0.0.0.0 public
3.退回到Public
interface GigabitEthernet1/0/2
 ip address 10.1.12.1 255.255.255.0
#
firewall zone trust
 set priority 85
 add interface Virtual-if0
#
firewall zone untrust
 set priority 5
 add interface GigabitEthernet1/0/2
#
security-policy
 rule name inter
  source-zone trust
  destination-zone untrust
  source-address 10.1.1.0 mask 255.255.255.0
  action permit
#
nat-policy
 rule name s_nat
  source-zone trust
  destination-zone untrust
  source-address 10.1.1.0 mask 255.255.255.0
  action source-nat easy-ip
#
ip route-static 0.0.0.0 0.0.0.0 10.1.12.2
ip route-static 10.1.1.0 255.255.255.0 vpn-instance vsysa
ip route-static vpn-instance vsysa 10.1.2.0 24 vpn-instance vsysb
ip route-static vpn-instance vsysb 10.1.1.0 24 vpn-instance vsysa
4.进入虚拟系统vsysb
#
interface GigabitEthernet1/0/1
 ip address 10.1.2.254 255.255.255.0
#
 firewall zone trust
 set priority 85
 add interface Virtual-if2
#
firewall zone untrust
 set priority 5
 add interface GigabitEthernet1/0/1
#
security-policy
 rule name vsysa_vsysb
  source-zone trust
  destination-zone untrust
  source-address 10.1.1.0 mask 255.255.255.0
  destination-address 10.1.2.0 mask 255.255.255.0
  service icmp
  action permit
```