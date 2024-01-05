---
author: Hugo Authors
title: linux基础学习
date: 2023-12-29
description: Linux系统编程
series:
  - Linux系统编程

---
```
```
<!--more-->
# Linux中根目录下的常见目录：
  - /：根目录
  - /home：家目录，一般存放普通用户的家
  - /root：超级用户root的家/主目录
  - /var：一般存放经常变化的文档比如日志
  - /dev：一般存放设备

  ```
  [root@localhost ~]#
  root：用户名（当前登录的用户）
  @：分隔符
  localhost：主机名
  ~所在的未知：代表当前的工作目录
  ~：用户的家目录
  ```
# 命令
  - `P`rint `W`orking `D`irectory
    - 显示当前工作目录
    - pwd
  - `C`hange `D`irectory
    - 改变工作目录
    - cd 目录路径
  - `L`i`s`t
    - 列出目录内容/文档权限
    - ls [选项]……[工作路径]
    - 常用命令选项
      - -A：列出目录内文档（包括.开头的隐藏文档）
      - -a：列出目录内所有文档（包括隐藏文档及`.`和`..`）
      - -l：以长格式显示（详细信息）
      - -h：带容量单位（一般与-l一起使用）
      - -d：如果目标是目录，只列出目录本身（不包括目录下内容）
      ![](/images/ls输出详细内容详解.png)
      具体内容详见：https://blog.csdn.net/tsummer2010/article/details/104438878
  - useradd
    - 创建用户
    - useradd 用户名
  - passwd
    - 为用户设置密码
    - passwd 用户名
  - userdel
    - 格式：userdel[-r]用户名
    - 添加-r选项时，可以将宿主目录/用户邮箱也一并删除
  - `S`ubstitute `U`ser
    - 替换为新用户
    - su - 用户名（`-`代表切换用户时进入到用户的家目录）
    - 禁止滥用su切换权限
      - 通常情况下，任何用户都被允许使用su切换
      - 但openeuler中启用了wheel组限制
        - 除了root以外，只有加入wheel组的用户才被允许使用su切换
        - 其他人使用su切换时，会提醒拒绝权限
    ```
    gpasswd -a student wheel      //将student用户加入到wheel组
    ```
  - exit
    - 退出当前的命令行环境
    - exit
  - route -n
    - 查看网关（以数字方式列出路由表）
  - cat：显示文件的全部内容
    - 格式：cat[文件路径]
    ```
    cat /etc/resolv.conf————显示DNS记录文件的内容
    ```
  - less：分页显示
    - 使用PgUp/PgDn翻页、按q键退出
  - ifconfig
    - 查看IP地址参数
  - hostname
    - 查看主机名
  - nmtui
    - 更改IP地址
    - 更改主机名
    - 更改网关
    - 更改DNS
    - 更改网卡名称
    - 结束之后不要忘记重启网卡
  - mkdir
    - 创建目录
    - 格式：mkdir[-p][/路径/]目录名……
    - \-p：创建目录时连同父目录一起创建
  - cp（copy）复制文档
    - 格式：cp[路径]源文件，目标路径
    - 常用命令选项
      - \-r：递归，复制目录时必须有此选项
  - mv（move）移动/改名文档
    -格式：mv[选项]原文档目标路径/[新文档名]
  - rm（remove）删除文档
    - 格式：rm[选项]文件或目录
    - 常用命令选项
      - -r：递归删除（含目录）
      - -f：强制删除

# 组
  - 组的作用
    - 用来批量授权
    - 作为访问文档、进程等资源的身份凭证
  - 组账号的主要属性
    - 组名、成员用户列表
  - groupadd命令
    - 格式：groupadd组名
    ```
    [root@svr223 ~]# groupadd gaibang
    [root@svr223 ~]# cat /etc/group
    ```
  - gpasswdmingling 
    - 格式：gpasswd[-a|-d用户名]组名
    - -a（add）：添加
    - -d（delete）：删除
    ```
    [root@svr223 ~]# useradd nvshen       //添加nvshen用户
    [root@svr223 ~]# gpasswd -a nvshen gaibang    //将nvshen用户添加到gaibang组
    正在将用户“nvshen”加入到“gaibang”组中
    [root@svr223 ~]# id nvshen        //确认结果
    用户id=1002(nvshen) 组id=1003(nvshen) 组=1003(nvshen),1002(gaibang)
    ```
  - groupdel命令
    - 格式：groupdel组名
    ```
    [root@svr223 ~]# groupdel gaibang     //删除gaibang用户名
    [root@svr223 ~]# id nvshen        //原有组成员自动被解散
    用户id=1002(nvshen) 组id=1003(nvshen) 组=1003(nvshen)
    ```
# vim
![](/images/vim.png)

  https://blog.csdn.net/hsforpyp/article/details/113833465

# 查找
## |
- 利用管道符进行两个命令的链接查找
## find
- 用法
  - find[目录……][条件……]
- 条件如何表示
  - -name文件名：指定文档名称，允许使用通配符*
  - -size+|-大小：指定文档大小（带单位kMG），+|-号表示超过|低于
  - -type类型：指定文档类型，f普通文件、d目录、l链接文件
  - -mtime+|-天数：指定最近修改日期，+|-号表示超过|低于
  - -user用户名：指定文档属主
  - -perm[-ugo]=[rwx]：指定权限
  - -a：用于连接多个条件，表示“`并且`”(-a可以省略)
  - -o：用于连接多个条件，表示“`或者`”
## grep
- 用法
  - grep[-iv]"关键词"文件
  - -i表示忽略大小写
  - -v表示把条件反过来（不包含关键词）
  - 关键词：
    - 仅输出文件中包含“关键词”的行
    - 使用“^ab” 查找以ab开头的行
    - 使用“ab$”查找以ab结尾的行
    - #代表注释，被注释的配置不会生效
  ```
  [root@svr223 ~]# grep "root" /etc/passwd    //列出包含root的行
  root\:x\:0:0:root:/root:/bin/bash
  operator\:x\:11:0:operator:/root:/sbin/nologin
  
  [root@svr223 ~]# grep -i "ROOT" /etc/passwd //列出包含ROOT的行（不区分大小写）
  root\:x\:0:0:root:/root:/bin/bash
  operator\:x\:11:0:operator:/root:/sbin/nologin

  [root@svr223 ~]# grep -v "root" /etc/passwd //列出不包含root的行

  [root@svr223 ~]# grep "^#" /etc/selinux/config    //列出所有被注释的行

  [root@svr223 ~]# grep -v "^#" /etc/selinux/config   //列出没有被注释的行

  [root@svr223 ~]# grep "^$" /etc/selinux/config    //列出所有空行

  [root@svr223 ~]# grep -v "^$" /etc/selinux/config   //列出所有非空行
  ```
 
  https://blog.csdn.net/tsummer2010/article/details/105541433?spm=1001.2014.3001.5502

# 查看进程
## top
- 动态显示系统负载及资源占用排名，类似“任务管理器”
- 资源消耗越多的进程越靠前，按q退出
## 查找杀死进程
### pgrep查找进程
- 用法：pgrep[-l]关键词
  - 列出名称中包含关键词的进程的进程号
  - 选项-l可以一起列出进程名
  ```
  [root@svr223 ~]# pgrep ssh      //列出名称中包含ssh的进程的PID信息
  1065
  1815
  1820
  [root@svr223 ~]# pgrep -l ssh     //列出名称中包含ssh的进程名和PID
  1065 sshd
  1815 sshd
  1820 sshd
  ```
### 杀死进程
- 用法：pkill[-9]进程关键词或kill[-9]进程号
  - 杀死名称中包含关键词的进程或进程号对应的进程
  - 选项-9表示强制杀死进程
  ```
  [root@svr223 ~]# pkill -9 sshd      //强制杀死sshd进程（远程连接会断连）

  [root@svr223 ~]# systemctl restart sshd     //重启sshd（尝试再次连接）
  ```
# ACL访问控制
## 认识ACL策略
- 为文档定义访问规则，针对除u、g、o以外的个别用户或租
  - 文档的属主/属组标记，把访问者区分为三类————属主、属组、其他人
## getfacl查看ACL策略
- 用法：getfacl文档路径……
  - 可以列出文档归属，以及各类用户对此文档的访问权限
  ```
  [root@svr223 ~]# getfacl /root/
  getfacl: Removing leading '/' from absolute path names
  # file: root/
  # owner: root
  # group: root
  user::r-x             //属主的权限
  …… ……                 //acl策略（如果有的话）
  group::r-x            //属组的权限
  other::---            //其他人的权限
  ```

## setfacl配置ACL策略
- 用法：
  - 增加
    1. setfacl -m user：用户名：权限组合 文档……
    2. setfacl -m group：组名：权限组合 文档……
    ```
    [root@svr223 ~]# setfacl -m u:lisi:5 /root    //允许lisi读写
    [root@svr223 ~]# getfacl /root/
    getfacl: Removing leading '/' from absolute path names
    # file: root/
    # owner: root
    # group: root
    user::r-x
    user:lisi:r-x         //增加了一条lisi的权限策略
    group::r-x
    mask::r-x
    other::---          //其他人默认无权访问
    ```
  - 删除
    1. setfacl -x user：用户名 文档……
        - 删除用户的策略
    2. setfacl -x group：组名 文档……
        - 删除组的策略
    3. setfacl -b 文档……
        - 清空所有策略
    ```
    [root@svr223 ~]# setfacl -x u:lisi /root    //删除lisi的ACL策略
    [root@svr223 ~]# setfacl -b /root         //删除所有ACL策略，保留基本ugo权限
    ```

# sudo命令提权
## sudo命令授权及使用
### sudo：执行管理员授权的命令
  - 允许授权用户代替管理员（或其他用户）来完成预先授权的命令
  - 授权用户、授权命令需要提前配置
  - 通过sudo执行命令时，只验证自身的密码（5分钟内不重复验证）
### 配置sudo授权
  - 使用visudo专用工具
    - 授权规则：用户名 本机可用主机名列表=（用户身份）授权命令列表
    - 不指定用户身份时，缺省为root
    - 指定用户组使用%组名
  ```
  # visudo

  %wheel  ALL=(ALL) ALL             //允许wheel组用户使用所有命令
  zx      ALL=/usr/bin/nmtui        //允许用户zx使用nmtui管理网络
  zy      ALL=ALL             //允许用户zy使用所有命令
  ```
### sudo基本用法
  - sudo -l：查看自己的sudo授权
  ```
  [zy@svr223~]$sudo -l
  [sudo]zy的密码：            //初次使用时，验证用户zy的密码（5分钟内不在验证）
  用户zy可以在svr223上运行以下命令：
  (root)ALL
  ```
  - 执行管理员已允许提权的命令
    ```
    [zy@svr223~]$ mkdir /zydir            //正常使用，无权在/下创建子目录
    mkdir:无法创建目录"/zydir"：权限不够

    [zy@svr223~]$ sudo mkdir /zydir       //sudo提权，成功在/下创建子目录
    ```
## sudo日志跟踪
### 开启sudo操作日志
  - 日志审计功能默认不开启
  - 如果需要，可以手动开启sudo操作日志
```
[root@svr223~]# visudo
…………
Defaults logfile=/var/log/sudo      //添加此行，设置sudo日志位置

[root@svr223~]# cat /var/log/sudo     //查看sudo记录
```
# systemctl系统控制
## 控制服务状态
### systemctl工具
  - 用户主要通过systemctl来控制系统和服务状态
### 启动、停止、重启服务
```
systemctl --type service          //列出所有服务
systemctl start 服务名            //启动服务
systemctl stop 服务名             //停止服务
systemctl restart 服务名          //重启服务
```
### 检查服务状态
```
systemctl is-active 服务名            //检查服务状态（简洁）
systemctl status 服务名             //检查服务状态（详细）
```
## 控制服务自启
### 允许服务开机自启
```
systemctl enable 服务名             //允许开机自启
systemctl enable --now 服务名       //允许开机自启，并立即启动
systemctl is-enable 服务名            //查看服务是否被设置为开机自启
```
### 禁止服务开机自启
```
systemctl disable 服务名              //禁止开机自启
systemctl disable --now 服务名        //禁止开机自启，并立即停止
```
# SELinux防护
## SELinux状态控制
### 查看SELinux运行状态
  - 三种运行状态
    - Enforcing，强制（严格按策略执行保护）
    - Permissive，宽松（若有违规会记录，但不做真正的限制）
    - Disabled，禁用（内核不加载SELinux）
  - 检查当前的SELinux运行状态
    ```
    [root@svr223~]# getenforce            //获取SELinux状态
    Enforcing                             //强制保护
    ```
### 切换SELinux运行状态
  - 立即切换（在“强制”与“宽松”之间）
    - 使用setenfore 1|0，执行后立即生效
      ```
      [root@svr223~]# setenfore 0           //切换为宽松模式
      [root@svr223~]# setenfore 1           //切换为强制模式
      ```
# 访问存储设备
## 挂在/卸载设备
- Linux目录结构
  - /boot：硬盘
  - /mnt/dvd：光盘
  - /mnt/udisk：U盘
  - /data
  - /var…………
### 挂载设备
- 关于分区/U盘/光盘设备文件
  - 分区，一般是/dev/sda1、/dev/sda2等等
  - U盘，一般是/dev/sdb1
  - 光盘，一般是/dev/cdrom，指向/dev/sr0等设备
    ```
    [root@svr223 ~]# lsblk                  查看块设备列表
    NAME               MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
    sda                  8:0    0   80G  0 disk 
    ├─sda1               8:1    0    1G  0 part /boot     //第一块磁盘第一个分区
    └─sda2               8:2    0   79G  0 part 
      ├─openeuler-root 253:0    0 50.4G  0 lvm  /
      ├─openeuler-swap 253:1    0  3.9G  0 lvm  [SWAP]
      └─openeuler-home 253:2    0 24.6G  0 lvm  /home
    sr0                 11:0    1  3.4G  0 rom          //光盘
    ```
- 插入光盘——准备挂载点——挂载光盘
  ```
  [root@svr223 ~]# mkdir /mnt/dvd         //创建目录（挂载点）
  [root@svr223 ~]# mount /dev/cdrom /mnt/dvd    //进行挂载
  ```
- 挂在成功后通过挂载点即可访问光盘内文档
  ```
  [root@svr223 ~]# ls /mnt/dvd/
  ```
- 卸载已挂载的光盘
  ```
  [root@svr223 ~]# umount /mnt/dvd/           //卸载设备
  [root@svr223 ~]# ls /mnt/dvd/       //挂载点已为空
  ```
## 配置开机挂载
- 建挂载点——>配置fstab——>检查配置——>重启验证
  ```
  [root@svr223 ~]# mkdir -p /qwe/asd          //创建自动挂载点
  [root@svr223 ~]# vim /etc/fstab             //编辑开机自动挂载配置
  /dev/cdrom  /qwe/asd iso9660 ro 0 0
  [root@svr223 ~]# mount -a                 //检查开机挂在配置（无报错）
  [root@svr223 ~]# reboot                   //重启看看是否自动挂载了
  [root@svr223 ~]# ls /qwe/asd              //确认已自动挂载光盘到指定目录 
  ```
# 配置yum源
### 第一步禁用无效源
```
[root@svr223 ~]# rm -rf /etc/yum.repos.d/*        //删除所有旧的软件源
```
### 第二步设置有效源
- 通过辅助配置工具`快速设置`指定的软件源
  - 用法：yum-config-manager --add-repo   软件源URL地址
    ```
    root@svr223~]# yum-config-manager --add https://mirrors.aliyun.com/centos-stream/9-stream/BaseOS/x86_64/os                  //阿里的基础软件源
    [root@svr223~]# yum-config-manager --add https://mirrors.aliyun.com/centos-stream/9-stream/AppStream/x86_64/os             // 阿里的应用软件源

    如果是加载本地光盘
    [root@svr223~]# yum-config-manager --add-repo file：///qwe/asd/   //自己光盘的挂载
    [root@svr223~]# ls /etc/yum.repos.d/      //确认结果配置
    ```
### 第三步关闭软件来源检查
  - 系统对软件来源合法性的检查
    - 默认只建议安装官方来源的应用软件
    - 但是需要提供发布者密钥
    ```
    [root@svr223~]# vim /etc/yum.conf
    [main]
    gpgcheck=0      //将1改成0可以关闭检查
    …………
    ```
### 验证结果
- 重新获取源数据，确保有可用仓库
  ```
  [root@svr223~]# yum repolist -v                   //检查仓库列表
  …………
  Total packages：软件包数                  //软件源，显示提供包的数量 
  ```
## 使用yum管理软件
### yum查询软件资源
- list列出软件
  - 用来获知xx软件是否已安装/版本/来源
    - 格式：yum list [软件名]……
      ```
      [root@svr223~]# yum list httpd

      [root@svr223 ~]# yum list httpd
      Last metadata expiration check: 0:20:04 ago on 2024年01月05日 星期五 11时21分31秒.
      Available Packages
      httpd.x86_64                                                                  2.4.51-5.oe2203                                                                   repos_openEuler_
      ```
### yum安装/卸载软件
- install安装软件
  - 安装软件
    - 格式：yum [-y] install 软件名|文件名
    - 软件名（yum仓库），文件名（离线安装包）
      ```
      [root@svr223 ~]# yum -y install httpd
      Installed:              //已经成功安装一下软件
        apr-1.7.0-6.oe2203.x86_64                apr-util-1.6.1-14.oe2203.x86_64                 httpd-2.4.51-20.oe2203.x86_64          httpd-filesystem-2.4.51-20.oe2203.noarch     
        httpd-tools-2.4.51-20.oe2203.x86_64      mariadb-connector-c-3.1.13-2.oe2203.x86_64      mod_http2-1.15.25-1.oe2203.x86_64     

      Complete!
      ```
- remove 卸载软件
  - 卸载软件
    - 格式：yum [-y] remove 软件名
      ```
      [root@svr223 ~]# yum -y remove httpd
      Removed:              //已经成功卸载以下软件
        apr-1.7.0-6.oe2203.x86_64                apr-util-1.6.1-14.oe2203.x86_64                 httpd-2.4.51-20.oe2203.x86_64          httpd-filesystem-2.4.51-20.oe2203.noarch     
        httpd-tools-2.4.51-20.oe2203.x86_64      mariadb-connector-c-3.1.13-2.oe2203.x86_64      mod_http2-1.15.25-1.oe2203.x86_64     

      Complete!
      ```
# LAMP部署及测试
## 部署LAMP平台
### B/S服务框架
- 基于Browser/Server架构的网页资源通信
  - 服务端：支持HTTP协议的网页提供程序
  - 浏览器/客户端：下载并按标记规范显示网页的浏览器程序
### 如何获取网页资源
- URL网址
  - 统一资源定位器
  - 资源类别：//服务器地址/目录路径/文件名
### 什么是LAMP
- 一种成熟的动态企业网站服务器模式
  - Apache在最前端，负责处理来自浏览器的web访问请求