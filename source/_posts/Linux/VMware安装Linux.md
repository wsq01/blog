---
title: Linux vim编辑器
date: 2022-01-11 18:13:15
tags: [Linux]
categories: [Linux]
---

我们以 CentOS 8.1 为例，学习如何安装 Linux 系统。
# 安装VMware Workstation
VMware Workstation是一款桌面虚拟机软件，我们可以利用它来创建Windows、Linux等虚拟机。

1）下载VMware Workstation 安装包，我这里以VMware Workstation Pro v15.5.1为例。

{% asset_img 1.png %}

2）双击安装包进行安装

3）等待片刻，进入安装向导

{% asset_img 2.png %}

4）选择安装所在位置，可自定义。我这里选择默认安装位置

{% asset_img 3.png %}

5）以下两个服务，我不勾选（个人习惯，您可按需勾选）

{% asset_img 4.png %}

6）开始安装

{% asset_img 5.png %}

7）选择“完成”

# 安装 CentOS8.1 操作系统
1）准备镜像文件：`CentOS-8.1.1911-x86_64-dvd1.iso`

官网下载地址：`https://www.centos.org/download/`

2）双击，运行VMware Workstation软件

点击“文件”-->新建虚拟机-->选择“自定义（高级）”-->下一步

{% asset_img 6.png %}

3）启动安装向导

{% asset_img 7.png %}

选择“稍后安装操作系统”，我们在第12步挂载光驱

{% asset_img 8.png %}

4）选择客户机操作系统

{% asset_img 9.png %}

5）设置虚拟机名称；选择虚拟机安装位置

{% asset_img 10.png %}

6）设置虚拟机处理器的数量；我这里选择2个处理器
（注意：处理器的数量不能超过宿主机的数量，如果是做实验1-2个CPU即可）

{% asset_img 11.png %}

7）设置虚拟机内存数量：我这里选择1024MB
（注意：虚拟机的内存也不能超过宿主机，做实验的话1024MB即可满足需求）

{% asset_img 12.png %}

8）建议初学者使用网络地址转换的网络类型：选择该类型，开机自动获取IP

{% asset_img 13.png %}

9）选择磁盘类型

{% asset_img 14.png %}

选择磁盘大小20G
（注意：选择该模式并不会立即分配磁盘容量。它的磁盘空间是随着用户数据的添加而不断变大）

{% asset_img 15.png %}

10）指定磁盘文件名称–>下一步

{% asset_img 16.png %}

11）点击“完成”
（完成到这一步，相当于我们已经采购了一台电脑。我们可以看下它的配置信息）

{% asset_img 17.png %}

12）挂载虚拟光驱，根据箭头方向指示即可。

{% asset_img 18.png %}

# 安装操作系统
1）启动虚拟机，选择第一行，进行安装

{% asset_img 19.png %}

2）选择安装过程中使用的语言

{% asset_img 20.png %}

3）依次选择“时间和日期”，“软件选择”，“安装目的地”

{% asset_img 21.png %}

4）开始安装

{% asset_img 22.png %}

5）根据上图箭头提示，我们设置root密码

{% asset_img 23.png %}

6）我们耐心等待…

{% asset_img 24.png %}

7）重启然后使用吧！

{% asset_img 25.png %}