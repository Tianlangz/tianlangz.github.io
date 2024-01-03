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
  - /dev：一般存放和设备

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
    - su -用户名（`-`代表切换用户时进入到用户的家目录）
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
  - cd（copy）复制文档
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
  root:x:0:0:root:/root:/bin/bash
  operator:x:11:0:operator:/root:/sbin/nologin
  
  [root@svr223 ~]# grep -i "ROOT" /etc/passwd //列出包含ROOT的行（不区分大小写）
  root:x:0:0:root:/root:/bin/bash
  operator:x:11:0:operator:/root:/sbin/nologin

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