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

